# Polyline 折线与 Wall 墙体

mapv-three 提供了三种线型组件和一种墙体组件，用于在 3D 地图上绘制各类线状要素。本文档从源码中提取全部参数，涵盖 SimpleLine、Polyline（含 flat 贴地模式）和 Wall 的完整 API 参考。

---

## 组件总览

| 组件 | 类名 | 说明 |
|------|------|------|
| 简单线 | `mapvthree.SimpleLine` | 1px 基础线条，性能最优，功能最少 |
| 折线 | `mapvthree.Polyline` | 有宽度的线，支持虚线、动画、纹理，通过 `flat` 参数切换屏幕空间线/贴地线 |
| 墙体 | `mapvthree.Wall` | 沿线路径生成无厚度的垂直墙面，支持高度渐变、动画、纹理 |

---

## 快速开始

### 通用数据流

所有线型组件遵循统一的使用模式：创建组件 -> 添加到引擎 -> 设置数据源。

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，需 enableAnimationLoop: true）

// 1. 准备 GeoJSON 数据
const geojsonData = [{
    type: 'Feature',
    geometry: {
        type: 'LineString',
        coordinates: [[116.404, 39.915], [116.405, 39.920], [116.410, 39.918]]
    },
    properties: {}
}];

// 2. 创建数据源（也支持 GeoJSONDataSource.fromURL）
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);

// 3. 创建线组件并绑定数据
const line = engine.add(new mapvthree.Polyline({
    color: '#ff0000',
    lineWidth: 4,
}));
line.dataSource = dataSource;
```

---

## SimpleLine 简单线

最基础的线条渲染对象，仅支持 1px 宽度的线条。适合大量细线的绘制场景，性能开销极低。

### 构造函数

```javascript
const line = engine.add(new mapvthree.SimpleLine(parameters));
```

### 参数表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `string` | `'#ffffff'` | 线条颜色，支持 CSS 颜色值（如 `'rgb(233, 222, 55)'`、`'#ff0000'`） |
| `opacity` | `number` | `1.0` | 透明度，取值范围 0-1 |
| `granularity` | `number` | `0.5` | 线段细分距离（度），仅在 3D 球体模式下生效，用于使线段沿球面弯曲 |

### 属性

| 属性 | 类型 | 读写 | 说明 |
|------|------|------|------|
| `color` | `Color` | 读/写 | 线条颜色，运行时可动态修改 |
| `dataSource` | `DataSource` | 读/写 | 数据源 |

### 示例

```javascript
const line = engine.add(new mapvthree.SimpleLine({
    color: 'rgb(233, 222, 55)',
}));
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
line.dataSource = dataSource;
```

---

## Polyline 折线

功能最丰富的线组件。通过 `flat` 参数区分两种渲染模式：

- **`flat: false`（默认）** — 屏幕空间拉伸线（PolylineInternal），线宽以屏幕像素为单位，适用于普通折线
- **`flat: true`** — XY 平面拉伸线（FatLineInternal），线宽以地理坐标为单位，支持贴地渲染、动画、纹理贴图等高级特性

### 构造函数

```javascript
const polyline = engine.add(new mapvthree.Polyline(parameters));
```

`Polyline` 继承自 `THREE.Group`，内部使用 Proxy 将属性和方法调用透明转发到底层渲染对象。

### 通用参数表（两种模式共用）

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `flat` | `boolean` | `false` | 渲染模式开关。`false`=屏幕空间线，`true`=贴地线（XY 平面拉伸） |
| `color` | `string` | `'#00ffff'` | 线条颜色，支持十六进制或 CSS 颜色值 |
| `lineWidth` | `number` | `4` | 线条宽度 |
| `height` | `number` | - | 线条高度偏移，用于 3D 效果 |
| `opacity` | `number` | `1` | 透明度，取值范围 0-1 |
| `alphaTest` | `number` | `0` | 透明度剔除阈值，小于该值的像素将不被渲染 |
| `vertexColors` | `boolean` | `false` | 是否启用顶点颜色。为 `true` 时通过 `dataSource.defineAttribute('color', fn)` 设置逐点颜色 |
| `emissive` | `string` | - | 自发光颜色 |
| `map` | `string` | - | 纹理贴图路径 |
| `dashed` | `boolean` | `false` | 是否渲染虚线，需同时设置 `transparent: true` 才能生效 |
| `dashArray` | `number` | `20` | 每段虚线（实线+空白部分）的总长度（仅 `flat: true` 时可调） |
| `dashOffset` | `number` | `0` | 虚线起始绘制位置的偏移量（仅 `flat: true` 时可调） |
| `dashRatio` | `number` | `0.5` | 实线部分占每段虚线长度的比例，取值范围 0-1（仅 `flat: true` 时可调） |
| `isCurve` | `boolean` | `false` | 是否自动生成贝塞尔曲线。开启后仅取线数据的首尾点生成 3D 贝塞尔曲线，中间点失效 |
| `transparent` | `boolean` | - | 是否启用透明 |

### flat 模式专有参数（FatLineInternal）

当 `flat: true` 时，以下参数可用：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `keepSize` | `boolean` | `true` | 是否保持像素宽度（缩放时线宽不变） |
| `lineJoin` | `'miter' \| 'bevel' \| 'round'` | `'bevel'` | 线段拐角样式 |
| `lineCap` | `'butt' \| 'square' \| 'round'` | `'butt'` | 线段端点样式 |
| `miterLimit` | `number` | 线宽的2倍 | 当 `lineJoin='miter'` 时，超过该长度自动变平角 |
| `mapSrc` | `string` | - | 纹理贴图路径 |
| `mapGap` | `number` | `50` | 纹理贴图间隔（相对于线宽的倍数），默认值过大可能导致看不到纹理，建议设为 `0` |
| `antialias` | `boolean` | - | 是否启用抗锯齿 |
| `raycastBuffer` | `number` | `0` | 拾取缓冲区，上调该值可扩大鼠标拾取范围 |
| `keepDashLength` | `boolean` | `true` | 虚线长度是否按像素单位绘制 |
| `useMeterUnits` | `boolean` | - | 是否使用米为单位 |
| `mapBlend` | - | - | 贴图混合模式 |
| `needRound` | `boolean` | - | 是否需要圆角 |

### 动画参数（仅 flat 模式）

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enableAnimation` | `boolean` | `false` | 是否启用线动画 |
| `enableAnimationChaos` | `boolean` | `false` | 是否开启不规则动画（每条线动画不同步） |
| `animationSpeed` | `number` | `1` | 动画速度 |
| `animationTailType` | `1 \| 2` | - | 动画尾迹类型：`1`=按线长度比例，`2`=按固定长度 |
| `animationTailRatio` | `number` | `0.2` | 尾迹长度占整条线的比例（`animationTailType=1` 时使用） |
| `animationTailLength` | `number` | `100` | 尾迹固定长度（`animationTailType=2` 时使用） |
| `animationIdle` | `number` | `1000` | 动画间隔时间，单位毫秒 |
| `animationInterval` | `number` | `0` | 动画重复间隔比例，值越大间隔越大，`0` 表示不重复 |

### 属性列表

以下属性均可在创建后动态读写：

| 属性 | 类型 | 说明 |
|------|------|------|
| `lineWidth` | `number` | 线条宽度 |
| `color` | `string` | 线条颜色 |
| `opacity` | `number` | 透明度 |
| `dashed` | `boolean` | 虚线开关 |
| `dashArray` | `number` | 虚线段长度 |
| `dashOffset` | `number` | 虚线偏移 |
| `dashRatio` | `number` | 实线比例 |
| `keepSize` | `boolean` | 保持像素宽度（flat 模式） |
| `enableAnimation` | `boolean` | 动画开关（flat 模式） |
| `animationSpeed` | `number` | 动画速度（flat 模式） |
| `dataSource` | `DataSource` | 数据源 |

### 示例：屏幕空间折线

```javascript
const polyline = engine.add(new mapvthree.Polyline({
    flat: false,
    color: '#ff0000',
    lineWidth: 4,
    dashed: true,
    dashArray: 20,
    dashRatio: 0.5,
}));
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
polyline.dataSource = dataSource;
```

### 示例：贴地粗线

```javascript
const fatLine = engine.add(new mapvthree.Polyline({
    flat: true,
    color: 'rgb(233, 222, 55)',
    lineWidth: 10,
    keepSize: true,
    antialias: false,
    transparent: true,
}));
const dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/line.geojson');
fatLine.dataSource = dataSource;
```

### 示例：贴地粗线 + 顶点颜色

```javascript
const line = engine.add(new mapvthree.Polyline({
    flat: true,
    lineWidth: 10,
    vertexColors: true,
    keepSize: true,
}));
const dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/line.geojson');
// 为每条线定义随机颜色
dataSource.defineAttribute('color', p => Math.random() * 0xffffff);
line.dataSource = dataSource;
```

### 示例：虚线效果

```javascript
const dashedLine = engine.add(new mapvthree.Polyline({
    flat: true,
    color: '#00ffff',
    lineWidth: 6,
    keepSize: true,
    transparent: true,    // 虚线必须开启透明
    dashed: true,         // 启用虚线
    dashArray: 20,        // 每段虚线总长度
    dashOffset: 0,        // 偏移量
    dashRatio: 0.5,       // 实线占比 50%
}));
dashedLine.dataSource = dataSource;
```

---

## 飞线效果

飞线是 Polyline（flat 模式）的一种典型应用，通过 `isCurve` + `enableAnimation` 组合实现从起点到终点的弧线流动动画。

### 原理

- `isCurve: true` — 自动将首尾两点生成 3D 贝塞尔弧线
- `enableAnimation: true` — 启用流动动画
- 通常叠加两层线：底线（半透明静态弧线）+ 飞线（高亮动画弧线）

### 完整示例

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {DoubleSide, Color} from 'three';

// 引擎初始化（略，需 enableAnimationLoop: true, bloom.enabled: true）
const center = [106.472739, 29.561524];

// 目标点
const targets = [
    [106.476, 29.565],
    [106.469, 29.558],
    [106.480, 29.560],
];

// 构造飞线 GeoJSON（起点 -> 终点）
const flylineGeoJSON = targets.map(target => ({
    geometry: {
        type: 'LineString',
        coordinates: [[center[0], center[1], 0.0], target],
    },
}));
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(flylineGeoJSON);

// 飞线层：高亮、带拖尾动画
const flyline = engine.add(new mapvthree.Polyline({
    flat: true,
    isCurve: true,                        // 自动生成贝塞尔弧线
    color: new Color(10.2, 10.4, 0.2),    // HDR 颜色配合 bloom
    lineWidth: 6,
    keepSize: true,
    enableAnimation: true,                 // 开启流动动画
    enableAnimationChaos: true,            // 每条线动画不同步
    animationTailType: 2,                  // 按固定长度拖尾
    animationTailLength: 20,               // 拖尾长度
    animationSpeed: 0.1,                   // 动画速度
    animationIdle: 100,                    // 动画间隔（毫秒）
    animationInterval: 10,                 // 重复间隔比例
}));
flyline.material.side = DoubleSide;
flyline.dataSource = dataSource;

// 底线层：半透明静态弧线
const baseLine = engine.add(new mapvthree.Polyline({
    flat: true,
    isCurve: true,
    color: 0x00ff00,
    emissive: new Color(0.2, 0.4, 0.2),
    opacity: 0.2,
    keepSize: true,
    lineWidth: 5,
}));
baseLine.material.side = DoubleSide;
baseLine.dataSource = dataSource;
```

---

## Wall 墙体

沿线路径生成垂直墙面的组件，支持高度、透明度渐变和多种动画效果。适用于围栏、边界高亮、区域标识等场景。

### 构造函数

```javascript
const wall = engine.add(new mapvthree.Wall(parameters));
```

### 参数表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `height` | `number` | `100` | 墙的高度 |
| `color` | `string` | `'#00ffff'` | 墙的颜色 |
| `vertexColors` | `boolean` | `false` | 是否通过数据携带颜色配置 |
| `map` | `string` | - | 纹理贴图路径 |
| `mapScale` | `number \| number[]` | `1` | 纹理贴图缩放系数 |
| `opacity` | `number` | `1` | 墙整体透明度 |
| `minOpacity` | `number` | `0` | 墙最低透明度（用于渐变效果） |
| `maxOpacity` | `number` | `1` | 墙最高透明度（用于渐变效果） |
| `transparent` | `boolean` | - | 是否启用透明 |
| `emissive` | `string` | - | 自发光颜色 |

### 动画参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enableAnimation` | `boolean` | `false` | 是否开启墙动画 |
| `animationSpeed` | `number` | `1` | 动画速度 |
| `animationTailType` | `1 \| 2 \| 3 \| 4` | `3` | 动画类型 |
| `animationTailRatio` | `number` | `0.2` | 拖尾长度比例（`animationTailType=1` 时使用） |
| `animationTailLength` | `number` | `100` | 拖尾固定长度（`animationTailType=2` 时使用） |
| `animationIdle` | `number` | `1000` | 动画间隔时间，单位毫秒 |
| `animationRatio` | `number` | `0.5` | 条纹动画实线占比（`animationTailType=4` 时生效） |
| `animationBales` | `number` | `5` | 条纹动画显示组数（`animationTailType=4` 时生效） |

**animationTailType 取值说明：**

| 值 | 说明 | 搭配参数 |
|----|------|----------|
| `1` | 按墙长度比例流动 | `animationTailRatio` |
| `2` | 按固定长度流动 | `animationTailLength` |
| `3` | 垂直方向动画（从底部到顶部流动） | - |
| `4` | 条纹上升动画 | `animationRatio`, `animationBales` |

### 属性列表

以下属性均可在创建后动态读写：

| 属性 | 类型 | 说明 |
|------|------|------|
| `height` | `number` | 墙的高度（修改会触发几何体重建） |
| `color` | `string` | 墙的颜色 |
| `opacity` | `number` | 透明度 |
| `minOpacity` | `number` | 最低透明度 |
| `maxOpacity` | `number` | 最高透明度 |
| `enableAnimation` | `boolean` | 动画开关 |
| `animationSpeed` | `number` | 动画速度 |
| `animationTailType` | `number` | 动画类型 |
| `map` | `string` | 纹理贴图 |
| `mapScale` | `number \| number[]` | 贴图缩放 |
| `dataSource` | `DataSource` | 数据源 |

### 示例：基础墙体

```javascript
const wall = engine.add(new mapvthree.Wall({
    height: 200,
    color: '#00ffff',
    opacity: 0.8,
}));
const data = [{
    type: 'Feature',
    geometry: {
        type: 'LineString',
        coordinates: [[116.404, 39.915], [116.405, 39.920], [116.410, 39.918]]
    },
    properties: {}
}];
wall.dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(data);
```

### 示例：垂直流光墙

```javascript
const wall = engine.add(new mapvthree.Wall({
    height: 50,
    color: '#ff0',
    transparent: true,
    enableAnimation: true,
    animationSpeed: 2.3,
    animationTailType: 3,    // 垂直方向动画
    minOpacity: 0.1,
}));
wall.dataSource = dataSource;
```

### 示例：条纹上升动画墙

```javascript
const wall = engine.add(new mapvthree.Wall({
    height: 100,
    color: '#ff0',
    transparent: true,
    enableAnimation: true,
    animationSpeed: 2,
    animationTailType: 4,    // 条纹上升
    animationRatio: 0.5,     // 实线占比
    animationBales: 5,       // 条纹组数
}));
wall.dataSource = dataSource;
```

### 示例：带纹理的围栏墙

```javascript
const wall = engine.add(new mapvthree.Wall({
    height: 50,
    color: '#ff0',
    transparent: true,
    enableAnimation: true,
    animationSpeed: 2.3,
    animationTailType: 4,
    map: 'website_assets/textures/fence.png',
}));
wall.dataSource = dataSource;
```

---

## 事件交互

Polyline（两种模式）和 Wall 均支持事件拾取：

```javascript
line.addEventListener('click', e => {
    console.log('线被点击', e);
    console.log('点击的数据索引', e.index);
    console.log('点击的世界坐标', e.point);
});

line.addEventListener('mousemove', e => {
    console.log('鼠标移入线', e);
});
```

FatLineInternal 提供 `raycastBuffer` 参数用于扩大拾取范围：

```javascript
const line = engine.add(new mapvthree.Polyline({
    flat: true,
    lineWidth: 4,
    raycastBuffer: 5,  // 额外增加 5 个单位的拾取缓冲
}));
```

---

## 注意事项

1. **动画必须开启动画循环** — 使用动画参数（`enableAnimation`）时，引擎必须设置 `rendering.enableAnimationLoop: true`，否则动画不会播放。

2. **flat 模式与非 flat 模式的差异** — `flat: false`（默认）的线宽以屏幕像素为单位，始终等宽；`flat: true` 的线宽以地理坐标为单位，缩放时会变化（除非设置 `keepSize: true`）。非 flat 模式不支持动画参数。虚线方面，`flat: true` 的虚线功能完整（`dashArray`、`dashRatio`、`dashOffset` 均生效），`flat: false` 的虚线为固定效果（段长和占比不可调）。

3. **isCurve 仅取首尾点** — 开启 `isCurve` 后，线数据中除首尾点之外的中间坐标将被忽略，仅用首尾点生成贝塞尔曲线。

4. **bloom 与 HDR 颜色** — 飞线效果通常需要配合 `engine.rendering.bloom.enabled = true` 和 HDR 颜色值（分量大于 1，如 `new Color(10, 10, 0)`）来实现发光效果。

5. **Polyline 的代理机制** — `Polyline` 继承自 `THREE.Group`，通过 `Proxy` 将属性读写和方法调用透传到内部的 `PolylineInternal` 或 `FatLineInternal` 实例，因此可以直接在 `Polyline` 实例上访问所有内部属性。

6. **Wall 的 height 属性** — 修改 `height` 会触发几何体的完整重建（通过 `defineGeometryUpdateProxyProperties` 实现），有一定性能开销。

7. **SimpleLine 不支持宽度设置** — SimpleLine 的线宽固定为 1px（受 WebGL 限制），如需可变宽度请使用 Polyline。

8. **数据源格式** — 所有组件接受 `LineString` 类型的 GeoJSON 数据。Wall 同样使用 `LineString`，沿路径向上拉伸生成墙面。
