# 引擎初始化与地图绑定

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 最简初始化：传入容器元素或 ID
const engine = new mapvthree.Engine('map_container', {
    rendering: { sky: null },
});

// 设置视野
engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(14);

// 添加物体到场景
const sky = engine.add(new mapvthree.DynamicSky());

// 使用完毕后销毁
engine.dispose();
```

## 构造函数

```javascript
const engine = new mapvthree.Engine(container, options);
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `container` | `HTMLElement \| string` | 是 | 容器 DOM 元素或元素 ID。容器会自动设置 `position: relative`（若当前为 `static`） |
| `options` | `object` | 否 | 配置选项对象，包含以下子配置 |

### options 配置项

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.map` | `object` | `{}` | 地图配置，传入 `EngineMap` 构造函数 |
| `options.rendering` | `object` | `{}` | 渲染配置，传入 `EngineRendering` 构造函数 |
| `options.event` | `object` | `{}` | 事件系统配置 |
| `options.selection` | `object` | `{}` | 选择器配置 |
| `options.widgets` | `object` | `{}` | UI 控件配置 |
| `options.controller` | `object` | `{}` | 键鼠交互控制器配置 |
| `options.clock` | `object` | `{}` | 时钟系统配置 |

#### options.map 地图配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `projection` | `string` | `'EPSG:3857'` | 目标投影。可选：`EPSG:4326`（WGS84）、`EPSG:3857`（Web墨卡托）、`EPSG:4978`（ECEF 球体模式） |
| `is3DControl` | `boolean` | `true` | 是否启用 3D 交互控制（支持旋转和倾斜） |
| `center` | `number[]` | - | 初始视野中心点 `[经度, 纬度]` |
| `heading` | `number` | `0` | 初始旋转角度（度），正北为 0，逆时针递增 |
| `pitch` | `number` | `0` | 初始俯仰角（度），俯视为 0，平视为 90 |
| `range` | `number` | - | 初始相机距地面距离（米） |
| `provider` | `TileProvider` | 自动 | 底图 Provider，设为 `null` 可禁用默认底图 |

#### options.rendering 渲染配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enableAnimationLoop` | `boolean` | `false` | 是否开启动画循环渲染。有动画效果时需设为 `true` |
| `animationLoopFrameTime` | `number` | `16` | 动画帧间隔（毫秒），16 约等于 60fps |
| `pixelRatio` | `number` | `window.devicePixelRatio` | 像素比，影响渲染清晰度 |
| `logarithmicDepthBuffer` | `boolean` | `true` | 是否使用对数深度缓冲，解决远距离 z-fighting |
| `sky` | `Sky` | `DefaultSky` | 初始天空对象实例 |
| `forceWebGL` | `boolean` | `false` | 是否强制使用 WebGL |
| `contextParameters` | `object` | - | WebGL 上下文参数 |
| `contextParameters.preserveDrawingBuffer` | `boolean` | `false` | 是否保留绘图缓冲区，截图时需设为 `true` |
| `features` | `object` | - | 渲染特性配置（见下表） |

##### features 渲染特性

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `features.antialias.enabled` | `boolean` | `true` | 是否开启抗锯齿 |
| `features.antialias.method` | `string` | `'smaa'` | 抗锯齿方法：`'smaa'` 或 `'msaa'` |
| `features.bloom.enabled` | `boolean` | `false` | 是否开启泛光效果 |
| `features.bloom.strength` | `number` | `0.1` | 泛光强度 |
| `features.bloom.threshold` | `number` | `1` | 泛光阈值 |
| `features.bloom.radius` | `number` | `0` | 泛光半径 |
| `features.ao.enabled` | `boolean` | `false` | 是否开启环境光遮蔽 |
| `features.shadow.enabled` | `boolean` | `false` | 是否开启阴影 |
| `features.reflection.enabled` | `boolean` | `false` | 是否开启反射 |

#### options.widgets UI 控件配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `widgets.logo.enabled` | `boolean` | `false` | Logo 控件 |
| `widgets.zoom.enabled` | `boolean` | `false` | 缩放按钮控件 |
| `widgets.scale.enabled` | `boolean` | `false` | 比例尺控件 |
| `widgets.fullscreen.enabled` | `boolean` | `false` | 全屏按钮控件 |
| `widgets.compass.enabled` | `boolean` | `false` | 指南针控件 |
| `widgets.geoLocate.enabled` | `boolean` | `false` | 地理定位控件 |
| `widgets.exportImage.enabled` | `boolean` | `false` | 导出图片控件 |
| `widgets.mapInfo.enabled` | `boolean` | `false` | 地图信息控件 |
| `widgets.mapInfo.template` | `string` | - | 信息模板，如 `'CZHP'`（中心点、缩放、方向、俯仰） |
| `widgets.drawer.enabled` | `boolean` | `false` | 图层抽屉控件 |

#### options.controller 控制器配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | `boolean` | `true` | 是否启用控制器 |
| `enableRotate` | `boolean` | `true` | 是否允许旋转 |
| `enableZoom` | `boolean` | `true` | 是否允许缩放 |
| `enablePan` | `boolean` | `true` | 是否允许平移 |
| `enableTilt` | `boolean` | `true` | 是否允许倾斜 |
| `enableFixCenter` | `boolean` | `false` | 是否固定缩放中心 |
| `enableTerrainCollision` | `boolean` | `false` | 是否启用地形碰撞检测 |
| `inertiaTranslate` | `number` | `0.9` | 惯性拖拽系数（0-1） |
| `inertiaZoom` | `number` | `0.8` | 惯性缩放系数（0-1） |
| `minimumZoomDistance` | `number` | `1.0` | 最小缩放距离（米） |
| `maximumZoomDistance` | `number` | `Infinity` | 最大缩放距离（米） |

#### options.clock 时钟配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `currentTime` | `Date` | 当天 10:00 | 初始当前时间 |
| `startTime` | `Date` | 同 currentTime | 开始时间 |
| `stopTime` | `Date` | 同 currentTime | 停止时间 |
| `speed` | `number` | `1` | 时间流速倍率 |
| `tickMode` | `number` | `0` | 模式：`0` 不流逝 / `1` 正常 / `2` 循环 / `3` 限制 |
| `timeZoneOffset` | `number` | `8` | 时区偏移（小时），东八区为 8 |

## 核心属性

Engine 实例创建后，可通过以下属性访问内部子系统：

| 属性 | 类型 | 说明 |
|------|------|------|
| `engine.container` | `HTMLElement` | 容器 DOM 元素 |
| `engine.map` | `EngineMap` | 地图视野系统，控制中心点、缩放、旋转等 |
| `engine.rendering` | `EngineRendering` | 渲染系统，控制渲染循环、后处理等 |
| `engine.event` | `EngineEvent` | 事件系统，处理点击、鼠标等事件 |
| `engine.selection` | `EngineSelection` | 选择器系统 |
| `engine.widgets` | `EngineWidgets` | UI 控件系统 |
| `engine.controller` | `EngineController` | 键鼠交互控制器 |
| `engine.clock` | `EngineClock` | 时钟系统 |
| `engine.renderer` | `WebGLRenderer` | three.js 渲染器（快捷访问） |
| `engine.scene` | `Scene` | three.js 场景（快捷访问） |
| `engine.camera` | `Camera` | three.js 相机（快捷访问） |
| `engine.id` | `number` | 引擎实例唯一 ID |

## 方法列表

### 场景管理

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `add(object)` | `Object3D` | `Object3D` | 将对象添加到渲染场景 |
| `remove(object)` | `Object3D` | - | 将对象从场景移除 |

### 渲染控制

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `requestRender()` | - | - | 请求下一帧执行一次渲染 |
| `addBeforeRenderListener(callback)` | `Function` | - | 添加每帧渲染前回调（内部状态已更新） |
| `removeBeforeRenderListener(callback)` | `Function` | - | 移除渲染前回调 |
| `addPrepareRenderListener(callback)` | `Function` | - | 添加渲染准备阶段回调（最早时机） |
| `removePrepareRenderListener(callback)` | `Function` | - | 移除准备阶段回调 |
| `renderVideo(videoConfig)` | `VideoConfig` | `Promise` | 渲染视频 |

### 视野控制（engine.map）

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `setCenter(center)` | `number[]` | - | 设置地图中心点 `[经度, 纬度]` |
| `setZoom(zoom)` | `number` | - | 设置缩放级别 |
| `setHeading(heading)` | `number` | - | 设置旋转角度（度） |
| `setPitch(pitch)` | `number` | - | 设置俯仰角（度） |
| `setRange(range)` | `number` | - | 设置相机离地距离（米） |
| `lookAt(target, offset)` | `number[], object` | - | 综合设置视野目标和角度，见下方详细说明 |
| `flyTo(target, options)` | `number[], object` | - | 飞行动画过渡到目标位置，见下方详细说明 |
| `getCenter()` | - | `number[]` | 获取当前中心点 |
| `getZoom()` | - | `number` | 获取当前缩放级别 |
| `getHeading()` | - | `number` | 获取当前旋转角度 |
| `getPitch()` | - | `number` | 获取当前俯仰角 |
| `getRange()` | - | `number` | 获取当前相机离地距离 |
| `getGeoBoundingBox()` | - | `{ box: Box3 } \| null` | 获取当前视口的经纬度包围盒，`box.min = Vector3(minLng, minLat, 0)`，`box.max = Vector3(maxLng, maxLat, 0)` |
| `getResolution()` | - | `Vector2` | 获取容器分辨率 |
| `projectArrayCoordinate(input)` | `number[]` | `number[]` | 经纬度转投影坐标 |
| `unprojectArrayCoordinate(input)` | `number[]` | `number[]` | 投影坐标转经纬度 |
| `setMaxRange(range)` | `number` | - | 设置最远视距 |
| `setMinRange(range)` | `number` | - | 设置最近视距 |
| `setBounds(bounds)` | `Array` | - | 设置可拖动视野范围 |

### lookAt 详细说明

立即将视角设置到指定位置和角度，无动画过渡。

```javascript
engine.map.lookAt(target, offset)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `target` | `number[]` | 目标中心点 `[经度, 纬度]` 或 `[经度, 纬度, 高度]` |
| `offset.heading` | `number` | 方位角（度），正北为 0，顺时针递增。默认 0 |
| `offset.pitch` | `number` | 俯仰角（度），0=俯视，90=平视。默认 0 |
| `offset.range` | `number` | 相机到目标的距离（米） |
| `offset.zoom` | `number` | 缩放级别（与 range 二选一，2D 模式用 zoom，3D 模式用 range） |

```javascript
// 示例：从东南方向 60 度俯角看天安门，距离 2000 米
engine.map.lookAt([116.404, 39.915], {
    heading: 135,
    pitch: 60,
    range: 2000,
});
```

### flyTo 详细说明

带动画过渡地飞行到目标位置。引擎会自动选择最短旋转路径（不超过 180 度）。

```javascript
engine.map.flyTo(target, options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `target` | `number[]` | - | 目标中心点 `[经度, 纬度]` 或 `[经度, 纬度, 高度]` |
| `options.heading` | `number` | 当前值 | 目标方位角（度） |
| `options.pitch` | `number` | 当前值 | 目标俯仰角（度） |
| `options.range` | `number` | 当前值 | 目标距离（米） |
| `options.zoom` | `number` | 当前值 | 目标缩放级别 |
| `options.duration` | `number` | `3000` | 动画时长（毫秒） |
| `options.complete` | `Function` | - | 飞行完成回调 |
| `options.cancel` | `Function` | - | 飞行被取消回调（用户拖拽地图会取消飞行） |

```javascript
// 示例：3 秒内飞到上海
engine.map.flyTo([121.473, 31.230], {
    heading: 30,
    pitch: 60,
    range: 5000,
    duration: 3000,
    complete: () => console.log('到达'),
    cancel: () => console.log('飞行被取消'),
});
```


### 生命周期

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `dispose()` | - | - | 销毁引擎，释放全部资源。调用后引擎不再可用 |

## 常见场景

### 场景1：完整初始化示例

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    map: {
        is3DControl: true,
        projection: 'EPSG:3857',
    },
    rendering: {
        sky: null,
        enableAnimationLoop: true,
        features: {
            bloom: {
                enabled: true,
                strength: 0.1,
            },
        },
    },
    widgets: {
        zoom: { enabled: true },
        compass: { enabled: true },
    },
});

// 使用 lookAt 一次性设置视野
engine.map.lookAt([116.404, 39.915], {
    heading: 40,
    pitch: 60,
    range: 2000,
});

// 添加天空和底图
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 17 * 3600;

const mapView = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.BaiduVectorTileProvider({}),
}));
```

### 场景2：引擎创建与销毁（生命周期管理）

```javascript
import * as mapvthree from '@baidu/mapv-three';

let engine = null;

// 创建引擎
function init() {
    if (engine) return;

    engine = new mapvthree.Engine('map_container', {
        rendering: {
            sky: null,
            enableAnimationLoop: true,
        },
    });
    engine.map.setCenter([103.7489, 29.5994]);
    engine.map.setZoom(17);
    engine.map.setPitch(70);

    // 开启泛光
    engine.rendering.bloom.enabled = true;

    const sky = engine.add(new mapvthree.DynamicSky());
    sky.time = 16 * 3600;
}

// 销毁引擎，释放全部资源
function dispose() {
    if (!engine) return;
    engine.dispose();
    engine = null;
}

// 初始化
init();

// 在需要时销毁（如 SPA 路由切换、组件卸载时调用 dispose()）
```

### 场景3：多实例管理

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 创建多个引擎实例，每个绑定不同容器
const engine1 = new mapvthree.Engine(document.getElementById('map1'), {
    map: { is3DControl: true },
    rendering: { sky: null },
});

const engine2 = new mapvthree.Engine(document.getElementById('map2'), {
    map: { is3DControl: true },
    rendering: { sky: null },
});

// 各自独立初始化
function initMap(engine) {
    const center = [119.475, 28.432];
    engine.map.setCenter(center);
    engine.map.setZoom(14);

    const sky = engine.add(new mapvthree.DynamicSky());
    sky.time = 17 * 3600;
}

initMap(engine1);
initMap(engine2);

// 销毁时逐一释放
function disposeAll() {
    engine1.dispose();
    engine2.dispose();
}
```

### 场景4：事件系统与交互

```javascript
import * as mapvthree from '@baidu/mapv-three';
import * as THREE from 'three';

const engine = new mapvthree.Engine('map_container', {
    rendering: {
        enableAnimationLoop: true,
    },
});
engine.map.setCenter([0.01, 0]);
engine.map.setZoom(23);
engine.map.setPitch(80);

// 绑定地图点击事件
engine.map.addEventListener('click', e => {
    console.log('地图被点击', e);
});

// 添加物体并绑定事件
const box = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshStandardMaterial({ color: 0xff0000 })
);
const position = engine.map.projectArrayCoordinate([0.01, 0]);
box.position.set(position[0], position[1], 1);
engine.add(box);

// 物体级别事件监听
box.addEventListener('click', e => {
    console.log('Box 被点击', e);
});
```

### 场景5：飞行动画（flyTo）

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine('map_container', {
    rendering: { enableAnimationLoop: true },
});
engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(14);

// 飞行到目标位置
engine.map.flyTo([121.473, 31.230], {
    heading: 30,
    pitch: 60,
    range: 5000,
    duration: 3000,  // 动画时长（毫秒）
    complete: () => {
        console.log('飞行动画完成');
    },
    cancel: () => {
        console.log('飞行动画被取消');
    },
});
```

## 碰撞检测

引擎提供全局碰撞检测能力，用于处理标注、图标等可视化元素重叠时的自动隐藏。通过 `engine.rendering.collision` 管理，支持所有继承 GeoObject 的可视化组件（Label、SimplePoint、Icon、IconPoint、Circle 等）。

### 基本用法

```javascript
const label = engine.add(new mapvthree.Label({ ... }));
label.dataSource = dataSource;

// 为组件启用碰撞检测
engine.rendering.collision.add(label);
```

### 带配置的用法

```javascript
// 第二个参数为选项，第三个参数为分组名称
engine.rendering.collision.add(label, {
    margin: [10, 10],  // 碰撞包围盒外边距（像素）
}, 'poi-labels');
```

### 全局配置

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `margin` | `number \| number[]` | `[0, 0]` | 碰撞包围盒外边距，数字会转为 `[n, n]` |
| `needsUpdate` | `boolean` | `false` | 标记需要重新计算碰撞 |

```javascript
engine.rendering.collision.margin = [5, 5];
engine.rendering.collision.needsUpdate = true;
```

### 数据属性

可在数据的 properties 中设置碰撞相关属性：

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tolerance` | `number` | `0` | 碰撞容差（像素），值越大越不容易被碰撞隐藏 |
| `rank` | `number` | `0` | 排序权重，值越大优先级越高（越不容易被隐藏） |

```javascript
const data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
data.defineAttribute('tolerance', () => 5);
data.defineAttribute('rank', p => p.priority || 0);
label.dataSource = data;

engine.rendering.collision.add(label);
```

### 分组碰撞

不同分组的对象之间不会产生碰撞，适合将不同类型的标注分离：

```javascript
engine.rendering.collision.add(nameLabel, {}, 'names');
engine.rendering.collision.add(poiLabel, {}, 'pois');
// nameLabel 和 poiLabel 之间不会互相碰撞
```

---

## 注意事项

1. **容器要求**：容器必须是一个有效的 `HTMLElement` 或其 ID 字符串。容器需要有明确的 CSS 宽高，否则渲染区域为 0。
2. **循环渲染**：默认关闭循环渲染（`enableAnimationLoop: false`），属性变更不会自动触发重绘。对于静态场景手动调用 `engine.requestRender()` 即可；有动画时务必开启循环渲染。
3. **dispose 必须调用**：在 SPA 应用中切换页面时，务必调用 `engine.dispose()` 释放 GPU 资源，否则会导致内存泄漏。
4. **多实例隔离**：每个 Engine 实例完全独立，拥有各自的 scene、camera、renderer，互不影响。每个实例需绑定不同的容器。
5. **事件系统**：使用 `addEventListener / removeEventListener` 方式绑定事件。
6. **投影坐标系**：添加 three.js 原生对象（如 `THREE.Mesh`）时，其 position 需使用投影坐标（通过 `engine.map.projectArrayCoordinate()` 转换），而非经纬度。
7. **3D 交互控制**：`is3DControl: true`（默认）启用 3D 交互，支持旋转和倾斜视角。设为 `false` 则为 2D 平面模式。
