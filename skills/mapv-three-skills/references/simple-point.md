# SimplePoint 简单点

SimplePoint 用于在地图上绘制基础的点标记，支持自定义颜色、大小、形状以及数据驱动的样式配置。适用于大批量散点可视化场景。

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略，参见 Engine 文档）

// 1. 创建 SimplePoint 实例并添加到引擎
const point = engine.add(new mapvthree.SimplePoint({
    color: 'rgba(250, 90, 50, 1)',
    size: 50,
    opacity: 0.8,
    vertexColors: true,
    vertexSizes: true,
}));

// 2. 创建数据源并绑定
const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: {
        type: 'Point',
        coordinates: [116.404, 39.915],
    },
    properties: {
        color: '#ff0000',
        size: 30,
    },
}]);

// 3. 定义数据驱动的属性映射
data.defineAttribute('color', 'color');
data.defineAttribute('size', 'size');

// 4. 将数据源赋给 point
point.dataSource = data;
```

## 构造函数

```javascript
new mapvthree.SimplePoint(parameters)
```

### 参数表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `string \| THREE.Color` | `0xffff00`（黄色） | 点的全局颜色，支持 CSS 颜色字符串、十六进制等格式 |
| `size` | `number` | `30` | 点的大小（像素） |
| `opacity` | `number` | `1` | 不透明度，取值范围 0~1。需配合 `transparent: true` 才能实现半透明效果 |
| `transparent` | `boolean` | `false` | 是否开启透明渲染 |
| `vertexColors` | `boolean` | `false` | 是否启用数据驱动的颜色。开启后每个点可通过数据源中的 `color` 字段设置独立颜色 |
| `vertexSizes` | `boolean` | `false` | 是否启用数据驱动的大小。开启后每个点可通过数据源中的 `size` 字段设置独立大小 |
| `uShapeType` | `number` | `2` | 点的形状类型。`1` = 方形，`2` = 圆形 |
| `mapSrc` | `string` | `null` | 自定义纹理贴图的 URL。设置后点将使用该贴图渲染，替代默认的几何形状 |
| `offset` | `Array<number>` | `[0, 0]` | 点的屏幕偏移量 `[x, y]`，单位为像素 |
| `emissive` | `Array<number>` | `[0, 0, 0]` | 自发光颜色 `[r, g, b]`，取值范围 0~1 |

## 属性列表

以下属性均支持 getter/setter，可在实例创建后动态修改。

| 属性 | 类型 | 说明 |
|------|------|------|
| `color` | `string` | 点颜色，支持 CSS 颜色字符串，如 `'#ff0000'`、`'rgba(255,0,0,0.5)'` |
| `size` | `number` | 点大小（像素） |
| `opacity` | `number` | 不透明度 |
| `vertexColors` | `boolean` | 是否启用数据驱动颜色 |
| `vertexSizes` | `boolean` | 是否启用数据驱动大小 |
| `uShapeType` | `number` | 点形状类型 |
| `mapSrc` | `string` | 纹理贴图 URL |
| `emissive` | `Array<number>` | 自发光颜色 |
| `transparent` | `boolean` | 是否开启透明（修改后自动触发材质更新） |
| `dataSource` | `DataSource` | 数据源对象 |
| `visible` | `boolean` | 是否可见（继承自 THREE.Object3D） |
| `needsUpdate` | `boolean` | 设为 `true` 可强制在下一帧更新渲染数据 |
| `parameters` | `Object` | 只读，返回当前构造参数 |

## 方法列表

| 方法 | 说明 |
|------|------|
| `dispose()` | 销毁 geometry 和 material，释放 GPU 资源。通常通过 `engine.remove(point)` 自动调用，无需手动执行 |
| `addAttributeRename(key, value)` | 添加属性重命名映射。`key` 为组件期望的属性名，`value` 为数据源中的实际属性名 |
| `removeAttributeRename(key)` | 移除指定的属性重命名映射 |
| `clearAttributeRename()` | 清空所有属性重命名映射 |

## 常见场景

### 基础散点图

```javascript
const point = engine.add(new mapvthree.SimplePoint({
    color: '#00ff00',
    size: 40,
    uShapeType: 1,  // 方形点
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
point.dataSource = data;
```

### 数据驱动的多色散点

通过 `vertexColors` 和 `vertexSizes` 让每个点拥有独立的颜色和大小。

```javascript
const point = engine.add(new mapvthree.SimplePoint({
    vertexColors: true,
    vertexSizes: true,
    size: 10,
    uShapeType: 2,  // 圆形点
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);
data.defineAttribute('color', p => Math.random() * 0xffffffff);
data.defineAttribute('size', p => Math.random() * 20);
point.dataSource = data;
```

### 动态添加数据点

支持在运行时通过 `DataItem` 逐条添加数据。

```javascript
const point = engine.add(new mapvthree.SimplePoint({
    size: 40,
    uShapeType: 1,
}));
point.color = '#00ff0066';

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([]);
point.dataSource = data;

// 点击地图时添加新的点
engine.map.addEventListener('click', e => {
    data.add(new mapvthree.DataItem(e.point));
});
```

### 使用自定义贴图

```javascript
const point = engine.add(new mapvthree.SimplePoint({
    size: 60,
    mapSrc: 'https://example.com/marker.png',
}));
```

### 半透明效果

```javascript
const point = engine.add(new mapvthree.SimplePoint({
    color: 'blue',
    size: 30,
    opacity: 0.5,
    transparent: true,
}));
```

## 注意事项

1. **透明度生效条件**：设置 `opacity` 小于 1 时，必须同时将 `transparent` 设为 `true`，否则半透明效果不会生效。

2. **数据驱动模式**：使用 `vertexColors` 或 `vertexSizes` 时，需要在数据源上通过 `defineAttribute` 定义对应的 `color` 或 `size` 属性映射，否则数据驱动不会生效。

3. **颜色格式**：`color` 属性支持多种格式，包括十六进制字符串（`'#ff0000'`）、带透明度的十六进制（`'#00ff0066'`）、RGBA 字符串（`'rgba(250, 90, 50, 1)'`）等。在数据驱动模式下，`color` 值也可以使用 `0xffffffff` 格式的整数。

4. **形状类型**：`uShapeType` 目前支持 `1`（方形）和 `2`（圆形），默认为圆形。

5. **性能建议**：SimplePoint 基于 WebGL Points 原语实现，适合大规模散点渲染（万级以上数据量）。数据量较小且需要复杂交互时，也可考虑其他点类型组件。

6. **资源释放**：使用 `engine.remove(point)` 移除点对象时会自动调用 `dispose()` 释放资源，无需手动销毁。如果设置了 `mapSrc` 贴图，贴图纹理也会被一并释放。

7. **球面模式**：在地球（Globe）模式下，SimplePoint 会自动进行水平面剔除（Horizon Culling），背面不可见的点不会被渲染，无需额外处理。
