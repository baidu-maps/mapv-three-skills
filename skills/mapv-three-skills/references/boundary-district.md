# 行政区划服务

mapv-three 提供两个行政区划相关组件：`Boundary`（行政区边界查询服务）和 `DistrictLayer`（行政区划聚合图层）。两者均通过 `mapvthree.services` 命名空间导出，支持百度地图和天地图两种数据源。

## 数据源配置

在使用行政区划服务之前，需要了解数据源常量和全局配置函数：

| 常量 / 函数 | 说明 |
|---|---|
| `API_SOURCE_BAIDU` | 百度地图 LBS 服务（坐标系 BD09），值为 `'baidu'` |
| `API_SOURCE_TIANDITU` | 天地图 LBS 服务（坐标系 CGCS2000，与 WGS84 近似），值为 `'tdt'` |
| `setApiSource(source)` | 设置全局默认数据源 |
| `getApiSource()` | 获取当前全局数据源，默认为 `API_SOURCE_BAIDU` |

使用百度地图数据源前需设置 AK：
```javascript
mapvthree.BaiduMapConfig.ak = 'YOUR_BAIDU_AK';
```

使用天地图数据源前需设置 TK：
```javascript
mapvthree.TiandituConfig.tk = 'YOUR_TIANDITU_TK';
```

---

## Boundary - 行政区边界查询服务

`Boundary` 用于获取省、直辖市、县的轮廓线坐标数据，以坐标数组形式返回边界数据，不涉及地图渲染。

**导入方式：**
```javascript
import * as mapvthree from '@baidu/mapv-three';
const boundary = new mapvthree.services.Boundary(options);
```

### 构造函数

```javascript
new Boundary(options?)
```

#### 参数表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `options` | `Object` | 否 | `{}` | 配置对象 |
| `options.apiSource` | `string` | 否 | 全局配置值 | 数据源类型，可选 `API_SOURCE_BAIDU` 或 `API_SOURCE_TIANDITU` |

### 方法列表

#### `get(name, callback)`

获取指定行政区划的边界坐标。

| 参数 | 类型 | 说明 |
|---|---|---|
| `name` | `string` | 行政区划名称，如 `'山东省'`、`'北京市'`、`'台湾省'`、`'海南'` 等 |
| `callback` | `Function` | 回调函数，参数为结果对象 |

**回调参数结构：**

```typescript
{
    boundaries: Array<Array<[number, number]>>
    // boundaries[i] 是一个坐标环（闭合多边形），每个坐标为 [lng, lat]
}
```

**特殊处理说明：**
- 百度数据源内置了台湾省、钓鱼岛、赤尾屿的边界数据，无需网络请求
- 查询 `'中国'`、`'全国'`、`'中华人民共和国'` 时，百度数据源返回空数组
- 海南省边界会自动进行岛屿分割处理

#### `toString()`

返回 `'Boundary'`。

### 使用示例

```javascript
// 创建边界服务实例
const boundary = new mapvthree.services.Boundary({
    apiSource: mapvthree.services.API_SOURCE_BAIDU,
});

// 查询山东省边界
boundary.get('山东省', result => {
    console.log('边界数量:', result.boundaries.length);
    // result.boundaries[0] = [[lng1, lat1], [lng2, lat2], ...]

    // 将边界数据渲染为多边形
    const geojson = result.boundaries.map(ring => ({
        type: 'Feature',
        geometry: {
            type: 'Polygon',
            coordinates: [ring.map(([lng, lat]) => [lng, lat, 0])],
        },
    }));

    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
    dataSource.defineAttribute('color', () => '#ff6600');

    const polygon = engine.add(new mapvthree.Polygon({
        color: '#ff6600',
        opacity: 0.5,
        vertexColors: true,
    }));
    polygon.dataSource = dataSource;
});
```

---

## DistrictLayer - 行政区划聚合图层

`DistrictLayer` 是一个集数据获取与地图渲染于一体的高级组件，支持多层级行政区划展示（省、市、区县），可自动渲染填充多边形并跳转到对应视口。继承自 Three.js 的 `EventDispatcher`。

**导入方式：**
```javascript
import * as mapvthree from '@baidu/mapv-three';
const district = new mapvthree.services.DistrictLayer(options);
```

### 构造函数

```javascript
new DistrictLayer(options?)
```

#### 参数表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `options` | `Object` | 否 | `{}` | 配置对象 |
| `options.name` | `string` | 是 | - | 行政区划名称，格式如 `'(山东省)'`、`'(北京市)'`，需用括号包裹 |
| `options.kind` | `number` | 是 | - | 行政区划层级：`0` = 省级，`1` = 市级，`2` = 区县级 |
| `options.apiSource` | `string` | 否 | 全局配置值 | 数据源类型，可选 `API_SOURCE_BAIDU` 或 `API_SOURCE_TIANDITU` |
| `options.renderOptions` | `Object` | 否 | - | 渲染配置，提供后自动渲染到地图 |
| `options.renderOptions.engine` | `Engine` | 是* | - | 地图引擎实例（*若需要自动渲染则必填） |
| `options.renderOptions.autoViewport` | `boolean` | 否 | `false` | 是否自动跳转到区域中心 |
| `options.renderOptions.fillColor` | `string \| string[]` | 否 | `'#ff0'` | 填充颜色，支持单色字符串或颜色数组（随机取色） |
| `options.renderOptions.fillOpacity` | `number` | 否 | `0.6` | 填充透明度，0 ~ 1 |
| `options.renderOptions.*` | - | 否 | - | 其余属性将透传给底层 `Polygon` 组件作为选项 |

### 方法列表

#### `setName(name)`

设置行政区划名称并重新加载数据与渲染。

| 参数 | 类型 | 说明 |
|---|---|---|
| `name` | `string` | 行政区划名称，格式如 `'(广东省)'` |

```javascript
district.setName('(广东省)');
```

#### `setKind(kind)`

设置行政区划层级并重新加载数据与渲染。

| 参数 | 类型 | 说明 |
|---|---|---|
| `kind` | `number` | 层级值：`0` = 省级，`1` = 市级，`2` = 区县级 |

```javascript
district.setKind(1); // 切换为市级
```

#### `setOptions(options)`

同时更新名称和层级，并重新加载数据与渲染。

| 参数 | 类型 | 说明 |
|---|---|---|
| `options` | `Object` | 配置选项 |
| `options.name` | `string` | 行政区划名称（可选） |
| `options.kind` | `number` | 行政区划层级（可选） |

```javascript
district.setOptions({
    name: '(广东省)',
    kind: 1,
});
```

#### `searchBoundary(options, callback)`

搜索行政区划边界数据，不触发渲染，仅通过回调返回边界坐标数据。

| 参数 | 类型 | 说明 |
|---|---|---|
| `options` | `Object` | 搜索选项 |
| `options.name` | `string` | 行政区划名称（可选，默认使用实例当前值） |
| `options.kind` | `number` | 行政区划层级（可选，默认使用实例当前值） |
| `callback` | `Function` | 回调函数，参数为 `{boundaries: Array}` |

```javascript
district.searchBoundary({
    name: '(山东省)',
    kind: 1,
}, result => {
    console.log('边界数据:', result.boundaries);
});
```

#### `reset()`

移除当前渲染的行政区划图层，从引擎中清除多边形对象。

```javascript
district.reset();
```

### 事件列表

`DistrictLayer` 继承自 `EventDispatcher`，支持通过 `addEventListener` / `removeEventListener` 监听事件。

| 事件名 | 触发时机 | 事件对象属性 |
|---|---|---|
| `renderComplete` | 行政区划数据加载完毕且渲染完成后触发 | `results`: GeoJSON FeatureCollection 数据；`target`: DistrictLayer 实例 |

```javascript
district.addEventListener('renderComplete', event => {
    console.log('渲染完成，数据:', event.results);
    console.log('features 数量:', event.results.features.length);
});
```

### 使用示例

**基础用法 - 创建省级行政区划图层：**

```javascript
const engine = new mapvthree.Engine(container, {
    map: {projection: 'EPSG:3857'},
});
engine.add(new mapvthree.MapView({
    imageryProviders: [new mapvthree.BingImageryTileProvider()],
}));

const district = new mapvthree.services.DistrictLayer({
    name: '(山东省)',
    kind: 0,
    renderOptions: {
        engine: engine,
        fillColor: '#618bf8',
        fillOpacity: 0.6,
        autoViewport: true,
    },
});
```

**多色填充 - 使用颜色数组随机着色：**

```javascript
const district = new mapvthree.services.DistrictLayer({
    name: '(山东省)',
    kind: 1,  // 市级，显示各市区域
    renderOptions: {
        engine: engine,
        fillColor: ['red', 'blue', 'green', 'yellow'],
        fillOpacity: 0.6,
    },
});
```

**动态切换行政区划：**

```javascript
// 切换到广东省市级视图
district.setOptions({
    name: '(广东省)',
    kind: 1,
});

// 仅切换层级
district.setKind(2); // 切换到区县级

// 移除图层
district.reset();
```

---

## 常见场景

### 场景一：查询边界并自定义渲染

当需要对边界数据进行自定义处理（如拉伸 3D 效果）时，使用 `Boundary` 获取原始坐标：

```javascript
const boundary = new mapvthree.services.Boundary();
boundary.get('山东', result => {
    const geojson = result.boundaries.map(ring => ({
        type: 'Feature',
        geometry: {
            type: 'Polygon',
            coordinates: [ring.map(([lng, lat]) => [lng, lat, 0])],
        },
    }));

    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(geojson);
    dataSource.defineAttribute('color', () => Math.random() * 0xffffff);

    const polygon = engine.add(new mapvthree.Polygon({
        vertexColors: true,
        extrude: true,
        extrudeValue: 1000,
        opacity: 0.5,
    }));
    polygon.dataSource = dataSource;
    engine.map.flyTo([...result.boundaries[0][0], 10000], {range: 1000000});
});
```

### 场景二：行政区划下钻

实现省 -> 市 -> 区县的逐级下钻：

```javascript
const district = new mapvthree.services.DistrictLayer({
    name: '(中国)',
    kind: 0,
    renderOptions: {
        engine: engine,
        fillColor: ['#4e79a7', '#f28e2b', '#e15759', '#76b7b2'],
        fillOpacity: 0.6,
        autoViewport: true,
    },
});

// 下钻到省级
function drillDown(provinceName) {
    district.setOptions({
        name: `(${provinceName})`,
        kind: 1,
    });
}

// 继续下钻到区县级
function drillDownCity(cityName) {
    district.setOptions({
        name: `(${cityName})`,
        kind: 2,
    });
}
```

### 场景三：仅获取数据不渲染

不传 `renderOptions.engine`，仅利用 `searchBoundary` 获取数据：

```javascript
const district = new mapvthree.services.DistrictLayer({
    name: '(北京市)',
    kind: 1,
});

district.searchBoundary({name: '(北京市)', kind: 1}, result => {
    // result.boundaries 可用于自定义处理
    console.log(result.boundaries);
});
```

---

## 注意事项

1. **name 格式差异**：`Boundary.get()` 接受裸名称（如 `'山东省'`），而 `DistrictLayer` 的 `name` 需要用括号包裹（如 `'(山东省)'`）。
2. **异步加载**：`DistrictLayer` 构造后立即开始异步加载数据，渲染完成通过 `renderComplete` 事件通知，无需手动触发。
3. **kind 层级含义**：`kind=0` 表示获取该区域自身的整体边界；`kind=1` 获取其下级（如省的市级划分）；`kind=2` 获取区县级划分。当 `kind=1` 时，百度数据源会自动过滤首个 feature 的第一个坐标环（即省级外轮廓），仅保留子区域边界。
4. **坐标系**：百度数据源返回 BD09 坐标系数据；天地图数据源返回 CGCS2000 / WGS84 坐标系数据。请确保地图引擎的投影方式与数据源坐标系匹配。
5. **颜色数组**：`fillColor` 传入数组时，每个 feature 会从数组中随机选取一个颜色进行填充，适合多区域差异化着色。
6. **renderOptions 透传**：除 `engine`、`fillColor`、`fillOpacity`、`autoViewport` 外，`renderOptions` 中的其他属性会透传给底层 `Polygon` 组件，可借此控制 `vertexColors`、`transparent`、`opacity` 等渲染参数。
7. **图层清理**：调用 `reset()` 后图层被移除，可通过重新调用 `setName` / `setKind` / `setOptions` 重新加载并渲染。
8. **台湾省边界**：百度数据源内置台湾省（含附属岛屿）、钓鱼岛、赤尾屿的边界坐标，不依赖网络请求。
9. **autoViewport**：开启后，数据加载完成时会自动计算边界范围并调用 `engine.map.setViewport()` 跳转视口，需要引擎的 map 实例支持该方法。
