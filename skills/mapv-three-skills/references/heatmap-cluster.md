# Heatmap 热力图与 ClusterPoint 聚合点

---

## 目录

- [Heatmap - 2D 热力图](#heatmap---2d-热力图)
- [Heatmap3D - 3D 热力图](#heatmap3d---3d-热力图)
- [ClusterPoint - 聚合点](#clusterpoint---聚合点)

---

## Heatmap - 2D 热力图

`mapvthree.Heatmap` 用于在地图上展示数据密度分布的 2D 热力图。支持自定义颜色渐变、透明度、半径等属性，可以高效渲染大量数据点。继承自 `GeoMesh`。

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `gradient` | `Object` | `{0: 'rgba(0,0,255,1)', 0.3: 'rgba(0,255,0,1)', 0.6: 'rgba(255,255,0,1)', 1: 'rgba(255,0,0,1)'}` | 热力渐变颜色配置，格式为 `{stop: 'color'}`，stop 取值范围 0~1 |
| `opacity` | `number` | `1` | 透明度整体系数 |
| `minValue` | `number` | `0` | 最小热力值 |
| `maxValue` | `number` | `1` | 最大热力值 |
| `radius` | `number` | `100` | 热力绘制半径 |
| `keepSize` | `boolean` | `false` | 是否保持大小（即按像素为单位绘制）。设为 `true` 时，热力点半径不随缩放变化 |
| `attenuateMValueFactor` | `number` | `0` | 径向渐变速度，值越大渐变越快 |

### 属性（getter/setter）

| 属性 | 类型 | 读写 | 说明 |
|------|------|------|------|
| `dataSource` | `DataSource` | get / set | 热力图数据源。数据源中应包含 `position`（坐标）和 `count`（权重值）字段 |
| `gradient` | `Object` | set | 热力渐变颜色配置 |
| `radius` | `number` | get / set | 热力绘制半径 |
| `minValue` | `number` | get / set | 最小热力值 |
| `maxValue` | `number` | get / set | 最大热力值 |
| `opacity` | `number` | get / set | 透明度 |
| `keepSize` | `boolean` | get / set | 是否保持大小（按像素为单位绘制） |
| `attenuateMValueFactor` | `number` | set | 径向渐变速度 |
| `isHeatmap` | `boolean` | readonly | 始终为 `true`，用于类型判断 |

### 方法

| 方法 | 说明 |
|------|------|
| `dispose()` | 释放热力图资源，包括材质、几何体和渲染目标 |

### 基本用法

```javascript
const heatmap = engine.add(new mapvthree.Heatmap({
    radius: 30,
    keepSize: true,
    maxValue: 10,
    attenuateMValueFactor: 0.9,
}));

// 设置数据源
let data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
data.defineAttribute('count');
heatmap.dataSource = data;
```

### 动态修改属性

```javascript
// 运行时修改渐变色
heatmap.gradient = {
    0: 'red',
    0.5: 'white',
    1: 'yellow',
};

// 根据缩放等级动态调整半径
engine.addPrepareRenderListener(() => {
    const zoom = engine.map.getZoom();
    heatmap.radius = 10 * THREE.MathUtils.clamp((zoom - 1) * 0.2, 0.3, 1);
});
```

---

## Heatmap3D - 3D 热力图

`mapvthree.Heatmap3D` 用于在地图上展示立体热力图效果，数据点会以三维柱状形态呈现。继承自 `GeoMesh`。

> **注意：3D 热力图目前只适用于小面积覆盖范围（区、乡镇级别以下），在大面积覆盖场景下（省、市级别以上）会有明显性能问题。** 内部使用 Canvas 绘制灰度图再映射为纹理，画布尺寸上限为 2048px，大面积范围会导致精度下降和性能恶化。

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `gradient` | `Object` | `{0.4: 'rgba(0,0,255,1)', 0.6: 'rgba(0,255,0,1)', 0.8: 'rgba(255,255,0,1)', 1: 'rgba(255,0,0,1)'}` | 热力渐变颜色配置 |
| `opacity` | `number` | `1` | 透明度整体系数 |
| `heightRatio` | `number` | `100` | 高度系数，控制 3D 热力柱的高度 |
| `maxValue` | `number` | `1` | 最大热力值 |
| `radius` | `number` | `100` | 热力绘制半径 |

### 属性（getter/setter）

| 属性 | 类型 | 读写 | 说明 |
|------|------|------|------|
| `dataSource` | `DataSource` | get / set | 继承自 `GeoMesh`，设置后自动触发 3D 热力图重建 |
| `gradient` | `Object` | get / set | 热力渐变颜色配置 |
| `radius` | `number` | get / set | 热力绘制半径 |
| `maxValue` | `number` | get / set | 最大热力值 |
| `opacity` | `number` | get / set | 透明度整体系数（通过材质代理） |
| `heightRatio` | `number` | get / set | 高度系数（通过材质代理） |
| `isHeatmap3D` | `boolean` | readonly | 始终为 `true`，用于类型判断 |

### 基本用法

```javascript
const heatmap3d = engine.add(new mapvthree.Heatmap3D({
    radius: 100,
    maxValue: 2,
    heightRatio: 1200,
}));

let data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
heatmap3d.dataSource = data;
```

### 地球模式支持

Heatmap3D 同时支持平面地图和球体（Globe）模式。在球体模式下，会自动将数据点投影到地球表面的切平面上进行计算，并正确设置旋转以贴合球面。

---

## ClusterPoint - 聚合点

`mapvthree.ClusterPoint` 用于将大量点要素进行聚合展示。它能根据当前地图缩放级别和视野范围，自动将距离较近的点聚合为一个聚合点，并显示聚合数量。继承自 `PointGroup`。

ClusterPoint 内部使用 [Supercluster](https://github.com/mapbox/supercluster) 算法进行聚合计算，支持通过 `addComponent` 方法添加子组件来自定义聚合点的可视化效果。

### 构造参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `cluster` | `Object` | — | 聚合算法配置 |
| `cluster.maxZoom` | `number` | `18` | 生成聚合数据的最大缩放级别 |
| `cluster.minZoom` | `number` | `5` | 生成聚合数据的最小缩放级别 |
| `cluster.radius` | `number` | `50` | 聚合半径，单位 px |
| `icon` | `Object \| null` | — | 图标配置，设为 `null` 则不显示图标 |
| `icon.width` | `number` | `30` | 图标宽度（px） |
| `icon.height` | `number` | `30` | 图标高度（px） |
| `icon.mapSrc` | `string` | — | 图标图片地址 |
| `label` | `Object` | — | 标签配置 |
| `label.fillStyle` | `string` | `'#ccc'` | 文字颜色 |
| `label.fontSize` | `number` | `16` | 文字大小（px） |
| `label.flat` | `boolean` | `false` | 是否平面化显示 |
| `label.width` | `number` | — | 标签宽度 |
| `label.height` | `number` | — | 标签高度 |
| `label.background` | `string` | — | 标签背景图片路径 |
| `label.padding` | `Array<number>` | — | 标签内边距，格式为 `[top, right, bottom, left]` |
| `label.offset` | `Array<number>` | — | 标签偏移量，格式为 `[x, y]` |

### 属性（getter/setter）

| 属性 | 类型 | 读写 | 说明 |
|------|------|------|------|
| `dataSource` | `DataSource` | get / set | 聚合点的原始数据源（继承自父类），设置后会触发聚合计算 |
| `clusterDataSource` | `GeoJSONDataSource` | readonly | 聚合后的子数据源，可通过 `defineAttribute` 自定义属性 |
| `clusterData` | `Array` | readonly | 当前聚合后的数据数组，包含聚合点和非聚合点（见下方数据结构） |
| `minUpdateInterval` | `number` | get / set | 聚合数据更新的最短时间间隔，单位 ms，默认 `300`，最小值 `16` |
| `isEventEntitySupported` | `boolean` | readonly | 始终为 `true`，表示支持事件参数中携带 entity 实体数据 |

### 方法（继承自 PointGroup）

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `addComponent(object)` | `GeoObject` | `GeoObject` | 添加子组件到聚合点，子组件会自动接收聚合后的数据源 |
| `removeComponent(object)` | `GeoObject` | — | 移除已添加的子组件 |
| `dispose()` | — | — | 释放所有子组件资源 |

### clusterData 数据结构

`clusterData` 返回一个 GeoJSON Feature 数组，每项为以下两种类型之一：

```javascript
// 非聚合数据（单个点）
{
    type: 'Feature',
    geometry: {
        coordinates: [119, 36],
    },
    properties: {
        count: 9,    // 原始数据属性
    }
}

// 聚合数据（多个点合并）
{
    type: 'Feature',
    geometry: {
        coordinates: [120, 36],
    },
    properties: {
        cluster: true,         // 标记为聚合数据
        cluster_id: 1,         // 聚合数据 ID
        point_count: 10,       // 聚合包含的数据点数量
    }
}
```

### 基本用法

```javascript
const cluster = engine.add(new mapvthree.ClusterPoint({
    cluster: {
        radius: 100,
        maxZoom: 18,
        minZoom: 5,
    },
    icon: {
        width: 30,
        height: 30,
        mapSrc: 'path/to/icon.png',
    },
    label: {
        fontSize: 14,
        width: 90,
        height: 40,
        fillStyle: '#ffffff',
        background: 'website_assets/images/speed-panel.png',
        padding: [18, 0, 0, 50],
        offset: [0, -30],
    },
}));

// 设置原始数据源
let data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
cluster.dataSource = data;
```

### 添加自定义子组件

ClusterPoint 通过 `addComponent` 方法支持添加任意 `GeoObject` 类型的子组件。子组件会自动绑定到聚合后的 `clusterDataSource`，随聚合结果同步更新。

```javascript
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';

// 添加 3D 模型作为聚合点的可视化组件
const loader = new GLTFLoader();
loader.load('models/diamond.glb', function (gltf) {
    const effectModelPoint = cluster.addComponent(new mapvthree.EffectModelPoint({
        normalize: true,
        rotateToZUp: true,
        size: 50,
        keepSize: false,
    }));
    effectModelPoint.model = gltf.scene;
});
```

### 自定义聚合数据源属性

通过 `clusterDataSource.defineAttribute` 可以为聚合后的每个点定义动态计算的属性：

```javascript
// 自定义文字内容
cluster.clusterDataSource.defineAttribute('text', (properties) => {
    return (properties && properties.cluster) ? properties.point_count.toString() : '1';
});

// 自定义背景图
cluster.clusterDataSource.defineAttribute('background', () => {
    return 'path/to/marker.png';
});
```

### 事件绑定

ClusterPoint 支持事件代理，点击聚合点时可获取对应的 entity 信息：

```javascript
cluster.addEventListener('click', (e) => {
    console.log('点击聚合点:', e);
    // e 中包含 index、value、itemIndex、pairs 等信息
});
```
