# EffectPoint 特效点、Circle 圆形与 Pillar 柱体

## 概述

本文档涵盖 mapv-three 中三种基于实例化渲染的点状可视化组件：**EffectPoint**（特效点）、**Circle**（圆形）和 **Pillar**（柱体）。三者均继承自 `GeoInstancedPointMesh`，支持通过实例化技术高效渲染大量几何体。

> **注意**：所有带动画效果的组件均需在引擎初始化时设置 `rendering.enableAnimationLoop = true`。

---

## 一、EffectPoint 特效点

`EffectPoint` 是一个多态特效组件，通过 `type` 字段自动创建不同类型的特效渲染效果。

### 基本用法

```javascript
// 引擎初始化（略，需 enableAnimationLoop: true）

const effectPoint = engine.add(new mapvthree.EffectPoint({
    type: 'Fan',
    color: '#eac654',
    size: 50,
}));

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
effectPoint.dataSource = dataSource;
```

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `string` | `'Bubble'` | 特效类型，可选值见下方类型列表 |
| `color` | `string` | `'rgb(255, 0, 0)'` | 特效颜色，支持十六进制、RGB、RGBA 等格式 |
| `size` | `number` | `100` | 特效尺寸 |
| `duration` | `number` | `5000` | 动画间隔时长（毫秒） |
| `opacity` | `number` | `1` | 整体透明度，取值 0-1 |
| `keepSize` | `boolean` | `true` | 是否保持屏幕像素大小不随缩放变化 |
| `vertexColors` | `boolean` | `false` | 是否通过数据携带逐实例颜色 |
| `vertexSizes` | `boolean` | `false` | 是否通过数据携带逐实例尺寸 |
| `segmentAngle` | `number` | `0.25 * Math.PI` | 雷达扇形弧度值，仅对 `Radar` 类型有效 |
| `sideColor` | `string` | `[0.87, 0.98, 1.00]` | 最外圈底色，仅对 `RadarLayered` 类型有效 |

### 运行时可修改属性

通过材质代理机制，以下属性支持在创建后动态修改：

```javascript
effectPoint.color = '#ff0000';
effectPoint.size = 80;
effectPoint.duration = 3000;
effectPoint.opacity = 0.5;
effectPoint.segmentAngle = Math.PI;
```

完整列表：`color`、`size`、`duration`、`size3`、`height`、`opacity`、`vertexColors`、`segmentAngle`、`sideColor`、`vertexSizes`。

### 支持的 type 类型

#### 1. Bubble（气泡） — 默认类型

向外扩散的气泡脉冲效果。不指定 `type` 时默认使用此类型。

```javascript
const bubble = engine.add(new mapvthree.EffectPoint({
    color: 'rgba(90, 160, 117, 1.0)',
    size: 100,
    duration: 5000,
    keepSize: true,
}));
```

#### 2. Breath（呼吸）

周期性缩放的呼吸灯效果。

```javascript
const breath = engine.add(new mapvthree.EffectPoint({
    type: 'Breath',
    color: 'rgba(90, 160, 117, 1.0)',
    size: 100,
    duration: 5000,
    keepSize: true,
}));
```

#### 3. Wave（波纹）

向外扩散的波纹效果。

```javascript
const wave = engine.add(new mapvthree.EffectPoint({
    type: 'Wave',
    color: 'rgba(90, 160, 117, 1.0)',
    size: 100,
    duration: 5000,
    keepSize: true,
}));
```

#### 4. Fan（扇形）

静态扇形雷达效果。

```javascript
const fan = engine.add(new mapvthree.EffectPoint({
    type: 'Fan',
    color: '#eac654',
    size: 50,
}));
```

#### 5. Radar（旋转雷达）

带旋转动画的雷达扫描效果。

**特有参数**：
- `segmentAngle` — 扫描扇形的弧度值，默认 `0.25 * Math.PI`

```javascript
const radar = engine.add(new mapvthree.EffectPoint({
    type: 'Radar',
    color: '#eac654',
    segmentAngle: Math.PI,
    size: 50,
}));
```

#### 6. RadarLayered（分层雷达）

带分层环纹的静态雷达效果。

**特有参数**：
- `sideColor` — 最外圈底色，支持颜色数组 `[r, g, b]`（值域 0-1）

```javascript
const radarLayered = engine.add(new mapvthree.EffectPoint({
    type: 'RadarLayered',
    color: '#22ee55',
    size: 50,
}));
radarLayered.material.sideColor = [1.0, 0.78, 0.0];
```

#### 7. RadarSpread（扩散雷达）

向外扩散的雷达环效果。

```javascript
const radarSpread = engine.add(new mapvthree.EffectPoint({
    type: 'RadarSpread',
    color: '#c53de1',
    size: 50,
}));
```

### 逐实例数据

通过 `vertexColors` 和 `vertexSizes` 启用后，可在数据源中为每个实例定义独立的颜色和尺寸：

```javascript
const effectPoint = engine.add(new mapvthree.EffectPoint({
    type: 'Bubble',
    vertexColors: true,
    size: 100,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
data.defineAttribute('color', p => Math.random() * 0xffffff);
effectPoint.dataSource = data;
```

### 自定义实例变换

可覆盖 `getInstanceLocalMatrix` 方法为每个实例提供独立的变换矩阵：

```javascript
effectPoint.getInstanceLocalMatrix = (coordinates, dataItem, index) => {
    const matrix = new THREE.Matrix4();
    const scale = dataItem.attributes.count;
    matrix.makeScale(scale, scale, scale);
    return matrix;
};
```

---

## 二、Circle 圆形

`Circle` 用于高效绘制大量圆形标记，支持填充、边框、渐变等样式。

### 基本用法

```javascript
const circle = engine.add(new mapvthree.Circle({
    color: '#f4f27a',
    borderWidth: 20,
    borderColor: '#b73145',
    opacity: 0.8,
    vertexSizes: true,
}));

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
circle.dataSource = dataSource;
```

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `string` | `'#ff8000'` | 圆形填充颜色 |
| `size` | `number` | `100` | 圆形大小（`keepSize` 为 true 时单位为像素，否则为米） |
| `opacity` | `number` | `1` | 整体透明度，取值 0-1 |
| `type` | `string` | `'Default'` | 渲染类型，可选 `'Default'` 或 `'Gradient'` |
| `borderWidth` | `number` | `1` | 边框宽度（`keepSize` 为 true 时单位为像素，否则为米） |
| `borderColor` | `string` | `'#00ff00'` | 边框颜色 |
| `borderOpacity` | `number` | `1` | 边框透明度，取值 0-1 |
| `fillOpacity` | `number` | `1` | 填充区域透明度，取值 0-1（设为 0 可实现空心环效果） |
| `vertexColors` | `boolean` | `false` | 是否启用逐实例顶点颜色 |
| `vertexSizes` | `boolean` | `false` | 是否启用逐实例顶点大小 |
| `keepSize` | `boolean` | - | 是否保持屏幕像素大小不随缩放变化 |
| `transparent` | `boolean` | - | 是否开启透明渲染 |

### 运行时可修改属性

```javascript
circle.color = '#ff0000';
circle.size = 200;
circle.opacity = 0.5;
circle.borderWidth = 10;
circle.borderColor = '#000000';
circle.borderOpacity = 0.8;
circle.fillOpacity = 0;  // 空心环
```

### 支持的 type 类型

#### 1. Default（默认）

标准实心圆形，带可选边框。

```javascript
const circle = engine.add(new mapvthree.Circle({
    color: '#0000ff',
    borderWidth: 20,
    borderColor: '#000000',
    keepSize: true,
    transparent: true,
    opacity: 0.5,
}));
```

#### 2. Gradient（渐变）

从中心向边缘渐变透明的圆形效果。

```javascript
const circleGradient = engine.add(new mapvthree.Circle({
    color: '#f00',
    size: 200,
    borderWidth: 20,
    type: 'Gradient',
    transparent: true,
    opacity: 0.1,
}));
```

### 逐实例数据

```javascript
const circle = engine.add(new mapvthree.Circle({
    vertexColors: true,
    vertexSizes: true,
    keepSize: true,
}));

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
dataSource.defineAttribute('color', p => Math.random() * 0xffffff);
dataSource.defineAttribute('size', p => 100 + Math.random() * 100);
circle.dataSource = dataSource;
```

### 发光效果（Emissive）

Circle 支持自发光属性，可配合 bloom 后处理实现光晕效果：

```javascript
engine.rendering.bloom.enabled = true;

circle.material.emissiveEnabled = true;
circle.material.emissiveIntensity = 1;
circle.material.emissive = new Color(0.01, 0.01, 0.1);
```

### 碰撞检测

Circle 内置 `collisionTest` 方法，在 `keepSize` 模式下返回实例的屏幕占用区域，可用于避让和拾取计算。

---

## 三、Pillar 柱体

`Pillar` 用于渲染柱状体和锥形体，适合柱状图、3D 数据可视化等场景。

### 基本用法

```javascript
// 引擎初始化（略，需 enableAnimationLoop: true）

const pillar = engine.add(new mapvthree.Pillar({
    height: 300,
    radialSegments: 32,
    openEnded: false,
    radius: 20,
    color: '#ffff00',
    vertexHeights: true,
    gradient: {
        0.1: '#73d19a',
        1.0: '#d15d3b',
    },
}));

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
dataSource.defineAttribute('height', p => 100 + 200 * Math.random());
pillar.dataSource = dataSource;
```

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `shape` | `string` | `'pillar'` | 形状类型，可选 `'pillar'`（柱形）或 `'cone'`（锥形） |
| `color` | `string \| Array` | `[80, 20, 170, 0.8]` | 柱体颜色，支持十六进制、颜色名称或 RGBA 数组 |
| `opacity` | `number` | `1` | 整体透明度，取值 0-1 |
| `height` | `number` | `1` | 柱体高度（米） |
| `radius` | `number` | `6` | 柱体半径（米），同时设置 `radiusTop` 和 `radiusBottom` |
| `radiusTop` | `number` | `radius` 值 | 顶部半径（柱形模式下有效） |
| `radiusBottom` | `number` | `radius` 值 | 底部半径 |
| `radialSegments` | `number` | `4` | 柱体横截面边数（4=方柱，32=近似圆柱） |
| `heightSegments` | `number` | `1` | 柱体高度方向分段数 |
| `openEnded` | `boolean` | `true` | 柱体顶底面是否打开（`true` 为无顶盖底盖） |
| `vertexHeights` | `boolean` | `false` | 是否通过数据携带逐实例高度 |
| `vertexSizes` | `boolean` | `false` | 是否通过数据携带逐实例大小 |
| `gradient` | `Object` | - | 颜色渐变配置，键为 0-1 的位置，值为颜色 |
| `colorMode` | `string` | `'gradient'` | 颜色模式，可选 `'gradient'`（渐变）或 `'band'`（颜色分带） |
| `heatmap` | `boolean` | `true` | 是否开启热力模式 |

### 运行时可修改属性

以下属性可通过材质代理在运行时动态修改：

```javascript
pillar.material.color = '#ff0000';
pillar.material.opacity = 0.8;
pillar.material.heatmap = false;
pillar.material.gradient = { 0.0: '#0000ff', 0.5: '#00ff00', 1.0: '#ff0000' };
pillar.material.bandColors = [...]; // colorMode 为 'band' 时使用
```

### 支持的 shape 类型

#### 1. pillar（柱形） — 默认

标准柱状体，支持设置 `radiusTop` 和 `radiusBottom` 来控制上下半径。

```javascript
const pillar = engine.add(new mapvthree.Pillar({
    shape: 'pillar',
    height: 300,
    radius: 20,
    radialSegments: 32,
    openEnded: false,
    color: '#ffff00',
}));
```

#### 2. cone（锥形）

从底部向顶部收缩的锥形体，顶部自动封闭。

```javascript
const cone = engine.add(new mapvthree.Pillar({
    shape: 'cone',
    height: 80,
    radius: 15,
    color: '#00ff00',
}));
```

### 颜色渐变

Pillar 支持沿高度方向的颜色渐变，通过 `gradient` 参数配置：

```javascript
const pillar = engine.add(new mapvthree.Pillar({
    height: 300,
    vertexHeights: true,
    gradient: {
        0.0: '#73d19a',  // 底部颜色
        0.5: '#eac654',  // 中部颜色
        1.0: '#d15d3b',  // 顶部颜色
    },
}));
```

也可使用 `colorMode: 'band'` 配合 `bandColors` 实现颜色分带效果。

### 逐实例数据

通过 `vertexHeights` 和 `vertexSizes` 为每个实例定义独立的高度和大小：

```javascript
const pillar = engine.add(new mapvthree.Pillar({
    vertexHeights: true,
    radius: 20,
    radialSegments: 32,
    gradient: { 0.1: '#73d19a', 1.0: '#d15d3b' },
}));

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
dataSource.defineAttribute('height', p => 100 + 200 * Math.random());
pillar.dataSource = dataSource;
```

---

## 四、通用说明

### 数据源绑定

三个组件均通过 `dataSource` 属性绑定 `GeoJSONDataSource`，使用 `defineAttribute` 定义逐实例属性（如 `color`、`size`、`height`）。

### 引擎初始化要求

- 使用动画类型（Bubble、Breath、Wave、Radar、RadarSpread）时，必须开启 `rendering.enableAnimationLoop: true`。

### 继承关系

```
GeoInstancedMesh
  └── GeoInstancedPointMesh
        ├── EffectPoint
        ├── Circle
        └── Pillar
```

三者均支持 `GeoInstancedPointMesh` 提供的逐实例颜色（`vertexColors`）和逐实例大小（`vertexSizes`）能力。Pillar 额外支持逐实例高度（`vertexHeights`）。

### 动画属性（通过材质设置）

所有组件均继承自 `InstancedEffectPointMaterial`，支持以下动画属性：

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `animationRotate` | `boolean` | `false` | 是否启用旋转动画 |
| `animationRotatePeriod` | `number` | `4000` | 旋转动画周期（毫秒） |
| `animationJump` | `boolean` | `false` | 是否启用跳跃动画 |
| `animationJumpPeriod` | `number` | `4000` | 跳跃动画周期（毫秒） |
| `animationJumpHeight` | `number` | `30` | 跳跃高度 |
| `animationScale` | `boolean` | `false` | 是否启用缩放动画 |
| `animationBreath` | `boolean` | `false` | 是否启用呼吸动画 |
| `animationPeriod` | `number` | `4000` | 通用动画周期（毫秒） |
| `animationPeriodOffset` | `boolean` | `false` | 是否对每个实例的动画添加随机偏移 |

```javascript
// 示例：为 Circle 添加旋转动画
circle.material.animationRotate = true;
circle.material.animationRotatePeriod = 2000;
```
