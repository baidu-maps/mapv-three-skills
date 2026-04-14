# TileLightManager 实时信号灯

## 简介

`TileLightManager` 是 MapV-Three 提供的实时信号灯管理组件，基于瓦片化的方式加载和渲染路口红绿灯。它自动根据当前视野范围请求信号灯数据，并以时间表驱动方式实时更新灯态（红、黄、绿及闪烁），支持倒计时数字显示和方向箭头图标。

核心能力：
- 按瓦片分块请求信号灯状态数据，仅加载视野内数据
- 支持两种内置数据请求器（`DefaultLampRequest` 按城市、`RegionLampRequest` 按区域），也可自定义
- 两种数据刷新模式：循环播放时间表 / 定时重新请求
- 自动根据地图缩放层级显示或隐藏信号灯

---

## 交互判断

用户要求加载实时信号灯时，先检查用户已提供了哪些信息，已提供的直接使用，只对缺少的信息进行询问。

**判断规则：**
- 用户提供了 `regionId` → 直接用 `RegionLampRequest`，不问数据源类型
- 用户提供了 `cityCode` → 直接用 `DefaultLampRequest`，不问数据源类型
- 用户提供了 `crs` 或能从项目 projection 推断 → 不问坐标系
- 用户提供了 `host`、`ak` → 不问这些参数
- 全部信息齐全 → 直接生成代码，不做任何询问

**缺少信息时的询问方式：**
- 数据源类型不明确（没有 cityCode 也没有 regionId）→ 通过交互确认同时询问数据源类型和坐标系（用 2 个 questions），推荐项放在第一个：
  - 问题 1: "信号灯数据源使用哪种请求方式？" 选项: [`按城市请求 — DefaultLampRequest`（推荐）, `按区域请求 — RegionLampRequest`]
  - 问题 2: "使用哪种坐标系？" 选项: [`gcj02ll — 国测局坐标`（推荐）, `wgs84ll — WGS84 坐标`]
- 用户回答后，在回复中列出仍然缺少的必填参数（host、ak、cityCode/regionId）让用户补充
- 如果只缺坐标系（数据源类型已明确）→ 单独通过交互确认询问坐标系

全部信息收集完成后生成代码。

---

## 快速开始

```js
// 假设 engine 已初始化（需开启 enableAnimationLoop）

// 1. 创建数据请求器
const api = new DefaultLampRequest({
    host: 'https://your-api-host.com',
    ak: 'your_ak',
    cityCode: '131',
    crs: 'gcj02ll',
});

// 2. 创建信号灯管理器
const tileLightManager = new TileLightManager(api, {
    height: 10,
    periodUpdate: true,
    maxShowLevel: 18,
    lampConfig: {
        emissiveIntensity: 1,
        showLine: true,
        color: 0x2B385E,
        worldUnits: false,
        worldScale: 10,
    },
});

// 3. 添加到引擎
engine.add(tileLightManager);
```

---

## 构造函数

```js
new TileLightManager(api, options?)
```

### 参数 `api`

| 类型 | 说明 |
| --- | --- |
| `LampRequest` | 数据请求器实例，必须是 `LampRequest` 的子类（如 `DefaultLampRequest`、`RegionLampRequest` 或自定义子类） |

### 参数 `options`

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `height` | `number` | `10` | 信号灯离地高度（世界单位），同时也是灯杆连接线的长度 |
| `maxShowLevel` | `number` | `18` | 最大显示层级。当 `autoHide` 为 `true` 时，地图缩放等级需大于此值才显示信号灯 |
| `maxCount` | `number` | `6` | 缓存队列中最多保留多少组瓦片数据，超出时自动移除最旧的数据并清理场景对象 |
| `tileSize` | `number` | `500` | 每个瓦片的大小（用于将经纬度映射为瓦片编号，单位为经纬度 × 100000） |
| `range` | `number` | `0` | 同时获取视野中心周围多少层瓦片，实际取 `min(range, 1)`。`0` 表示只取中心点所在瓦片 |
| `autoHide` | `boolean` | `true` | 是否根据地图缩放层级自动隐藏信号灯。为 `true` 时仅在 `zoom > maxShowLevel` 时显示 |
| `periodUpdate` | `boolean` | `false` | 是否启用定时刷新模式。为 `true` 时，当灯态时间表的所有阶段播放完毕后，会自动重新请求数据 |
| `lampConfig` | `object` | `{showLine: true, color: 0x2B385E}` | 信号灯实例的外观配置，详见下方 lampConfig 参数表 |

---

## lampConfig 参数

`lampConfig` 控制信号灯的渲染外观，传入 `options.lampConfig`。

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `showLine` | `boolean` | `true` | 是否显示信号灯下方的灯杆连接线 |
| `color` | `number \| string` | `0x2B385E` | 灯杆连接线的颜色，支持十六进制数值或 CSS 颜色字符串 |
| `emissiveIntensity` | `number` | — | 信号灯材质的自发光强度，数值越大灯光越亮（配合 bloom 后处理效果更佳） |
| `worldUnits` | `boolean` | `true` | 是否使用世界单位渲染信号灯。为 `true` 时灯的大小固定在世界空间；为 `false` 时灯的大小在屏幕空间保持固定像素 |
| `worldScale` | `number` | `1` | 当 `worldUnits` 为 `true` 时，信号灯面板的缩放系数。灯面板宽度 = `1.5 × worldScale`，高度 = `4.28 × worldScale` |
| `sizeAttention` | `boolean` | `true` | 是否让信号灯始终面向相机（Billboard 效果） |
| `pixelHeight` | `number` | `20` | 当 `worldUnits` 为 `false` 时，信号灯在屏幕上的固定像素高度 |

---

## 数据请求器

### DefaultLampRequest

按城市维度请求信号灯数据，接口路径为 `/duearth/lamp/state`。

```js
import { DefaultLampRequest } from '@baidu/mapv-three';

const api = new DefaultLampRequest({
    host: 'https://your-api-host.com',  // 服务地址
    ak: 'your_ak',                       // 鉴权密钥
    cityCode: '131',                     // 城市编码
    crs: 'gcj02ll',                      // 坐标系，默认 'gcj02ll'
});
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `host` | `string` | 是 | — | API 服务地址 |
| `ak` | `string` | 是 | — | 鉴权密钥 |
| `cityCode` | `string` | 是 | — | 城市编码 |
| `crs` | `string` | 否 | `'gcj02ll'` | 坐标参考系统，可选 `'gcj02ll'`、`'wgs84ll'` 等 |

请求 URL 格式：`{host}/duearth/lamp/state?tile_x={x}&tile_y={y}&ak={ak}&crs={crs}&city_code={cityCode}`

### RegionLampRequest

按区域 ID 请求信号灯数据，接口路径为 `/duearth/lamp/region/state`。适用于不按城市而按自定义区域划分的场景。

```js
import { RegionLampRequest } from '@baidu/mapv-three';

const api = new RegionLampRequest({
    host: 'https://your-api-host.com',
    ak: 'your_ak',
    regionId: 'your_region_id',
    crs: 'gcj02ll',
});
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `host` | `string` | 是 | — | API 服务地址 |
| `ak` | `string` | 是 | — | 鉴权密钥 |
| `regionId` | `string` | 是 | — | 区域 ID |
| `crs` | `string` | 否 | `'gcj02ll'` | 坐标参考系统 |

请求 URL 格式：`{host}/duearth/lamp/region/state?tile_x={x}&tile_y={y}&ak={ak}&crs={crs}&region_id={regionId}`

### 自定义数据源 (LampRequest)

通过继承 `LampRequest` 基类并实现 `fetchLamps` 方法，可以接入任意数据源。

```js
import { LampRequest, TileLightManager } from '@baidu/mapv-three';

class CustomLampRequest extends LampRequest {
    async fetchLamps(params) {
        // params: { x: tileX, y: tileY }
        const res = await fetch(`your-api?x=${params.x}&y=${params.y}`);
        return await res.json();
    }
}

const api = new CustomLampRequest();
const manager = new TileLightManager(api, { /* options */ });
```

#### `fetchLamps` 返回数据格式

```js
{
    now_timestamp: 1728897508563,       // 服务器当前时间戳（ms）
    lamps: [
        {
            signalmachine_timestamp: 1728897501000,  // 信号机时间戳（ms），用于计算灯态偏移
            light_id: '16801854100',                 // 灯组唯一 ID
            light_location: '116.404,39.915',        // 经度,纬度（逗号分隔字符串）
            light_flows: [                           // 该灯组的各方向灯态
                {
                    road_flow_direction: '12',       // 方向编码（见下方方向编码表）
                    lamp_stage_0: {                  // 阶段0（最多4个阶段：lamp_stage_0 ~ lamp_stage_3）
                        lamp_status: 23,             // 灯态编码（见下方灯态编码表）
                        count_down: 30,              // 倒计时秒数（10000 表示不显示倒计时数字）
                        period: 30,                  // 该阶段持续秒数
                    },
                    lamp_stage_1: { /* ... */ },
                    lamp_stage_2: { /* ... */ },
                    // lamp_stage_3 可选
                },
            ],
        },
    ],
}
```

#### 灯态编码表 (lamp_status)

| 编码 | 含义 | 显示 |
| --- | --- | --- |
| `21` | 红灯 | 红色常亮 |
| `22` | 黄灯 | 黄色常亮 |
| `23` | 绿灯 | 绿色常亮 |
| `24` | 黄闪 | 黄色闪烁 |
| `41` | 黄闪 | 黄色闪烁 |
| `11` | 关灯 | 灭灯 |
| `31` | 关灯 | 灭灯 |

#### 方向编码表 (road_flow_direction)

| 编码 | 含义 |
| --- | --- |
| `11` | 掉头 |
| `12` | 左转 |
| `13` | 左转待行 |
| `21` | 直行 |
| `22` | 直行待行 |
| `23` | 右转 |
| `24` | 右转待行 |
| `31` | 其他 |
| `99` | 特殊 |

---

## 方法与属性

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `visible` | `boolean` | 获取或设置信号灯的可见性。设为 `false` 隐藏所有信号灯及灯杆线；重新设为 `true` 时会立即触发一次数据更新 |

### 生命周期方法（由引擎自动调用）

| 方法 | 说明 |
| --- | --- |
| `afterAddToEngine(engine)` | 被添加到引擎后由引擎调用，保存引擎引用 |
| `onBeforeScenePrepareRender(engine, scene, camera, renderState)` | 每帧渲染前由引擎调用，负责检测视野变化、更新瓦片数据和渲染灯态 |

> `TileLightManager` 继承自 Three.js 的 `Object3D`，因此也具备标准的 `add()`、`remove()`、`traverse()` 等方法。

---

## 常见场景

### 场景一：使用 DefaultLampRequest 接入城市信号灯

```js
// 假设 engine 已初始化（需开启 enableAnimationLoop）
// 开启 bloom 后处理，让信号灯自发光效果更突出
engine.rendering.bloom.enabled = true;

const api = new DefaultLampRequest({
    host: 'https://your-api-host.com',
    ak: 'your_ak',
    cityCode: '131',
    crs: 'wgs84ll',
});

const tileLightManager = new TileLightManager(api, {
    height: 35,
    periodUpdate: true,
    maxShowLevel: 18,
    autoHide: false,
    lampConfig: {
        emissiveIntensity: 1,
        showLine: true,
        worldUnits: false,
        worldScale: 1,
    },
});

engine.add(tileLightManager);
```

### 场景二：使用 RegionLampRequest 按区域请求

```js
import { TileLightManager, RegionLampRequest } from '@baidu/mapv-three';

const api = new RegionLampRequest({
    host: 'https://your-api-host.com',
    ak: 'your_ak',
    regionId: 'your_region_id',
    crs: 'gcj02ll',
});

const tileLightManager = new TileLightManager(api, {
    height: 10,
    periodUpdate: true,
    maxShowLevel: 15,
    lampConfig: {
        emissiveIntensity: 1,
        showLine: true,
        color: '#eeeeee',
        worldUnits: false,
        worldScale: 1,
    },
});

engine.add(tileLightManager);
```

### 场景三：自定义数据源

```js
import { LampRequest, TileLightManager } from '@baidu/mapv-three';

class CustomLampRequest extends LampRequest {
    async fetchLamps(params) {
        // params: { x: tileX, y: tileY }
        return {
            now_timestamp: Date.now(),
            lamps: [
                {
                    signalmachine_timestamp: Date.now(),
                    light_id: 'light_001',
                    light_location: '116.404,39.915',
                    light_flows: [
                        {
                            road_flow_direction: '21',
                            lamp_stage_0: {
                                lamp_status: 23,
                                count_down: 30,
                                period: 30,
                            },
                            lamp_stage_1: {
                                lamp_status: 22,
                                count_down: 3,
                                period: 3,
                            },
                            lamp_stage_2: {
                                lamp_status: 21,
                                count_down: 40,
                                period: 40,
                            },
                        },
                    ],
                },
            ],
        };
    }
}

const tileLightManager = new TileLightManager(
    new CustomLampRequest(),
    {
        height: 15,
        autoHide: false,
        periodUpdate: true,
        lampConfig: {
            showLine: true,
            worldUnits: false,
            worldScale: 10,
        },
    }
);

engine.add(tileLightManager);
```

### 场景四：动态切换可见性

```js
// 隐藏信号灯
tileLightManager.visible = false;

// 显示信号灯（会自动触发数据刷新）
tileLightManager.visible = true;
```

---

## 注意事项

1. **必须开启动画循环**：`TileLightManager` 依赖每帧渲染回调来驱动灯态动画，创建 Engine 时必须设置 `rendering.enableAnimationLoop: true`。

2. **坐标系一致性**：`DefaultLampRequest` 和 `RegionLampRequest` 的 `crs` 参数必须与地图的坐标系保持一致。使用 `projection: 'ecef'` 时通常对应 `'wgs84ll'`，使用百度墨卡托投影时对应 `'gcj02ll'`。

3. **`periodUpdate` 与数据刷新**：
   - `periodUpdate: false`（默认）：数据的灯态时间表一次请求后持续循环播放，不会重新请求。
   - `periodUpdate: true`：当所有灯态阶段播放完毕后，自动重新请求最新数据，适用于需要实时同步信号机状态的场景。

4. **bloom 后处理**：建议搭配 `engine.rendering.bloom.enabled = true` 使用，可以让信号灯的自发光（`emissiveIntensity`）产生辉光效果，提升视觉表现。

5. **`autoHide` 与 `maxShowLevel`**：
   - 当 `autoHide: true`（默认）时，地图缩放等级必须大于 `maxShowLevel` 才会显示信号灯，适合在大范围浏览时避免渲染过多信号灯。
   - 设为 `autoHide: false` 可以在任意层级都显示信号灯。

6. **`worldUnits` 默认使用 `false`**：生成代码时默认设置 `worldUnits: false`（屏幕空间固定像素大小），除非用户明确要求世界空间大小。
   - `worldUnits: false`（推荐）：信号灯保持固定像素大小，不随镜头远近变化，始终清晰可读。
   - `worldUnits: true`：信号灯大小在世界空间中固定，随镜头远近自然缩放。

7. **`maxCount` 限制**：组件最多缓存 `maxCount`（默认 6）组瓦片数据，超出时最旧的灯组会被从场景中移除。如果视野范围较大需要显示更多信号灯，可适当增大此值。

8. **`count_down` 特殊值**：当 `count_down` 为 `10000` 时，信号灯面板不显示倒计时数字，仅显示灯色和方向箭头。

9. **`lampConfig` 可选**：`lampConfig` 不传也能渲染，所有子参数都有默认值（`showLine: true`、`worldUnits: true`、`worldScale: 1`、`color: 0x2B385E`）。按需调整即可，不需要全部传入。

10. **`crs` 必须与投影匹配**：这是最常见的信号灯渲染不出来的原因。`crs` 不正确会导致信号灯坐标偏移到视野之外。`projection: 'ecef'` 对应 `crs: 'wgs84ll'`，百度墨卡托对应 `crs: 'gcj02ll'`。
