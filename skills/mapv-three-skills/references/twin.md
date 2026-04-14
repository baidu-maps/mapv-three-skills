# Twin 数字孪生车流

Twin 是 mapv-three 提供的车辆数字孪生组件，用于在 3D 地图上实时渲染和管理大量移动车辆模型。通过定时推送位置数据驱动车辆平滑移动，支持多种内置车辆模型（写实风格和极简风格），并提供车辆追踪、颜色设置、可见性控制等交互能力。

---

## 快速开始

以下是一个最小可运行示例，展示如何创建 Twin 并推送数据：

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，参见 engine-bindmap.md），需设置 enableAnimationLoop: true

// 1. 创建 Twin 实例
const twin = engine.add(new mapvthree.Twin({
    delay: 2000,
    modelConfig: {
        3: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.CAR,
        6: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.BUS,
        10: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.MAN,
    },
}));

// 2. 定时推送车辆数据
setInterval(() => {
    twin.push([
        { id: 'car_1', lng: 116.39, lat: 39.9, dir: Math.PI / 2, time: Date.now(), modelType: 3 },
        { id: 'car_2', lng: 116.40, lat: 39.91, dir: Math.PI, time: Date.now(), modelType: 6 },
    ]);
}, 2000);
```

**关键步骤说明：**
1. 引擎必须设置 `enableAnimationLoop: true`，否则车辆不会动画渲染。
2. 通过 `modelConfig` 配置 modelType 与模型的映射关系。
3. 定时调用 `push()` 推送数据，Twin 内部会自动计算车辆的插值位移，实现平滑移动。`delay` 应设为推送帧间隔时间的 3-5 倍（如 2 秒推送间隔，delay 设 6000-10000）。

---

## Twin 构造函数参数表

```javascript
new mapvthree.Twin(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.modelConfig` | `{ [modelType: number]: string }` | - | 模型配置，key 为 modelType 编号，value 为模型文件地址（glb/gltf 格式） |
| `options.delay` | `number` | `2000` | 插值时间窗口（毫秒），推荐设为数据推送帧间隔的 3-5 倍（如 200ms 推送间隔，delay 设 600-1000ms） |
| `options.objects` | `Array<Object3D>` | `[]` | 车辆额外伴随物体（如标签、特效点等），每辆车都会克隆一份 |
| `options.objectAttributes` | `{ [key: string]: string }` | `{}` | 附加物体所需的数据映射，key 为物体属性名，value 为数据中的字段名 |
| `options.extraDir` | `number` | `0` | 模型额外旋转角度（度），用于修正模型朝向 |
| `options.keepSize` | `boolean` | `false` | 模型是否保持像素大小不变（不随缩放变化） |
| `options.maxScale` | `number` | `20` | keepSize 开启后模型的最大缩放倍数 |
| `options.enableColorList` | `Array<string>` | `['body']` | 允许修改颜色的模型子网格名称列表 |
| `options.autoClearTrack` | `boolean` | `true` | 追踪目标消失时是否自动清除追踪 |

---

## DataProvider 数据提供者

DataProvider 负责将原始推送数据映射为 Twin 所需的标准格式。Twin 默认要求数据包含以下字段：

| 标准字段 | 说明 |
|----------|------|
| `id` | 车辆唯一标识（必需） |
| `point` | 坐标数组 `[lng, lat, alt]`，若未配置则使用 `lng`、`lat` 字段 |
| `time` | 时间戳（毫秒），若未配置 DataProvider 的 time 映射，将自动使用当前时间 |
| `dir` | 车辆朝向（弧度），若不传则根据前后两帧位置自动计算 |
| `modelType` | 模型类型编号，对应 `modelConfig` 中的 key |
| `color` | 车身颜色（CSS 色值字符串） |
| `scale` | 缩放比例 `[x, y, z]`，默认 `[1, 1, 1]` |

### process 方法链

通过 `twin.dataProvider.process(name, callback)` 进行字段映射，支持链式调用。

**参数说明：**
- `name` (string) - 目标标准字段名
- `callback` (string | Function) - 当为字符串时，表示从原始数据中取同名字段；当为函数时，接收原始数据项，返回计算后的值

```javascript
twin.dataProvider
    // 字符串映射：将原始数据的 timestamp 字段映射为 time
    .process('time', 'timestamp')
    // 函数映射：从原始数据计算坐标
    .process('point', item => [item.longitude, item.latitude, item.altitude])
    // 函数映射：角度转弧度（heading 为北偏角度数，需转换为弧度）
    // 注意：不同数据源的角度字段含义不同，需选择对应的转换公式：
    //   - heading（北偏角度数）：(90 - item.heading) / 180 * Math.PI
    //   - global_heading（已为弧度偏移）：item.global_heading + Math.PI / 2
    .process('dir', item => (90 - item.heading) / 180 * Math.PI)
    // 函数映射：根据条件选择模型类型
    .process('modelType', item => item.type)
    // 函数映射：设置颜色
    .process('color', item => item.vehicleColor || '#fff')
    // 自定义字段：用于附加物体（如标签显示文本）
    .process('speed', item => Math.round(item.speed))
    // 自定义字段：控制标签可见性
    .process('labelVisible', item => true);
```

> **注意：** 如果没有通过 `process('time', ...)` 配置 time 映射，DataProvider 会自动为每条数据添加当前时间戳。

---

## 数据推送

### push 方法

```javascript
twin.push(data: Array<Object>)
```

向 Twin 推送一帧车辆数据。每帧数据是一个数组，每个元素代表一辆车的当前状态。

**标准数据格式（未配置 DataProvider 时）：**

```javascript
twin.push([
    {
        id: 'car_1',        // 必需，车辆唯一标识
        lng: 116.39,        // 经度（未配置 point 映射时使用）
        lat: 39.9,          // 纬度
        dir: 1.57,          // 朝向，弧度
        time: 1700000000000,// 时间戳，毫秒
        modelType: 3,       // 模型类型
        color: '#ff0000',   // 车身颜色（可选）
    },
]);
```

**配置 DataProvider 后的数据格式示例：**

```javascript
// 原始数据格式（来自服务端）
const rawData = [
    {
        id: 'car_1',
        x: 116.39,
        y: 39.9,
        z: 0,
        heading: 90,
        timestamp: 1700000000000,
        color: 0,
        speed: 80,
    },
];

// 配置映射后直接推送原始数据
twin.push(rawData);
```

### 数据要求与排查指南

当 Twin 车辆不显示或行为异常时，按以下步骤逐项排查：

#### 步骤 1：检查必需字段

每条车辆数据（经过 DataProvider 映射后）必须包含以下字段，缺少任何一个都会导致该车辆不渲染：

| 字段 | 检查方法 | 缺失表现 |
|------|---------|---------|
| `id` | 每条数据必须有唯一 ID | 车辆无法创建和追踪 |
| `point` 或 `lng`+`lat` | 坐标值必须是有效数字，不能为 NaN/undefined | 车辆不显示 |
| `modelType` | 必须与 `modelConfig` 中的 key 对应 | 该车辆不渲染（静默忽略） |

```javascript
// 排查代码：在 push 前打印检查
twin.dataProvider.process('__debug', item => {
    if (!item.id) console.error('缺少 id:', item);
    if (!item.point && (isNaN(item.lng) || isNaN(item.lat))) console.error('坐标无效:', item);
    return item;
});
```

#### 步骤 2：检查时间戳

| 检查项 | 说明 | 异常表现 |
|--------|------|---------|
| 时间戳递增 | 每次 `push()` 的帧时间戳必须大于上一次 | 车辆倒退或跳闪 |
| 同帧时间一致 | 同一次 `push()` 的数据应使用相同时间戳（内部取 `data[0].time`） | 部分车辆位置计算错误 |
| 时间戳单位 | 必须是毫秒级时间戳（13位数字） | 秒级时间戳会导致插值失败 |
| 时间戳不为空 | 未配置 `process('time', ...)` 时自动用当前时间，配置了则必须有值 | 插值公式除零，车辆卡住 |

```javascript
// 排查代码：检查时间戳顺序
let lastTime = 0;
const originalPush = twin.push.bind(twin);
twin.push = (data) => {
    const time = data[0]?.time;
    if (time && time <= lastTime) {
        console.error('时间戳未递增! 上一帧:', lastTime, '当前帧:', time);
    }
    if (time && String(time).length <= 10) {
        console.warn('时间戳可能是秒级，需要毫秒级（13位）:', time);
    }
    lastTime = time || Date.now();
    originalPush(data);
};
```

#### 步骤 3：检查推送频率与 delay 配置

| 检查项 | 说明 | 异常表现 |
|--------|------|---------|
| 推送了至少2帧 | 插值需要前后两帧，只有1帧时车辆不显示 | 推送了数据但看不到任何车辆 |
| delay 过小 | delay < 推送间隔，缓冲不足 | 车辆频繁卡顿、跳变 |
| delay 过大 | delay >> 推送间隔，延迟太高 | 车辆出现严重延迟，响应滞后 |
| 推送间隔不均匀 | 有时 200ms 有时 5s | 车辆时快时慢、忽停忽走 |

```javascript
// 排查代码：监控推送频率
let pushCount = 0;
let pushStartTime = Date.now();
const originalPush2 = twin.push.bind(twin);
twin.push = (data) => {
    pushCount++;
    const elapsed = Date.now() - pushStartTime;
    if (elapsed > 5000) {
        console.log(`推送频率: ${(pushCount / elapsed * 1000).toFixed(1)} 次/秒, 共 ${pushCount} 帧`);
        pushCount = 0;
        pushStartTime = Date.now();
    }
    originalPush2(data);
};
```

**推送间隔与 delay 对照表：**

| 推送间隔 | 推荐 delay | 说明 |
|----------|-----------|------|
| 200ms | 600-1000ms | 高频实时数据 |
| 1s | 3000-5000ms | 普通实时数据 |
| 2s | 6000-10000ms | 低频数据 |

#### 步骤 4：检查坐标与朝向

| 检查项 | 说明 | 异常表现 |
|--------|------|---------|
| 坐标范围 | 经度 -180~180，纬度 -90~90 | 车辆在地图外不可见 |
| 坐标系 | 必须是 WGS84 经纬度 | 使用投影坐标会导致位置严重偏移 |
| 朝向单位 | `dir` 必须是弧度制（0~2π） | 使用角度制会导致朝向错乱 |
| 前后帧位移 | 相邻帧同一车辆的坐标变化应合理 | 瞬移说明坐标跳变或 ID 复用 |

```javascript
// 排查代码：检查坐标范围
twin.dataProvider.process('__coordCheck', item => {
    const p = item.point || [item.lng, item.lat];
    if (Math.abs(p[0]) > 180 || Math.abs(p[1]) > 90) {
        console.error('坐标超出范围:', item.id, p);
    }
    return item;
});
```

#### 步骤 5：检查引擎配置

| 检查项 | 说明 |
|--------|------|
| `enableAnimationLoop: true` | 必须开启，否则车辆不会动画渲染 |
| 视野范围 | 相机位置是否能看到车辆所在区域 |
| 模型加载 | 检查 `modelLoaded` 事件是否触发，模型 URL 是否可访问 |

```javascript
// 排查代码：确认模型加载
twin.addEventListener('modelLoaded', e => {
    console.log('模型加载完成:', Object.keys(e.modelMap));
});

// 确认有车辆数据在渲染
twin.addEventListener('ticking', e => {
    console.log('当前渲染车辆数:', e.buffers.id.length);
});
```

### 推送频率

`delay` 应设置为推送帧间隔时间的 3-5 倍，以确保在网络抖动或推送不稳定时仍有足够的插值缓冲。例如每 200ms 推送一次数据，`delay` 推荐设为 600-1000ms。Twin 内部通过线性插值在两帧数据之间平滑过渡车辆位置。

**推送间隔与 delay 的关系示例：**

| 推送间隔 | 推荐 delay | 说明 |
|----------|-----------|------|
| 200ms | 600-1000ms | 高频实时数据 |
| 1s | 3000-5000ms | 普通实时数据 |
| 2s | 6000-10000ms | 低频数据 |

---

## 模型配置

### 模型常量 (twinConstants)

通过 `mapvthree.twinConstants` 访问内置模型和颜色常量。

#### REALISTIC_TEMPLATE_MODEL - 写实风格模型

| 常量名 | 说明 |
|--------|------|
| `CAR` | 小汽车 |
| `BUS` | 公交车 |
| `SUV` | SUV |
| `MPV` | MPV |
| `TAXI` | 出租车 |
| `TOUR` | 旅游大巴 |
| `TRUCK` | 货车 |
| `BIGTRUCK` | 大货车 |
| `LARGETRUCK` | 重型货车 |
| `SMALLTRUCK` | 小货车 |
| `MINIBUS` | 小巴 |
| `SMALLBUS` | 中巴 |
| `AMBULANCE` | 救护车 |
| `FIRETRUCK` | 消防车 |
| `POLICECAR` | 警车 |
| `WATERCAR` | 洒水车 |
| `DANGEROUS` | 危险品车 |
| `MAN` | 男性行人 |
| `WOMAN` | 女性行人 |
| `BICYCLE` | 自行车 |
| `TRICYCLE` | 三轮车 |
| `MOTORCYCLE` | 摩托车 |
| `ELECTRICBICYCLE` | 电动自行车 |
| `TRAFFICCONE` | 交通锥 |

```javascript
// 使用写实风格模型
const twin = engine.add(new mapvthree.Twin({
    modelConfig: {
        3: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.CAR,
        6: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.BUS,
    },
}));
```

#### MINIMALIST_TEMPLATE_MODEL - 极简风格模型

与写实风格 key 相同，额外支持带动画的模型：

| 额外常量名 | 说明 |
|-------------|------|
| `MAN_ANIMATE` | 男性行人（带行走动画） |
| `WOMAN_ANIMATE` | 女性行人（带行走动画） |
| `BICYCLE_ANIMATE` | 自行车（带骑行动画） |

```javascript
// 使用极简风格带动画模型
const twin = engine.add(new mapvthree.Twin({
    modelConfig: {
        2: mapvthree.twinConstants.MINIMALIST_TEMPLATE_MODEL.BICYCLE_ANIMATE,
        10: mapvthree.twinConstants.MINIMALIST_TEMPLATE_MODEL.MAN_ANIMATE,
    },
}));
```

> **注意：** 带动画的模型（`*_ANIMATE`）不使用 InstancedMesh 渲染，每辆车有独立的模型实例，性能消耗较高，适合数量较少的场景。

#### SERVICE_TEMPLATE_MODEL - 服务端标准模型映射

基于写实风格，提供了一套后端服务 type 字段与模型的默认映射，可直接作为 `modelConfig` 传入：

| type | 模型 |
|------|------|
| 3 | 小汽车 |
| 4 | 小巴 |
| 5 | 货车 |
| 6 | 公交车 |
| 7 | 自行车 |
| 8 | 摩托车 |
| 9 | 三轮车 |
| 10 | 行人 |
| 11 | 交通锥 |
| 12 | 危险品车 |
| 13 | 警车 |
| 14 | 消防车 |
| 15 | 救护车 |
| 17 | 洒水车 |
| 18 | SUV |
| 19 | MPV |
| 20 | 出租车 |
| 21 | 中巴 |
| 22 | 旅游大巴 |
| 23 | 公交车 |
| 24 | 小货车 |
| 25 | 大货车 |
| 26 | 重型货车 |
| 27 | 货车 |

```javascript
// 使用全量服务端标准模型
const twin = engine.add(new mapvthree.Twin({
    delay: 3000,
    modelConfig: mapvthree.twinConstants.SERVICE_TEMPLATE_MODEL,
}));
```

#### REALISTIC_TEMPLATE_COLOR - 写实风格配色

| key | 颜色 | 色值 |
|-----|------|------|
| `white` | 白色 | `#F4F8FC` |
| `black` | 黑色 | `#25272F` |
| `gray` | 灰色 | `#6F7580` |
| `blue` | 蓝色 | `#284CC7` |
| `red` | 红色 | `#D11800` |
| `green` | 绿色 | `#3FA765` |
| `brown` | 棕色 | `#CC7C42` |
| `yellow` | 黄色 | `#DBA100` |
| `orange` | 橙色 | `#D14D00` |
| `purple` | 紫色 | `#8100D1` |
| `cyanl` | 青色 | `#00BEA5` |
| `pink` | 粉色 | `#D37AD0` |

```javascript
// 在 DataProvider 中使用配色
twin.dataProvider.process('color', item => {
    return mapvthree.twinConstants.REALISTIC_TEMPLATE_COLOR['red'];
});
```

#### 自定义模型

除内置模型外，也可直接传入 glb/gltf 模型文件的 URL：

```javascript
const twin = engine.add(new mapvthree.Twin({
    modelConfig: {
        2: 'https://your-domain.com/models/custom-car.glb',
        3: 'website_assets/models/kache.glb',
    },
}));
```

---

## 控制方法

### pause() - 暂停播放

```javascript
twin.pause();
```

暂停车辆动画，车辆会停留在当前位置。

### start() - 恢复播放

```javascript
twin.start();
```

恢复被 `pause()` 暂停的车辆动画。会自动修正时间偏移，确保恢复后动画连贯。

### reset() - 重置状态

```javascript
twin.reset();
```

清空所有车辆数据和状态，等同于重新初始化。如果 `autoClearTrack` 为 `true`（默认），追踪也会被清除。

### trackById(id, option) - 追踪车辆

```javascript
twin.trackById(id, option)
```

让相机追踪指定 ID 的车辆。返回 `boolean` 表示当前帧是否存在该车辆。

**参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `id` | `string \| number` | - | 车辆 ID |
| `option.radius` | `number` | `100` | 追踪距离（米） |
| `option.pitch` | `number` | `1.2` | 俯仰角 |
| `option.heading` | `number` | `0` | 方位角 |
| `option.height` | `number` | - | 高度偏移（仅平面/2.5D 模式） |
| `option.inside` | `boolean` | `false` | 是否使用车内视角 |
| `option.lock` | `boolean` | `true` | 是否锁定视角 |
| `option.duration` | `number` | `0` | 持续时间（0 表示持续追踪） |
| `option.easing` | `string \| Function` | `'linear'` | 缓动函数 |

```javascript
// 外部追踪视角
twin.trackById('car_1', {
    radius: 200,
    pitch: 1.0,
});

// 车内视角
twin.trackById('car_1', {
    inside: true,
});
```

### clearTrack() - 清除追踪

```javascript
twin.clearTrack();
```

清除通过 `trackById` 设置的追踪。

### setColorById(id, color) - 设置车辆颜色

```javascript
twin.setColorById('car_1', '#ff0000');
twin.setColorById('car_1', 0xff0000);
```

动态设置指定车辆的颜色。颜色会应用到模型中 `enableColorList` 指定的子网格上。

### setVisibleByType(modelType, status) - 按模型类型设置可见性

```javascript
// 隐藏所有 modelType 为 3 的车辆
twin.setVisibleByType(3, false);
// 显示
twin.setVisibleByType(3, true);
```

### setObjectVisible(status) - 设置所有附加物可见性

```javascript
twin.setObjectVisible(false); // 隐藏所有附加物（标签、特效等）
```

### setObjectVisibleByType(modelType, status) - 按模型类型设置附加物可见性

```javascript
twin.setObjectVisibleByType(3, false);
```

### getModelInstance(modelType) - 获取模型实例

```javascript
const instance = twin.getModelInstance(3);
// 返回 DynamicInstancedMesh 实例
```

### models (属性) - 获取所有模型实例

```javascript
const allModels = twin.models;
// 返回 { [modelType]: DynamicInstancedMesh }
```

### dispose() - 销毁实例

```javascript
twin.dispose();
```

销毁 Twin 实例，释放所有模型、纹理等 GPU 资源。

---

## MockTwin 模拟数据

MockTwin 用于基于路径线数据模拟车辆行驶，无需真实轨迹数据即可生成车流效果，适合演示和测试。

### 构造函数

```javascript
new mapvthree.MockTwin(twinConfig, data, probabilityConfig)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `twinConfig` | `Object` | Twin 构造参数，字段与 `new mapvthree.Twin(options)` 的 options 一致（含 `delay`、`modelConfig`、`keepSize` 等，详见上方 Twin 构造函数参数表） |
| `data` | `Array<Array<number>> \| GeoJSON.LineString` | 路径数据，`[[lng, lat], ...]` 点数组或 GeoJSON 线 |
| `probabilityConfig` | `Object` | 颜色和模型类型的随机概率配置，包含 `color`（`{ [colorName\|cssColor: string]: number }`，颜色到概率的映射）和 `modelType`（`{ [modelType: number]: number }`，模型类型到概率的映射），概率值之和应为 1。详见下方格式说明 |

### probabilityConfig 格式

```javascript
{
    color: {
        'red': 0.3,    // 30% 概率为红色
        'blue': 0.3,   // 30% 概率为蓝色
        'yellow': 0.2, // 20% 概率为黄色
        '#333': 0.2,   // 20% 概率为自定义色值
    },
    modelType: {
        3: 0.5,  // 50% 概率为 modelType 3
        6: 0.3,  // 30% 概率为 modelType 6
        10: 0.2, // 20% 概率为 modelType 10
    },
}
```

> **注意：** `color` 中的 key 如果是 `REALISTIC_TEMPLATE_COLOR` 中定义的颜色名（如 `red`、`blue`），会自动映射为对应色值。也可以直接使用 CSS 色值字符串。

### start(options) - 开始模拟

```javascript
mockTwin.start(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.speed` | `number` | `80` | 车辆速度（km/h） |
| `options.gap` | `number` | `10` | 路径上相邻车辆间距（米） |
| `options.initialCount` | `number` | `0` | 启动时预先放置的车辆数量 |

### clear() - 停止模拟

```javascript
mockTwin.clear();
```

停止模拟并清空所有车辆，清除定时器。

### visibleCallback - 可见性回调

可覆盖此方法，控制 MockTwin 的显示/隐藏逻辑：

```javascript
mockTwin.visibleCallback = () => {
    return engine.map.getZoom() > 18;
};
```

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `speed` | `number` | 车辆速度（km/h），可动态修改 |
| `gap` | `number` | 车辆间距（米），可动态修改 |
| `data` | `Array \| GeoJSON` | 路径数据 |
| `probabilityConfig` | `Object` | 概率配置，可动态修改 |

### 完整示例

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，参见 engine-bindmap.md），需设置 enableAnimationLoop: true

// 从文件加载路径数据
fetch('data/json/routes.json').then(rs => rs.json()).then(positionList => {
    const mockTwin = engine.add(new mapvthree.MockTwin(
        {
            delay: 1000,
            modelConfig: mapvthree.twinConstants.SERVICE_TEMPLATE_MODEL,
        },
        positionList,
        {
            color: {
                'red': 0.1,
                'blue': 0.3,
                'yellow': 0.3,
                'pink': 0.2,
                '#333': 0.1,
            },
            modelType: {
                3: 0.3,
                6: 0.3,
                10: 0.2,
                18: 0.2,
            },
        }
    ));

    // 根据缩放级别控制可见性
    mockTwin.visibleCallback = () => engine.map.getZoom() > 18;

    // 启动模拟：预置1000辆车，间距2米，时速80km/h
    mockTwin.start({
        gap: 2,
        speed: 80,
        initialCount: 1000,
    });
});
```

---

## 车辆标签

使用 `Label` 组件（见 [Label 标签](./label-icon.md)）作为 Twin 的附加物体，在车辆上方显示信息标签（如速度、车牌号等）。

```javascript
// 1. 创建 Label
const label = engine.add(new mapvthree.Label({
    type: 'text',
    textSize: 14,
    textFillStyle: '#fff',
    keepSize: true,
    depthTest: false,
    transparent: true,
}));

// 2. 作为附加物体传入 Twin
const twin = engine.add(new mapvthree.Twin({
    delay: 2000,
    modelConfig: { 3: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.CAR },
    objectAttributes: {
        'text': 'speed',  // 标签显示的文本取数据中的 speed 字段
    },
    objects: [label],
}));

// 3. 在 DataProvider 中设置对应的字段
twin.dataProvider
    .process('speed', item => Math.round(item.speed) + ' km/h');
```

> `objectAttributes` 中的 key 对应附加物体的数据源属性名，value 对应推送数据中的字段名。

---

## 事件交互

### click 事件

Twin 支持点击事件，需要先开启射线检测：

```javascript
twin.receiveRaycast = true;

twin.addEventListener('click', (e) => {
    console.log('点击的车辆信息:', e.clickInfo);
});
```

### modelLoaded 事件

```javascript
twin.addEventListener('modelLoaded', (e) => {
    console.log('模型加载完成:', e.modelMap);
});
```

### ticking 事件

```javascript
twin.addEventListener('ticking', (e) => {
    console.log('当前车辆数:', e.buffers.id.length);
});
```

---

## 常见场景

### 场景一：WebSocket 实时车流

通过 WebSocket 接收实时车辆数据并推送到 Twin：

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {io} from 'socket.io-client';

// 引擎初始化（略，参见 engine-bindmap.md），需设置 enableAnimationLoop: true

// 创建标签
const label = engine.add(new mapvthree.Label({
    type: 'text',
    textSize: 14,
    textFillStyle: '#fff',
    keepSize: true,
    depthTest: false,
    transparent: true,
}));

// 创建 Twin
const twin = engine.add(new mapvthree.Twin({
    delay: 3000,
    modelConfig: mapvthree.twinConstants.SERVICE_TEMPLATE_MODEL,
    objectAttributes: { 'text': 'plateNo' },
    objects: [label],
}));

// 数据映射
twin.dataProvider
    .process('time', 'timestamp')
    .process('plateNo', 'id')
    .process('point', item => [item.longitude, item.latitude, item.altitude])
    .process('dir', item => (90 - item.heading) / 180 * Math.PI)
    .process('modelType', item => item.type)
    .process('color', item => item.color);

// WebSocket 连接
const socket = io('ws://your-server:8000/track_stream', {
    transports: ['websocket'],
});
socket.emit('tracks', JSON.stringify({
    lon: 114.437, lat: 22.604, radius: 800,
}));
socket.on('tracks', (e) => {
    e.tracks.forEach(item => { item.timestamp = e.timestamp; });
    twin.push(e.tracks);
});
```

### 场景二：本地 JSON 数据回放

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，参见 engine-bindmap.md），需设置 enableAnimationLoop: true

const twin = engine.add(new mapvthree.Twin({
    delay: 3000,
    modelConfig: {
        2: mapvthree.twinConstants.MINIMALIST_TEMPLATE_MODEL.CAR,
        3: mapvthree.twinConstants.MINIMALIST_TEMPLATE_MODEL.TRUCK,
    },
}));

twin.dataProvider
    .process('time', 'timestamp')
    .process('point', item => [item.position_lon, item.position_lat, 0])
    .process('dir', item => item.global_heading + Math.PI / 2)
    .process('modelType', item => 2);

// 加载 JSON 数据并按时间间隔回放
fetch('data/json/twin_hmi.json').then(rs => rs.json()).then(frames => {
    let index = 0;
    function playNextFrame() {
        if (index + 2 >= frames.length) {
            index = 0; // 循环播放
        }
        twin.push(frames[index]);
        index++;
        const nextInterval = frames[index + 1][0].timestamp - frames[index][0].timestamp;
        setTimeout(playNextFrame, nextInterval);
    }
    playNextFrame();

    // 追踪第一辆车
    twin.trackById(frames[0][0].id, {
        radius: 30,
        pitch: 70,
        height: 0,
    });
});
```

### 场景三：带特效的车辆（标签 + 波纹特效）

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，参见 engine-bindmap.md），需设置 enableAnimationLoop: true

// 速度标签
const label = engine.add(new mapvthree.Label({
    type: 'text',
    textSize: 14,
    textFillStyle: '#fff',
    keepSize: true,
    depthTest: false,
    transparent: true,
}));

// 波纹特效
const bubble = engine.add(new mapvthree.EffectPoint({
    color: 'rgba(210, 160, 57, 1.0)',
    size: 30,
    type: 'Wave',
    duration: 1000,
    transparent: true,
    depthTest: true,
    depthWrite: true,
}));

// Twin 配置附加物体
const twin = engine.add(new mapvthree.Twin({
    delay: 1000,
    keepSize: true,
    modelConfig: {
        2: mapvthree.twinConstants.MINIMALIST_TEMPLATE_MODEL.BICYCLE_ANIMATE,
        3: mapvthree.twinConstants.REALISTIC_TEMPLATE_MODEL.TRUCK,
    },
    objectAttributes: { 'text': 'speed' },
    objects: [label, bubble],
}));

twin.dataProvider
    .process('time', 'timestamp')
    .process('speed', item => 80 + Math.round(Math.random() * 20))
    .process('point', item => [item.x, item.y, item.z])
    .process('dir', item => (90 - item.heading) / 180 * Math.PI)
    .process('modelType', item => (item.color < 5 ? 2 : 3))
    .process('color', item => mapvthree.twinConstants.REALISTIC_TEMPLATE_COLOR['red'])
    .process('labelVisible', item => true);
```

---

## 注意事项

1. **引擎配置**：必须设置 `rendering.enableAnimationLoop: true`，否则 Twin 车辆不会进行动画渲染。

2. **delay 与推送频率**：`delay` 应设为数据推送帧间隔的 3-5 倍，以提供足够的插值缓冲。例如 200ms 推送间隔，`delay` 推荐设为 600-1000ms；1 秒推送间隔，`delay` 推荐设为 3000-5000ms。Twin 内部使用 delay 作为两帧数据之间的插值时间窗口。

3. **时间同步**：Twin 内部会自动计算数据时间与客户端时间的偏移量，并做抖动校正。如果数据的时间戳与当前时间差异过大，Twin 会自动调整。如果数据不包含 time 字段且未配置 DataProvider 的 time 映射，会自动使用当前时间。

4. **modelType 匹配**：推送数据中的 `modelType` 值必须与 `modelConfig` 中的 key 对应，否则该车辆不会被渲染。

5. **动画模型与实例化模型**：
   - 无动画的模型使用 `DynamicInstancedMesh` 进行实例化渲染，性能好，适合大量车辆。
   - 有动画的模型（如 `MAN_ANIMATE`、`BICYCLE_ANIMATE`）使用独立的 `AnimationModel` 实例，每辆车都有独立的模型和动画混合器，性能消耗较高。

6. **颜色设置**：`enableColorList` 默认为 `['body']`，即只有模型中名为 `body` 的子网格会响应颜色修改。如果自定义模型的网格名称不同，需要修改此配置。

7. **附加物体 (objects)**：通过 `objects` 传入的物体会被自动克隆到每种 modelType，并通过 `objectAttributes` 建立数据映射。物体需要先通过 `engine.add()` 添加后再传给 Twin。

8. **资源释放**：当不再需要 Twin 时，调用 `twin.dispose()` 释放 GPU 资源。对于 MockTwin，需调用 `mockTwin.dispose()` 或 `mockTwin.clear()`。

9. **追踪行为**：当 `autoClearTrack` 为 `true`（默认）时，如果被追踪的车辆从数据中消失（不再推送），追踪会自动清除。设为 `false` 可以保持追踪状态。

10. **坐标系**：推送的坐标需使用 WGS84 经纬度 `[lng, lat, alt]`。`dir`（朝向）使用弧度制。如果原始数据使用角度制，需在 DataProvider 中进行转换。
