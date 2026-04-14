# 数据源

MapV-Three 提供了灵活的数据源（DataSource）系统，负责管理原始数据到渲染数据的流转，支持 GeoJSON、CSV、JSON 等格式。几乎所有可视化组件都通过 `layer.dataSource = dataSource` 绑定数据。

## 快速开始

```javascript
// 假设 engine 已初始化

// 从 GeoJSON 创建数据源
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON({
    type: 'FeatureCollection',
    features: [{
        type: 'Feature',
        geometry: { type: 'Point', coordinates: [116.404, 39.915] },
        properties: { name: '北京', value: 100 },
    }],
});

// 定义属性映射
dataSource.defineAttribute('color', p => (p.value > 50 ? 0xff0000 : 0x00ff00));

// 关联可视化对象
const pointLayer = engine.add(new mapvthree.SimplePoint({
    size: 20,
    vertexColors: true,
}));
pointLayer.dataSource = dataSource;
```

---

## DataItem - 数据元素

数据源中的基本单元，表示单个地理要素。

### 构造函数

```javascript
new mapvthree.DataItem(feature, extraAttributes, forceProjected)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `feature` | `Object \| Array \| Vector3` | **必填** | 几何信息，支持 GeoJSON Feature、坐标数组或 Vector3 |
| `extraAttributes` | `Object` | `{}` | 附加属性，`id` 用于标识数据元素，`crs` 用于指定源投影 |
| `forceProjected` | `boolean` | `false` | 是否强制视为已投影坐标 |

### 创建方式

```javascript
// 方式1：从 GeoJSON Feature 创建
const item1 = new mapvthree.DataItem({
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.39, 39.9] },
    properties: { name: '北京', value: 100 },
});

// 方式2：从坐标数组创建
const item2 = new mapvthree.DataItem([116.39, 39.9], { id: 'point1', name: '北京' });

// 方式3：从 Vector3 创建
const item3 = new mapvthree.DataItem(new THREE.Vector3(116.39, 39.9, 0));

// 方式4：指定投影坐标（如 EPSG:3857 墨卡托坐标）
const item4 = new mapvthree.DataItem(
    [12958056.27, 4853597.99, 0],
    { id: 1, crs: 'EPSG:3857' }
);
```

### 属性与方法

| 属性/方法 | 类型 | 说明 |
| --- | --- | --- |
| `id` | `string \| number` | 数据项 ID，只能设置一次 |
| `coordinates` | `Array` | 经纬度坐标 |
| `attributes` | `Object` | 属性数据对象 |
| `type` | `number` | 几何类型（Point/Line/Polygon） |
| `isMulti` | `boolean` | 是否为多要素类型（MultiPoint 等） |
| `size` | `number` | 多要素时的要素数量 |
| `isValid` | `boolean` | 数据项是否有效 |
| `setAttribute(key, value)` | - | 设置单个属性 |
| `setAttributes(obj)` | - | 设置多个属性 |
| `setCoordinates(coords, projection)` | - | 设置坐标 |
| `toGeoJSON()` | `Object` | 导出为 GeoJSON Feature |

---

## DataSource - 数据源基类

抽象数据源类，管理原始数据到渲染数据的流转。

### 构造函数

```javascript
new mapvthree.DataSource(options)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `options.id` | `string` | 时间戳 | 数据源 ID |
| `options.attributes` | `Object` | `undefined` | 属性映射对象 |
| `options.coordType` | `string` | `undefined` | 源坐标类型 |

### 核心方法

| 方法 | 返回值 | 说明 |
| --- | --- | --- |
| `add(dataItem)` | `DataSource` | 添加数据元素（单个或数组） |
| `remove(dataItem)` | `DataSource` | 移除数据元素（单个、数组或 ID） |
| `clear()` | - | 清空所有数据 |
| `setData(data)` | - | 替换整个数据源的数据 |
| `load(url)` | `Promise<DataSource>` | 从 URL 加载数据 |
| `get(index)` | `Object` | 获取第 index 个解析后的数据 |
| `getDataItem(index)` | `DataItem` | 获取第 index 个原始 DataItem |
| `exportToGeoJSON()` | `Object` | 导出为 GeoJSON FeatureCollection |
| `dispose()` | - | 释放资源 |

### 属性映射方法

| 方法 | 说明 |
| --- | --- |
| `defineAttribute(attr, property)` | 定义单个属性映射，`property` 可以是字段名或回调函数 |
| `defineAttributes(obj)` | 批量定义属性映射 |
| `undefineAttribute(name)` | 移除属性映射 |
| `undefineAllAttributes()` | 清除所有属性映射 |
| `setAttributeValue(ids, key, value)` | 按 ID 设置单个属性值 |
| `setAttributeValues(ids, values)` | 按 ID 设置多个属性值 |
| `setCoordinates(ids, coordinates)` | 按 ID 设置坐标 |
| `setFilter(filter)` | 设置过滤器函数 |

### 事件

| 方法 | 说明 |
| --- | --- |
| `addEventListener(event, listener)` | 监听事件，支持 `'afterAdd'` 和 `'afterRemove'` |
| `removeEventListener(event, listener)` | 移除事件监听 |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `size` | `number` | 数据项数量（只读） |
| `dataItems` | `DataItem[]` | 所有数据项（只读） |

---

## GeoJSONDataSource - GeoJSON 数据源

支持标准 GeoJSON 格式（FeatureCollection、Feature、Features 数组），继承自 DataSource。

### 静态方法

| 方法 | 返回值 | 说明 |
| --- | --- | --- |
| `GeoJSONDataSource.fromGeoJSON(geoJSON, options)` | `GeoJSONDataSource` | 从 GeoJSON 对象创建 |
| `GeoJSONDataSource.fromURL(url, options)` | `Promise<GeoJSONDataSource>` | 从 URL 异步加载 |

### 完整示例

```javascript
// 从 GeoJSON 对象创建
const ds1 = mapvthree.GeoJSONDataSource.fromGeoJSON({
    type: 'FeatureCollection',
    features: [
        {
            type: 'Feature',
            geometry: { type: 'Point', coordinates: [116.404, 39.915] },
            properties: { name: '北京', value: 100 },
        },
        {
            type: 'Feature',
            geometry: { type: 'Point', coordinates: [121.47, 31.23] },
            properties: { name: '上海', value: 90 },
        },
    ],
});

// 定义颜色属性映射（使用回调函数）
ds1.defineAttribute('color', (attributes, dataItem, renderingIndex, dataIndex) => {
    return attributes.value > 80 ? 0xff0000 : 0x00ff00;
});

// 定义大小属性映射
ds1.defineAttribute('size', 'value');

// 关联可视化对象
const points = engine.add(new mapvthree.SimplePoint({
    vertexColors: true,
    vertexSizes: true,
    size: 20,
}));
points.dataSource = ds1;

// 从 URL 加载
const ds2 = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/point.geojson');
ds2.defineAttribute('color', p => p.color || 0xff0000);

// 动态添加和删除
ds1.add(new mapvthree.DataItem([116.414, 39.925], { id: 'new1', value: 80 }));
ds1.remove('new1');

// 按 ID 修改属性
ds1.setAttributeValues('point1', { color: 0xffffff, size: 30 });

// 导出为 GeoJSON
const geojson = ds1.exportToGeoJSON();
console.log(geojson.features.length);
```

### 共享数据源与属性重命名

```javascript
// 多个图层共享同一个数据源，通过属性重命名区分
const sharedDS = mapvthree.GeoJSONDataSource.fromGeoJSON(features);
sharedDS.defineAttribute('color1', (p, item, j, i) => {
    return new THREE.Color(i / total, 0, 0);
});
sharedDS.defineAttribute('color2', (p, item, j, i) => {
    return new THREE.Color(0, i / total, 0);
});

const layer1 = engine.add(new mapvthree.SimplePoint({ vertexColors: true }));
// 将数据源的 color1 映射到图层的 color 属性
layer1.addAttributeRename('color', 'color1');
layer1.dataSource = sharedDS;

const layer2 = engine.add(new mapvthree.EffectPoint({ }));
layer2.addAttributeRename('color', 'color2');
layer2.dataSource = sharedDS;
```

### 数据过滤

```javascript
// 使用过滤器，仅显示偶数索引的数据项
const filteredDS = mapvthree.GeoJSONDataSource.fromGeoJSON(features);
filteredDS.setFilter((item, index) => {
    return index % 2 === 0;
});
```

---

## CSVDataSource - CSV 数据源

支持 CSV 格式数据，几何字段使用 WKT 编码。继承自 JSONDataSource。

### 静态方法

| 方法 | 返回值 | 说明 |
| --- | --- | --- |
| `CSVDataSource.fromURL(url, options)` | `Promise<CSVDataSource>` | 从 URL 异步加载 |
| `CSVDataSource.fromCSVString(csvString, options)` | `CSVDataSource` | 从 CSV 字符串创建 |

### 完整示例

```javascript
// 从 URL 加载 CSV 数据（CSV 中 coordinates 列为 WKT 格式）
const csvDS = await mapvthree.CSVDataSource.fromURL('data/csv/point.csv');
csvDS.defineAttribute('color', attrs => {
    const value = attrs.value || 50;
    const ratio = value / 100;
    return (Math.round(255 * ratio) << 16) | (Math.round(128) << 8) | Math.round(255 * (1 - ratio));
});
csvDS.defineAttribute('size', attrs => (attrs.value || 50) * 0.5 + 10);

const pointLayer = engine.add(new mapvthree.SimplePoint({
    vertexColors: true,
    vertexSizes: true,
    size: 30,
}));
pointLayer.dataSource = csvDS;

// 从 CSV 字符串创建
const csvString = `name,value,coordinates
北京,100,"POINT(116.404 39.915)"
上海,90,"POINT(121.47 31.23)"`;

const csvDS2 = mapvthree.CSVDataSource.fromCSVString(csvString);

// 自定义几何字段名
const customDS = new mapvthree.CSVDataSource({
    coordinatesKey: 'geometry', // 使用 geometry 字段而非默认的 coordinates
});
customDS.setData(csvStringWithGeometryField);
```

---

## JSONDataSource - JSON 数据源

支持 JSON 格式数据，可自定义坐标解析。继承自 DataSource。

### 构造函数选项

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `options.coordinatesKey` | `string` | `'coordinates'` | 坐标字段名 |
| `options.parseCoordinates` | `Function` | `undefined` | 自定义坐标解析函数 |
| `options.parseFeature` | `Function` | `undefined` | 自定义要素解析函数 |

### 静态方法

| 方法 | 返回值 | 说明 |
| --- | --- | --- |
| `JSONDataSource.fromURL(url, options)` | `Promise<JSONDataSource>` | 从 URL 异步加载 |
| `JSONDataSource.fromJSON(jsonData, options)` | `JSONDataSource` | 从 JSON 对象创建 |

### 完整示例

```javascript
// 从 URL 加载（使用自定义 coordinatesKey）
const jsonDS = await mapvthree.JSONDataSource.fromURL('data/json/point.json', {
    coordinatesKey: 'position', // JSON 中 position 字段包含 WKT 坐标
});

// 从 JSON 数组创建（使用自定义坐标解析）
const jsonDS2 = mapvthree.JSONDataSource.fromJSON([
    { color: 0xff0000, lon: 116.404, lat: 39.915 },
    { color: 0x00ff00, lon: 121.47, lat: 31.23 },
], {
    // 自定义坐标解析函数
    parseCoordinates: (item) => {
        return [item.lon, item.lat];
    },
    // 构造时批量定义属性映射
    attributes: {
        color: p => p.color,
    },
});

const layer = engine.add(new mapvthree.SimplePoint({
    vertexColors: true,
    size: 20,
}));
layer.dataSource = jsonDS2;

// 动态添加数据
setTimeout(() => {
    jsonDS.add(new mapvthree.DataItem([0.001, 0.001], { count: 2 }));
}, 3000);
```

---

## 注意事项

1. **数据源与图层关联**：通过 `layer.dataSource = dataSource` 将数据源与可视化对象关联。一个数据源可以被多个图层共享，通过 `addAttributeRename` 实现属性名映射。

2. **defineAttribute 回调函数签名**：`defineAttribute('color', (attributes, dataItem, renderingIndex, dataIndex) => {...})`。`attributes` 为属性对象，`dataItem` 为 DataItem 实例，`renderingIndex` 为多要素内的子索引，`dataIndex` 为数据源内的全局索引。

3. **动态更新**：调用 `add`、`remove`、`setAttributeValues`、`setCoordinates` 后，数据源会在下一次渲染帧自动更新，无需手动调用 `update`。

4. **CSV 坐标格式**：CSVDataSource 默认使用 `coordinates` 列的 WKT 格式（如 `POINT(116.404 39.915)`）。可通过 `coordinatesKey` 自定义字段名，或通过 `parseCoordinates` 自定义解析逻辑。

5. **ID 管理**：DataItem 的 `id` 只能设置一次。如果不手动设置，数据源会在添加时自动生成 `_gid_N` 格式的 ID。交互操作（如点击修改、删除）依赖 ID 来定位数据项。
