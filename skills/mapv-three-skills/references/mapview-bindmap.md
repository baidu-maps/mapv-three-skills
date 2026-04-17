# MapView 底图系统

MapView 是 mapv-three 中用于承载地图底图数据的核心容器视图，支持多种瓦片数据加载，包括地形、影像、矢量等。本文档涵盖 MapView 的创建与配置，以及各种影像瓦片提供者的使用方法。

---

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 1. 创建引擎
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        sky: null,
        enableAnimationLoop: false,
    },
});

// 2. 设置地图中心和层级
engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(10);

// 3. 创建 MapView（默认使用 Bing 卫星影像 + 平面地形）
const mapView = engine.add(new mapvthree.MapView());

// 4. （可选）添加天空盒
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 15 * 3600;
```

---

## 架构概览

```
MapView
├── RasterSurface          // 栅格表面（地形 + 影像）
│   ├── TerrainProvider    // 地形提供者（生成几何网格）
│   └── ImageryProvider[]  // 影像提供者数组（叠加纹理贴图）
└── VectorSurface          // 矢量表面
    └── VectorProvider     // 矢量提供者（矢量瓦片数据）
```

- **RasterSurface**：负责地形网格 + 影像纹理的组合渲染，由一个 `TerrainProvider` 和多个 `ImageryProvider` 构成
- **VectorSurface**：负责矢量瓦片数据的解析和渲染
- 一个 MapView 可以同时包含多个 Surface

---

## MapView 构造函数

```javascript
const mapView = new mapvthree.MapView(options);
engine.add(mapView);
```

### 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.rasterSurface` | `RasterSurface` | - | 自定义栅格表面实例。提供后 `terrainProvider` 和 `imageryProviders` 无效 |
| `options.terrainProvider` | `TerrainProvider` | `PlaneTerrainTileProvider` | 地形提供者。设为 `null` 可跳过地形 |
| `options.imageryProvider` | `ImageryProvider` | - | 单个影像提供者（简写方式） |
| `options.imageryProviders` | `ImageryProvider[]` | `[BingImageryTileProvider]` | 影像提供者数组，支持多层叠加 |
| `options.vectorSurface` | `VectorSurface` | - | 自定义矢量表面实例 |
| `options.vectorProvider` | `VectorProvider` | - | 矢量瓦片提供者 |
| `options.vectorSurfaceOptions` | `Object` | - | 矢量表面的额外配置选项 |

### 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `addSurface(surface)` | `void` | 添加一层 Surface |
| `addRasterSurface(terrainProvider, imageryProviders, options)` | `RasterSurface` | 添加一层栅格表面。`terrainProvider` 不能为 `null`，需传入有效实例（如 `new PlaneTerrainTileProvider()`） |
| `addVectorSurface(vectorProvider, options)` | `VectorSurface` | 添加一层矢量表面 |
| `removeSurface(surface)` | `void` | 移除一层 Surface |
| `setTerrainProvider(terrainProvider)` | `void` | 替换默认栅格表面的地形提供者 |
| `setImageryProviders(imageryProviders)` | `void` | 替换默认栅格表面的影像提供者数组 |
| `setImageryProvider(imageryProvider)` | `void` | 替换默认栅格表面为单个影像提供者 |
| `setVectorProvider(vectorProvider, options)` | `void` | 设置或替换矢量提供者 |
| `getImageryProviders()` | `ImageryProvider[]` | 获取当前影像提供者列表 |
| `refresh()` | `void` | 刷新瓦片数据，用于更新动态图层（如路况） |
| `dispose()` | `void` | 销毁 MapView，释放所有资源 |

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `freezeUpdate` | `boolean` | 是否冻结瓦片更新 |
| `lodScaleFactor` | `number` | 细节层次缩放值，默认 2。值越大加载越精细 |
| `rasterSurface` | `RasterSurface` | 获取默认栅格表面 |
| `vectorSurface` | `VectorSurface` | 获取默认矢量表面 |
| `surfaces` | `Surface[]` | 获取所有 Surface 列表 |

---

## 常用底图创建方式

以下是通过 MapView + Provider 创建常用底图的标准写法：

```javascript
// Bing 卫星影像
const bingMap = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.BingImageryTileProvider(),
}));

// OSM 底图
const osmMap = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.OSMImageryTileProvider(),
}));

// XYZ 自定义瓦片
const xyzMap = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.XYZImageryTileProvider({
        url: 'https://example.com/tiles/{z}/{x}/{y}.png',
    }),
}));

// OGC WMS 底图
const wmsMap = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.WMSImageryTileProvider({
        url: 'https://example.com/geoserver/wms',
        params: {
            LAYERS: 'my_layer',
        },
    }),
}));

// OGC WMTS 底图
const wmtsMap = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.WMTSImageryTileProvider({
        url: 'https://example.com/geoserver/gwc/service/wmts',
        params: {
            LAYER: 'my_layer',
            TILEMATRIXSET: 'EPSG:900913',
        },
    }),
}));

// Mapbox 矢量瓦片底图
const mvtMap = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
        accessToken: 'your_mapbox_token',
    }),
}));

// 百度矢量瓦片底图
const baiduVectorMap = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.BaiduVectorTileProvider(),
}));

// 百度路况矢量底图
const trafficMap = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.BaiduTrafficTileProvider(),
}));

// 高精路况矢量底图
const hdTrafficMap = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.HDTrafficTileProvider({
        url: 'https://traffic-api.example.com/tiles/{z}/{x}/{reverseY}.json',
        apiHost: 'https://api.example.com',
    }),
}));

// Cesium 三维地形 + OSM 影像
const terrain = engine.add(new mapvthree.MapView({
    terrainProvider: new mapvthree.CesiumTerrainTileProvider({
        accessToken: 'your_cesium_token', // Cesium Ion 地形需要 token
    }),
    imageryProvider: new mapvthree.OSMImageryTileProvider(),
}));
```

---

## 影像瓦片提供者

所有影像提供者继承自 `ImageryTileProvider`（基类为 `BaseImageryTileProvider`），具有以下通用参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `opacity` | `number` | `1` | 不透明度，0-1 |
| `colorTint` | `number[]` | `[1, 1, 1]` | 色彩调整值，RGB 分量 |
| `placeholderColor` | `Color` | - | 瓦片加载中的占位底色 |
| `minLevel` | `number` | `0` | 最小展示层级 |
| `maxLevel` | `number` | 因提供者而异 | 最大展示层级 |

### BingImageryTileProvider

Bing 卫星影像，支持三种样式。

```javascript
const provider = new mapvthree.BingImageryTileProvider({
    style: mapvthree.mapViewConstants.BING_MAP_STYLE_AERIAL,
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `style` | `string` | `'Aerial'` | 地图样式 |

**样式常量：**

| 常量 | 值 | 说明 |
|------|-----|------|
| `BING_MAP_STYLE_AERIAL` | `'Aerial'` | 纯卫星影像 |
| `BING_MAP_STYLE_AERIAL_WITH_LABELS` | `'AerialWithLabels'` | 卫星影像 + 标注 |
| `BING_MAP_STYLE_ROAD` | `'Road'` | 道路地图 |

**动态切换样式：**

```javascript
provider.style = mapvthree.mapViewConstants.BING_MAP_STYLE_ROAD;
engine.requestRender();
```

### OSMImageryTileProvider

OpenStreetMap 影像瓦片，无需额外配置。

```javascript
const provider = new mapvthree.OSMImageryTileProvider();
const mapView = engine.add(new mapvthree.MapView({
    imageryProvider: provider,
}));
```

### XYZImageryTileProvider

标准 XYZ 瓦片服务，支持自定义 URL 模板。

```javascript
const provider = new mapvthree.XYZImageryTileProvider({
    url: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
    projection: 'EPSG:3857',  // 或 'EPSG:4326'
    maxLevel: 18,
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | **必填** | URL 模板，支持 `{x}`、`{y}`、`{reverseY}`、`{z}` 占位符 |
| `projection` | `string` | `'EPSG:3857'` | 投影方式，支持 `'EPSG:3857'` 和 `'EPSG:4326'` |
| `maxLevel` | `number` | `18` | 最大展示层级（当未指定或为 Infinity 时回退到 18） |
| `minLevel` | `number` | `0` | 最小展示层级 |
| `startLevel` | `number` | `0` | 瓦片网格起始层级。**必须与瓦片数据的实际起始层级一致**，否则低于此层级的瓦片请求会全部 404 失败，导致整个图层无法正常加载 |

> **注意：** `{y}` 对应谷歌瓦片规范（左上原点），`{reverseY}` 对应 TMS 规范（左下原点）。

#### 自定义瓦片服务常见问题排查

> **重要**：当用户反馈自定义瓦片加载不出来时，**不要直接修改代码**。应将以下排查清单列出，引导用户逐条确认，待用户确认具体原因后再进行修改。

当自定义 XYZ 瓦片地址加载不出影像时，请用户按以下顺序逐条排查：

1. **验证瓦片地址可访问性**：在浏览器中手动拼接一个具体的瓦片 URL（如 `http://your-server/tiles/4/8/5.png`），确认能正常返回图片。如果 404 或无法访问，说明是服务端问题而非 mapv-three 问题

2. **检查 startLevel 是否匹配数据**：确认瓦片数据的实际最小层级是多少。如果数据不是从第 0 级开始（比如从第 4 级开始），**必须设置 `startLevel`** 与之一致。
   - **如果用户已知最小层级**：直接设置 `startLevel` 即可
   - **如果用户不确定最小层级**：让用户提供一个可以正常访问的瓦片地址（或从浏览器 Network 面板中找一个），AI 自动探测：
     1. 从已知可用瓦片的 z/x/y 出发，利用四叉树父节点关系逐级向上推算：**z 减 1，x 和 y 各除以 2 取整**（此规则与投影无关，是所有瓦片金字塔的通用性质）
     2. AI 使用 WebFetch 逐级访问推算出的父瓦片 URL，检查返回状态
     3. 最后一个能正常返回图片的层级就是 startLevel
   - **示例**（用户提供 `http://10.224.22.115:8098/mianyang/dom/7/103/54.png` 可访问）：
     ```
     AI 自动逐级探测：
     z=7: /7/103/54.png   → 200 ✓
     z=6: /6/51/27.png    → 200 ✓
     z=5: /5/25/13.png    → 200 ✓
     z=4: /4/12/6.png     → 200 ✓
     z=3: /3/6/3.png      → 404 ✗
     → 自动确定 startLevel = 4
     ```
   - **注意**：**不能简单地访问 `/{z}/0/0.png`**，因为区域性数据（如只覆盖某个城市）在 `0/0` 位置对应的是全球左上角区域，不在数据覆盖范围内，所有层级都会 404
   - **原因**：mapv-three 的瓦片四叉树从 startLevel 开始构建，如果 startLevel=0 但数据从第 4 级才有，那么第 0~3 级的瓦片请求全部失败（触发失败计数），连续 3 次失败后该瓦片在 15 分钟内不再重试，导致后续层级也无法正常加载
   - **修复示例**（待用户确认最小层级后使用）：
   ```javascript
   new mapvthree.XYZImageryTileProvider({
       url: 'http://your-server/tiles/{z}/{x}/{y}.png',
       startLevel: 4,  // 改为用户确认的实际最小层级
   });
   ```

3. **检查 Y 坐标方向**：瓦片服务有两种 Y 轴方向标准：
   - 谷歌/OSM 规范（左上原点，Y 从上往下增大）→ URL 中使用 `{y}`
   - TMS 规范（左下原点，Y 从下往上增大）→ URL 中使用 `{reverseY}`

   **确认方法**：如果用 `{y}` 加载后影像上下颠倒或位置偏移，换成 `{reverseY}` 试试，反之亦然

4. **检查跨域（CORS）问题**：打开浏览器开发者工具的 Console/Network 面板，查看是否有 CORS 报错。自建瓦片服务器需要在响应头中添加 `Access-Control-Allow-Origin`

### Baidu09ImageryTileProvider

百度地图栅格影像瓦片，支持卫星影像和普通影像。

```javascript
// 需要先设置 AK
mapvthree.BaiduMapConfig.ak = 'your_baidu_ak';

// 普通影像
const provider = new mapvthree.Baidu09ImageryTileProvider({
    type: 'normal',
});

// 卫星影像
const satProvider = new mapvthree.Baidu09ImageryTileProvider({
    type: 'satellite',
});

// 路网图
const streetProvider = new mapvthree.Baidu09ImageryTileProvider({
    type: 'street',
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ak` | `string` | `BaiduMapConfig.ak` | 百度地图 API 密钥 |
| `type` | `string` | `'normal'` | 影像类型：`'normal'`（普通）/ `'satellite'`（卫星）/ `'street'`（路网） |

### TiandituImageryTileProvider

天地图影像瓦片，需要天地图 Token。

```javascript
// 设置天地图 Token
mapvthree.TiandituConfig.tk = 'your_tianditu_token';

const provider = new mapvthree.TiandituImageryTileProvider({
    // tk: 'your_tianditu_token',  // 也可单独传入
});

// 天地图建议使用 EPSG:4326 投影
const engine = new mapvthree.Engine(container, {
    map: {
        projection: 'EPSG:4326',
    },
});
const mapView = engine.add(new mapvthree.MapView({
    imageryProvider: provider,
}));
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tk` | `string` | `TiandituConfig.tk` | 天地图 Token |

### StadiaImageryTileProvider

Stadia 地图影像瓦片，提供多种风格化样式。

```javascript
const provider = new mapvthree.StadiaImageryTileProvider({
    style: mapvthree.mapViewConstants.STADIA_MAP_STYLE_STAMEN_WATERCOLOR,
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `style` | `string` | `'StamenWatercolor'` | 地图样式 |

**样式常量：**

| 常量 | 值 | 说明 |
|------|-----|------|
| `STADIA_MAP_STYLE_STAMEN_WATERCOLOR` | `'StamenWatercolor'` | 水彩画风格 |
| `STADIA_MAP_STYLE_STAMEN_TONER` | `'StamenToner'` | 素描风格 |
| `STADIA_MAP_STYLE_ALIDE_SMOOTH` | `'AlidadeSmooth'` | 平滑浅色风格 |
| `STADIA_MAP_STYLE_ALIDE_SMOOTH_DARK` | `'AlidadeSmoothDark'` | 平滑暗色风格 |
| `STADIA_MAP_STYLE_OUTDOORS` | `'Outdoors'` | 户外风格 |

### WMSImageryTileProvider

OGC WMS 服务影像瓦片，支持 1.1.1 和 1.3.0 版本。

```javascript
const provider = new mapvthree.WMSImageryTileProvider({
    url: 'https://example.com/geoserver/wms',
    params: {
        LAYERS: 'my_layer',
        SRS: 'EPSG:3857',
        VERSION: '1.1.1',
    },
    hidpi: true,
    serverType: 'geoserver',  // 用于 HiDPI 处理
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | **必填** | WMS 服务地址 |
| `params` | `Object` | `{}` | WMS 请求参数 |
| `params.LAYERS` | `string` | `''` | 图层名（逗号分隔多个） |
| `params.VERSION` | `string` | `'1.3.0'` | WMS 版本 |
| `params.SRS/CRS` | `string` | `'EPSG:3857'` | 投影坐标系 |
| `params.FORMAT` | `string` | `'image/png'` | 图片格式 |
| `params.TRANSPARENT` | `boolean` | `true` | 是否透明 |
| `params.STYLES` | `string` | `''` | 样式 |
| `hidpi` | `boolean` | `false` | 是否启用 HiDPI（需要同时指定 `serverType`） |
| `serverType` | `string` | - | 服务器类型：`'geoserver'`/`'mapserver'`/`'carmentaserver'`/`'qgis'`。仅在 `hidpi: true` 时生效，不同类型使用不同 DPI 参数 |

**VERSION 1.1.1 与 1.3.0 的区别：**

| 区别项 | VERSION < 1.3.0（如 1.1.1） | VERSION >= 1.3.0 |
|--------|-----|-----|
| 投影参数名 | `SRS` | `CRS` |
| BBOX 坐标顺序（地理坐标系） | `minX,minY,maxX,maxY`（lon,lat） | `minY,minX,maxY,maxX`（lat,lon） |

> **注意：** 源码会根据 VERSION 自动选择 `SRS` 或 `CRS` 参数名，用户也可以在 params 中显式指定。BBOX 坐标顺序的翻转仅在 VERSION >= 1.3.0 且使用地理坐标系（如 EPSG:4326）时发生。

**HiDPI 不同 serverType 的处理方式：**

| serverType | HiDPI 参数 |
|------------|-----------|
| `geoserver` | 设置 `FORMAT_OPTIONS` 中的 `dpi` 值 |
| `mapserver` | 设置 `MAP_RESOLUTION` 参数 |
| `carmentaserver` / `qgis` | 设置 `DPI` 参数 |

**方法：**

| 方法 | 说明 |
|------|------|
| `setParams(params)` | 更新 WMS 参数并刷新瓦片（会合并到现有参数中） |
| `getParams()` | 获取当前 WMS 参数的副本 |

### WMTSImageryTileProvider

OGC WMTS 服务影像瓦片，支持 KVP 和 REST 两种请求方式。

```javascript
// KVP 方式
const kvpProvider = new mapvthree.WMTSImageryTileProvider({
    url: 'https://example.com/geoserver/gwc/service/wmts',
    params: {
        LAYER: 'my_layer',
        TILEMATRIXSET: 'EPSG:900913',
        TILEMATRIX: 'EPSG:900913:{z}',
        VERSION: '1.0.0',
    },
});

// REST 方式
const restProvider = new mapvthree.WMTSImageryTileProvider({
    url: 'https://example.com/wmts/{layer}/{style}/{TileMatrixSet}/{TileMatrix}/{TileRow}/{TileCol}.png',
    requestEncoding: 'REST',
    params: {
        layer: 'USGSTopo',
        style: 'default',
        TileMatrixSet: 'default028mm',
    },
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | **必填** | WMTS 服务地址（KVP 模式为服务端点；REST 模式为含占位符的模板 URL） |
| `params` | `Object` | `{}` | WMTS 请求参数 |
| `params.LAYER` | `string` | `''` | 图层名 |
| `params.STYLE` | `string` | `''` | 样式名 |
| `params.TILEMATRIXSET` | `string` | `'EPSG:3857'` | 瓦片矩阵集（同时用于确定投影方式） |
| `params.TILEMATRIX` | `string` | `'{z}'` | 瓦片矩阵标识模板，`{z}` 会被替换为层级号或 matrixIds 中的对应值 |
| `params.TILEROW` | `string` | `'{y}'` | 行号模板（自动替换，无需修改） |
| `params.TILECOL` | `string` | `'{x}'` | 列号模板（自动替换，无需修改） |
| `params.VERSION` | `string` | `'1.0.0'` | WMTS 版本（仅 KVP 模式下发送） |
| `params.FORMAT` | `string` | `'image/png'` | 图片格式（仅 KVP 模式下发送） |
| `requestEncoding` | `string` | `'KVP'` | 请求方式：`'KVP'` 或 `'REST'` |
| `matrixIds` | `Array` | - | 自定义瓦片矩阵 ID 映射数组，索引为层级号。如 `['EPSG:3857:0', 'EPSG:3857:1', ...]` |

> **KVP 与 REST 模式区别：** KVP 模式会自动附加 `SERVICE=WMTS`、`REQUEST=GetTile`、`VERSION`、`FORMAT` 等标准参数到 URL 查询字符串中。REST 模式则通过 URL 模板中的 `{layer}`、`{style}`、`{tilematrixset}`、`{tilematrix}`、`{tilerow}`、`{tilecol}` 占位符替换来构建 URL（大小写不敏感）。

> **matrixIds 说明：** 当提供 `matrixIds` 时，URL 中 `{z}` 和 TILEMATRIX 参数的值会被替换为 `matrixIds[z]` 而非层级号本身。适用于 TILEMATRIX 标识不是简单数字的 WMTS 服务。

**方法：**

| 方法 | 说明 |
|------|------|
| `setParams(params)` | 更新 WMTS 参数并刷新瓦片（会合并到现有参数中） |
| `getParams()` | 获取当前 WMTS 参数的副本 |

---

## 常见场景

### 场景1：Bing 卫星底图 + 可切换样式

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
});
engine.map.lookAt([116.404, 39.915], { range: 1000000 });

// 创建 Bing 影像提供者
const bingProvider = new mapvthree.BingImageryTileProvider({
    style: mapvthree.mapViewConstants.BING_MAP_STYLE_AERIAL,
});

// 创建 MapView
const mapView = engine.add(new mapvthree.MapView({
    imageryProvider: bingProvider,
}));

// 动态切换样式
function switchStyle(style) {
    bingProvider.style = style;
    engine.requestRender();
}
```

### 场景2：多层影像叠加（WMS 叠加在 Bing 之上）

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
    map: { projection: 'EPSG:4326' },
});
engine.map.lookAt([116.404, 39.915], { range: 1000000 });

// Bing 底图 + WMS 叠加图层
const wmsProvider = new mapvthree.WMSImageryTileProvider({
    url: 'https://example.com/geoserver/wms',
    params: {
        LAYERS: 'overlay_layer',
        VERSION: '1.1.1',
    },
    hidpi: true,
    serverType: 'geoserver',
});

const mapView = engine.add(new mapvthree.MapView({
    imageryProviders: [
        new mapvthree.BingImageryTileProvider(),
        wmsProvider,
    ],
}));
```

### 场景3：天地图底图（EPSG:4326 投影）

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 设置天地图 Token
mapvthree.TiandituConfig.tk = 'your_tianditu_token';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
    map: {
        projection: 'EPSG:4326',  // 天地图建议使用 4326 投影
    },
});
engine.map.lookAt([116.404, 39.915], { range: 1000000 });

const mapView = engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.TiandituImageryTileProvider(),
}));
```

### 场景4：运行时追加影像图层

通过 `rasterSurface.addImageryLayer()` 在运行时动态叠加影像层，新图层渲染在已有图层之上。

```javascript
// 创建默认 MapView（默认有 Bing 底图）
const mapView = engine.add(new mapvthree.MapView());

// 运行时叠加 OSM 影像层（opacity < 1 才能看到下层底图）
const osmProvider = new mapvthree.OSMImageryTileProvider({ opacity: 0.5 });
mapView.rasterSurface.addImageryLayer(osmProvider);

// 移除影像层
mapView.rasterSurface.removeImageryLayer(osmProvider);
```

> **注意**：叠加的影像层如果 `opacity` 为 1（默认值），会完全遮挡下层底图。需要设置 `opacity < 1` 才能实现混合叠加效果。

### 场景5：WMTS REST 请求方式

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
});
engine.map.lookAt([-98.5795, 39.8283], { range: 5000000 });

const wmtsProvider = new mapvthree.WMTSImageryTileProvider({
    url: 'https://basemap.nationalmap.gov/arcgis/rest/services/USGSTopo/MapServer/WMTS/tile/1.0.0/{layer}/{style}/{TileMatrixSet}/{TileMatrix}/{TileRow}/{TileCol}.png',
    requestEncoding: 'REST',
    params: {
        layer: 'USGSTopo',
        style: 'default',
        TileMatrixSet: 'default028mm',
    },
});

const mapView = engine.add(new mapvthree.MapView({
    imageryProviders: [
        new mapvthree.BingImageryTileProvider(),
        wmtsProvider,
    ],
}));
```

---

## 全局配置

使用百度地图、天地图等服务前，需要设置对应的密钥：

```javascript
// 百度地图 AK
mapvthree.BaiduMapConfig.ak = 'your_baidu_ak';

// 天地图 Token
mapvthree.TiandituConfig.tk = 'your_tianditu_token';

// Cesium Ion AccessToken
mapvthree.CesiumConfig.accessToken = 'your_cesium_token';

// Mapbox AccessToken
mapvthree.MapboxConfig.accessToken = 'your_mapbox_token';
```

---

## 注意事项

1. **默认底图**：不传任何参数创建 `MapView` 时，默认使用 `BingImageryTileProvider`（Aerial 样式）+ `PlaneTerrainTileProvider`（平面地形）。如果只想创建纯矢量底图（不要默认的 Bing 栅格），必须显式传 `terrainProvider: null`，否则会同时出现 Bing 卫星图
2. **投影系统**：天地图推荐使用 `EPSG:4326`，其余大多数提供者默认使用 `EPSG:3857`（Web Mercator）。在 Engine 配置中通过 `map.projection` 设置
3. **多层叠加**：多个栅格图层叠加时用 `imageryProviders` 数组放在同一个 MapView 中（推荐，能够解决多个独立 MapView 栅格图层叠加的 z 冲突问题）。数组中越后面的图层渲染在越上层，可通过 `opacity` 控制各层透明度
4. **性能优化**：
   - `lodScaleFactor`：默认为 2，降低该值可减少瓦片加载量
5. **动态刷新**：路况等动态数据需调用 `mapView.refresh()` 触发更新
6. **样式切换**：Bing/Stadia 影像提供者的 `style` 属性修改后会自动清除缓存（`_needsClearCache = true`），需调用 `engine.requestRender()` 触发重绘
7. **XYZ 坐标约定**：mapv-three 内部使用 TMS（左下原点）。在 `XYZImageryTileProvider` 的 URL 模板中：`{y}` 对应谷歌瓦片规范（左上原点），`{reverseY}` 对应 TMS 规范（左下原点）。大多数公开 XYZ 服务使用谷歌规范，应使用 `{y}`
8. **参数动态更新**：WMS/WMTS 的 `setParams()` 方法会合并参数并标记缓存清除（`_needsClearCache = true`），无需手动调用 refresh
