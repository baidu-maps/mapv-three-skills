# 矢量瓦片与地形

本文档涵盖 mapv-three 的矢量瓦片提供者（Mapbox MVT、百度矢量、GeoJSON 矢量、路况矢量等）、地形瓦片提供者（平面地形、Cesium 三维地形）以及材质管理器的使用方法。

---

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 设置 Mapbox Token
mapvthree.MapboxConfig.accessToken = 'your_mapbox_token';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { sky: null, enableAnimationLoop: false },
    map: { is3DControl: true },
});
engine.map.lookAt([116.404, 39.915], { range: 3000 });

// 创建 Mapbox 矢量瓦片底图
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
        accessToken: 'your_mapbox_token',
    }),
}));

// 添加天空盒
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 14 * 3600;
```

---

## 矢量瓦片提供者

### MapboxVectorTileProvider

Mapbox 矢量瓦片提供者，支持 Mapbox Style Specification v8，提供完整的矢量瓦片渲染。

```javascript
const provider = new mapvthree.MapboxVectorTileProvider({
    style: 'mapbox://styles/mapbox/streets-v11',
    accessToken: 'your_mapbox_token',
});

// 通过 MapView 使用
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,  // 不需要地形
    vectorProvider: provider,
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `style` | `string` | `'mapbox://styles/mapbox/streets-v11'` | Mapbox 样式 URL 或本地 JSON 文件路径 |
| `accessToken` | `string` | `MapboxConfig.accessToken` | Mapbox 访问令牌 |
| `defaultStyle` | `Object` | - | 默认样式配置 |
| `displayOptions` | `Object` | `{}` | 显示选项 |
| `displayOptions.background` | `boolean` | `true` | 是否显示背景 |
| `displayOptions.base` | `boolean` | `true` | 是否显示基础面 |
| `displayOptions.link` | `boolean` | `true` | 是否显示道路 |
| `displayOptions.building` | `boolean` | `true` | 是否显示 3D 建筑物 |
| `displayOptions.poi` | `boolean` | `true` | 是否显示 POI |
| `placeholderColor` | `Color` | 浅灰色 | 瓦片加载中的占位底色 |

#### 通过 MapView 设置相关属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `mapView.lodScaleFactor` | `number` | 细节层次缩放值 |
| `mapView.vectorSurface.materialManager` | `MaterialManager` | 材质管理器，可设置自定义材质 |

---

### BaiduVectorTileProvider

百度矢量瓦片提供者，支持在线和 DuGIS 离线模式。

#### 交互判断

**确定使用百度矢量后，必须在生成代码前完成以下判断，不能跳过：**

用户已明确说了"在线"或"离线"时，直接按对应模式处理。否则通过交互确认询问用户一次，提供两个选项（推荐项放在第一个）：
- `在线模式 — 使用百度地图在线矢量服务，需要百度 AK`（推荐）
- `离线模式 — 使用离线 DuGIS 服务器，需要提供服务器地址`

用户选择在线后，检查项目中是否已配置 `BaiduMapConfig.ak`，如果没有则提醒用户添加实际 AK。
用户选择离线后，让用户提供离线服务器地址（url）和静态资源地址（staticUrl）。

> **注意**：在线模式下必须先配置 `BaiduMapConfig.ak`。离线模式（`isOffline: true`）不需要 AK。BaiduLaneVectorTileProvider 不需要 AK，但**必须提供 `url` 参数**（瓦片服务地址），没有内置默认地址。如果用户没有提供 url，必须询问用户的瓦片服务地址。

```javascript
// 在线模式 - 必须先设置百度地图 AK
mapvthree.BaiduMapConfig.ak = 'your_baidu_ak';

const provider = new mapvthree.BaiduVectorTileProvider({
    displayOptions: {
        base: true,       // 基础面
        link: true,       // 道路
        building: true,   // 3D 建筑物
        poi: true,        // POI 标注
        flat: false,      // 是否压平模式
    },
});

// 通过 MapView 使用（terrainProvider: null 避免创建默认 Bing 栅格底图）
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: provider,
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ak` | `string` | `BaiduMapConfig.ak` | 百度地图 API 密钥 |
| `styleId` | `string` | - | 百度个性化地图样式 ID |
| `styleJson` | `string` | - | 百度个性化地图样式 JSON |
| `isOffline` | `boolean` | `false` | 是否为离线 DuGIS 模式 |
| `url` | `string` | - | 离线模式下的瓦片服务器地址 |
| `staticUrl` | `string` | - | 离线模式下的静态资源地址 |
| `projection` | `string` | `'EPSG:3857'` | 离线模式下的投影方式 |
| `displayOptions` | `Object` | - | 显示配置选项 |
| `displayOptions.base` | `boolean` | `true` | 是否显示基础面 |
| `displayOptions.link` | `boolean` | `true` | 是否显示道路 |
| `displayOptions.building` | `boolean` | `true` | 是否显示 3D 建筑 |
| `displayOptions.poi` | `boolean` | `true` | 是否显示 POI |
| `displayOptions.flat` | `boolean` | `true` | 是否压平显示 |
| `placeholderColor` | `Color` | - | 占位符颜色 |

---

### BaiduLaneVectorTileProvider

百度车道级矢量瓦片提供者，加载百度地图的车道级矢量瓦片数据（PBF 格式），渲染精细道路结构。支持室内建筑多楼层渲染、POI 标签、AOI 区域标注和自定义样式。无需 AK，但 **必须配置 `url`**（瓦片服务地址），否则无法加载任何数据。

```javascript
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.BaiduLaneVectorTileProvider({
        url: 'https://your-tile-server/xyz/{z}/{x}/{y}.pbf',
        styleUrl: 'https://your-server/assets/map/style/fs.json',
        displayOptions: {
            poi: true,
            building: true,
            flat: false,
        },
    }),
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | **必填** | 瓦片服务地址，支持 `{z}`、`{x}`、`{y}`、`{reverseY}` 占位符。没有内置默认地址，不配置则无法加载 |
| `styleUrl` | `string` | 内置路径 | 样式配置 JSON 文件 URL |
| `indoorStyleUrl` | `string` | 内置路径 | 室内样式配置 JSON 文件 URL |
| `resourceBaseUrl` | `string` | - | 静态资源（图标、样式等）基础 URL |
| `isOffline` | `boolean` | `false` | 是否为离线 DuGIS 模式 |
| `staticUrl` | `string` | - | 离线模式下的静态资源服务器地址 |
| `projection` | `string` | `'EPSG:3857'` | 离线模式下的投影方式 |
| `styleId` | `string` | - | 百度个性化地图样式 ID |
| `styleJson` | `string` | - | 百度个性化地图样式 JSON |
| `fill404Tiles` | `boolean` | `true` | 是否对 404 瓦片使用占位填充 |
| `placeholderColor` | `Color` | - | 占位符颜色 |
| `displayOptions.base` | `boolean` | `true` | 是否显示基础面 |
| `displayOptions.link` | `boolean` | `true` | 是否显示道路 |
| `displayOptions.building` | `boolean` | `true` | 是否显示 3D 建筑物 |
| `displayOptions.poi` | `boolean` | `true` | 是否显示 POI 标注 |
| `displayOptions.flat` | `boolean` | `true` | 是否压平显示 |
| `displayOptions.catalogs` | `number[]` | `[]` | 只显示指定 catalog 类型的要素，为空不过滤 |

#### 方法

| 方法 | 说明 |
|------|------|
| `setMapStyle(config)` | 设置个性化地图样式。`config.styleId` 或 `config.styleJson` |

#### 离线模式示例

```javascript
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.BaiduLaneVectorTileProvider({
        isOffline: true,
        url: 'http://dugis-server/xyz/{z}/{x}/{y}.pbf',
        styleUrl: 'http://dugis-server/assets/style/fs.json',
        resourceBaseUrl: 'http://dugis-server/assets/map/style/',
    }),
}));
```

---

### GeoJSONVectorTileProvider

GeoJSON 矢量瓦片提供者，将 GeoJSON 数据在本地切片并渲染。

```javascript
const provider = new mapvthree.GeoJSONVectorTileProvider({
    data: geojsonData,  // GeoJSON FeatureCollection
    projection: 'EPSG:3857',
    minZoom: 0,
    maxZoom: 16,
    tolerance: 3,
    styleOptions: {
        renderOrder: 0,
    },
});

const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: provider,
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `data` | `Object` | **必填** | GeoJSON 数据 |
| `projection` | `string` | `'EPSG:3857'` | 数据源投影 |
| `minZoom` | `number` | `0` | 最小层级 |
| `maxZoom` | `number` | `16` | 最大层级 |
| `tolerance` | `number` | `3` | 简化容差 |
| `styleOptions` | `Object` | `{}` | 样式配置选项 |

---

### 路况加载交互判断

用户要求加载路况时，根据用户提供的信息自动判断使用哪个提供者，不需要问用户使用哪个类：

- 用户说"加载百度路况" → 使用 `BaiduTrafficTileProvider`。如果用户已明确说了"在线"或"离线"，直接按对应模式处理。否则通过交互确认询问用户一次，提供两个选项（推荐项放在第一个）：
  - `在线模式 — 使用百度地图在线路况服务，需要百度 AK`（推荐）
  - `离线模式 — 使用离线路况服务器，需要提供服务器地址`
  
  用户选择在线后，检查项目中是否已配置 `BaiduMapConfig.ak`，未配置则提示提供。
  用户选择离线后，让用户提供离线路况服务器地址（url）。
- 用户**提供了路况瓦片 URL 和 API 地址** → 直接使用 `HDTrafficTileProvider`
- 用户提到"高精路况"、"HD路况" → 直接使用 `HDTrafficTileProvider`，然后让用户补充 url 和 apiHost
- 用户**只说"加载路况"**，无法判断类型时，通过交互确认询问用户一次，提供两个选项（推荐项放在第一个）：
  - `百度路况 — 使用百度地图内置路况数据，只需 AK`（推荐）
  - `高精路况 — 使用自定义路况服务，需提供瓦片 URL 和 API 地址`
  用户选择高精路况后，直接列出需要补充的参数让用户提供（不要给代码，只列参数）：

  ```
  加载高精路况需要以下参数：
  
  必填：
  1. url — 路况几何数据瓦片地址（如：https://xxx/tiles/{z}/{x}/{reverseY}.json）
  2. apiHost — 路况状态 API 服务地址（如：https://xxx）
  
  可选：
  3. ak — 鉴权密钥（如需要）
  4. cityCode — 城市代码（如需要）
  ```
  
  用户提供完必填参数后再生成完整代码

---

### BaiduTrafficTileProvider

百度路况瓦片提供者，使用百度地图内置路况数据，只需配置百度 AK 即可加载。支持自动刷新。

```javascript
// 在线路况
const trafficProvider = new mapvthree.BaiduTrafficTileProvider({
    autoRefresh: true,
    refreshInterval: 60000,  // 60秒刷新一次
});

// 通过 MapView 使用
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: trafficProvider,
}));

// 离线路况
const offlineMapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.BaiduTrafficTileProvider({
        isOffline: true,
        url: 'http://your-offline-server',
        projection: 'EPSG:3857',
    }),
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | - | 离线模式下的路况服务器地址 |
| `params` | `Object` | - | 请求参数 |
| `isOffline` | `boolean` | `false` | 是否为离线模式 |
| `projection` | `string` | `'EPSG:3857'` | 离线模式下的投影方式 |
| `autoRefresh` | `boolean` | `true` | 是否启用自动刷新 |
| `refreshInterval` | `number` | `60000` | 刷新间隔（毫秒），最小 60000 |
| `colors` | `Object` | - | 路况颜色配置 |
| `lineWidth` | `number` | - | 路况线条宽度 |
| `transparent` | `boolean` | `false` | 材质是否透明 |
| `height` | `number` | `0` | 路况线条高度偏移 |

#### 动态属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `autoRefresh` | `boolean` | 开启/关闭自动刷新 |
| `refreshInterval` | `number` | 设置刷新间隔（毫秒，最小 60000） |

---

### HDTrafficTileProvider

高精路况瓦片提供者，用于高精地图场景下的路况数据渲染。

```javascript
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.HDTrafficTileProvider({
        url: 'https://traffic-api.example.com/tiles/{z}/{x}/{reverseY}.json',
        apiHost: 'https://api.example.com',
        ak: 'your_ak',
        cityCode: '131',
        autoRefresh: true,
        refreshInterval: 60000,
        colorMap: [
            [128, 128, 128],  // 状态0: 灰色（未知）
            [0, 200, 0],      // 状态1: 绿色（畅通）
            [255, 200, 0],    // 状态2: 黄色（缓行）
            [255, 0, 0],      // 状态3: 红色（拥堵）
            [128, 0, 0],      // 状态4: 深红（严重拥堵）
        ],
        height: 5,  // 路况线抬升5米
    }),
}));
```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | **必填** | 路况几何数据 URL，支持 `{x}`、`{y}`、`{z}`、`{reverseY}` |
| `apiHost` | `string` | **必填** | 路况状态 API 服务地址 |
| `ak` | `string` | - | 鉴权 AK |
| `cityCode` | `string` | - | 城市代码 |
| `autoRefresh` | `boolean` | `true` | 是否启用自动刷新 |
| `refreshInterval` | `number` | `60000` | 刷新间隔（毫秒） |
| `colorMap` | `Array<Array<number>>` | - | 自定义路况颜色映射 |
| `height` | `number` | `0` | 路况线抬升高度（米） |

---

## 地形瓦片提供者

### PlaneTerrainTileProvider

平面地形提供者，生成纯平面几何网格（无高程数据），是 MapView 的默认地形提供者。

```javascript
// 通常不需要手动创建，MapView 默认使用
const terrainProvider = new mapvthree.PlaneTerrainTileProvider();
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: terrainProvider,
    imageryProvider: new mapvthree.BingImageryTileProvider(),
}));
```

支持的投影方式：`EPSG:3857`、`EPSG:4326`、`ECEF`

### CesiumTerrainTileProvider

Cesium 三维地形提供者，从 Cesium Ion 或自定义服务器加载高程地形数据。

```javascript
// 设置 Cesium Token
mapvthree.CesiumConfig.accessToken = 'your_cesium_ion_token';

// 使用 Cesium Ion 默认地形
const terrainProvider = new mapvthree.CesiumTerrainTileProvider({
    accessToken: 'your_cesium_ion_token',
});

// 使用自定义地形服务器
const customTerrain = new mapvthree.CesiumTerrainTileProvider({
    url: 'https://your-terrain-server/tiles',
});

// 与影像组合
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: terrainProvider,
    imageryProvider: new mapvthree.OSMImageryTileProvider(),
}));

```

#### 构造函数参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `accessToken` | `string` | `CesiumConfig.accessToken` | Cesium Ion 访问令牌 |
| `url` | `string` | Cesium Ion 默认地址 | 自定义地形服务器 URL |
| `requestVertexNormals` | `boolean` | `true` | 是否请求顶点法线数据 |

支持的投影方式：`ECEF`、`EPSG:3857`、`EPSG:4326`

---

## 材质管理器

材质管理器（MaterialManager）用于自定义矢量瓦片的渲染材质。矢量瓦片中的每种图层类型（背景、填充、线条、建筑等）都通过 key 映射到材质对象。

### DefaultQuadMapMaterialManager

所有材质管理器的基类。

#### 方法

| 方法 | 说明 |
|------|------|
| `getMaterialByKey(key)` | 根据 key 获取材质 |
| `addMaterialByKey(key, material)` | 添加或覆盖材质。`key` 可以是字符串或字符串数组 |
| `removeMaterialByKey(key)` | 删除指定 key 的材质 |

#### 常用 key

| Key | 说明 |
|-----|------|
| `'background'` | 背景面材质 |
| `'fill_wood'` | 绿地/植被填充材质 |
| `'fill_water'` | 水域填充材质 |
| `'fill_opaque'` | 不透明填充材质 |
| `'fill_translucent'` | 半透明填充材质 |
| `'fill_pattern'` | 纹理图案填充材质 |
| `'line_opaque'` | 不透明线条材质 |
| `'line_translucent'` | 半透明线条材质 |
| `'line_dashed'` | 虚线材质 |
| `'fill_extrusion_opaque'` | 建筑物不透明挤压材质 |
| `'fill_extrusion_translucent'` | 建筑物半透明挤压材质 |
| `'*'` | 通配符，匹配所有未定义的 key |

### DarkMVTMaterialManager

预配置的暗色主题材质管理器，适用于夜间/暗色风格场景。

```javascript
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
        accessToken: 'your_token',
    }),
}));

// 应用暗色主题
mapView.vectorSurface.materialManager = new mapvthree.DarkMVTMaterialManager();
```

**暗色主题配色方案：**
- 背景：`#17173e`（深蓝紫色）
- 绿地：`#3a752f`（深绿色）+ 草地纹理
- 水域：暗色反射水面材质
- 填充/道路：`#2b3f78`（深蓝色）
- 建筑侧面：`#405c8c`（蓝灰色）
- 建筑顶面：`#708cbc`（浅蓝灰色）

### CustomMVTMaterialManager

可自定义配色的材质管理器，内置 PBR 纹理加载功能。

```javascript
const customManager = new mapvthree.CustomMVTMaterialManager();
mapView.vectorSurface.materialManager = customManager;
```

**默认配色方案：**
- 背景：`#071A3A`
- 绿地：PBR 草地材质
- 水域：深色反射水面
- 填充/道路：PBR 道路材质
- 建筑：纹理贴图 + PBR 材质

### 自定义材质管理器

继承 `DefaultQuadMapMaterialManager` 创建自定义材质管理器：

```javascript
import {DefaultQuadMapMaterialManager} from '@baidu/mapv-three';
import {MeshStandardMaterial, Color} from 'three';

class MyMaterialManager extends DefaultQuadMapMaterialManager {
    onInit() {
        // 自定义背景颜色
        this.addMaterialByKey('background', new MeshStandardMaterial({
            color: new Color(0x1a1a2e),
        }));

        // 同一材质映射到多个 key
        this.addMaterialByKey(['fill_opaque', 'fill_translucent'], new MeshStandardMaterial({
            color: new Color(0x16213e),
        }));

        // 建筑物材质：[侧面材质, 顶面材质]
        this.addMaterialByKey('fill_extrusion_opaque', [
            new MeshStandardMaterial({ color: new Color(0x0f3460) }),
            new MeshStandardMaterial({ color: new Color(0x533483) }),
        ]);
    }
}

// 使用自定义材质管理器
mapView.vectorSurface.materialManager = new MyMaterialManager();
```

---

## 常见场景

### 场景1：Mapbox 矢量底图 + 暗色主题

```javascript
import * as mapvthree from '@baidu/mapv-three';

mapvthree.MapboxConfig.accessToken = 'your_mapbox_token';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    map: { is3DControl: true },
    rendering: { sky: null, enableAnimationLoop: false },
});
engine.map.lookAt([119.588, 25.881], { heading: 0, pitch: 0, range: 3000 });

// 创建天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 14 * 3600;

// 创建 MVT 底图
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
        accessToken: 'your_mapbox_token',
    }),
}));
mapView.lodScaleFactor = 1.9;

// 切换到暗色主题
mapView.vectorSurface.materialManager = new mapvthree.DarkMVTMaterialManager();

// 动态切换回默认主题
// mapView.vectorSurface.materialManager = null;
```

### 场景2：百度矢量底图

```javascript
import * as mapvthree from '@baidu/mapv-three';

mapvthree.BaiduMapConfig.ak = 'your_baidu_ak';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { sky: null, enableAnimationLoop: false },
});
engine.map.lookAt([116.492, 39.906], { heading: 0, pitch: 0, range: 3000 });

// 百度矢量底图
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.BaiduVectorTileProvider({
        // styleUrl: 'path/to/custom_style.json',  // 可选：自定义样式
        // displayOptions: {
        //     base: true,
        //     link: true,
        //     building: true,
        //     poi: true,
        // },
    }),
}));

// 添加天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 3600 * 15.5;
```

### 场景3：Cesium 三维地形 + OSM 影像

```javascript
import * as mapvthree from '@baidu/mapv-three';

mapvthree.CesiumConfig.accessToken = 'your_cesium_token';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    map: { is3DControl: true },
    rendering: { sky: null, enableAnimationLoop: false },
});
engine.map.setCenter([113.301, 23.558]);
engine.map.setZoom(12);
engine.map.setPitch(80);

// Cesium 三维地形 + OSM 底图
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: new mapvthree.CesiumTerrainTileProvider(),
    imageryProvider: new mapvthree.OSMImageryTileProvider(),
}));

// 添加天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 17 * 3600;
```

### 场景4：百度路况叠加

```javascript
import * as mapvthree from '@baidu/mapv-three';

mapvthree.BaiduMapConfig.ak = 'your_baidu_ak';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
});
engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(14);

// 底图
const mapView = engine.add(new mapvthree.MapView());

// 叠加路况矢量图层
const trafficMapView = engine.add(new mapvthree.MapView({
    terrainProvider: null,
    vectorProvider: new mapvthree.BaiduTrafficTileProvider({
        autoRefresh: true,
        refreshInterval: 60000,
    }),
}));

// 手动刷新路况数据
// trafficMapView.refresh();
```

### 场景5：通过 MapView 组合地形 + 影像 + 矢量

```javascript
import * as mapvthree from '@baidu/mapv-three';

mapvthree.CesiumConfig.accessToken = 'your_cesium_token';
mapvthree.MapboxConfig.accessToken = 'your_mapbox_token';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { enableAnimationLoop: false },
    map: { is3DControl: true },
});
engine.map.lookAt([116.404, 39.915], { range: 50000, pitch: 60 });

// 方式一：通过构造函数一次性配置
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: new mapvthree.CesiumTerrainTileProvider(),
    imageryProviders: [
        new mapvthree.BingImageryTileProvider(),
    ],
    vectorProvider: new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
    }),
}));

// 方式二：逐步添加 Surface
const mapView2 = engine.add(new mapvthree.MapView({
    terrainProvider: null,  // 不创建默认栅格表面
}));
mapView2.addRasterSurface(
    new mapvthree.CesiumTerrainTileProvider(),
    [new mapvthree.BingImageryTileProvider()]
);
mapView2.addVectorSurface(
    new mapvthree.MapboxVectorTileProvider({
        style: 'mapbox://styles/mapbox/streets-v11',
    })
);
```

---

## RasterSurface 和 VectorSurface

### RasterSurface

栅格表面，组合地形提供者和影像提供者进行渲染。

```javascript
const rasterSurface = new mapvthree.RasterSurface(terrainProvider, imageryProviders);
mapView.addSurface(rasterSurface);
```

| 方法 | 说明 |
|------|------|
| `addImageryLayer(provider)` | 添加一个影像图层 |
| `removeImageryLayer(provider)` | 移除一个影像图层 |

### VectorSurface

矢量表面，负责矢量瓦片数据的加载和渲染。

```javascript
const vectorSurface = new mapvthree.VectorSurface(vectorTileProvider);
mapView.addSurface(vectorSurface);
```

---

## 注意事项

1. **Token 配置**：使用 Mapbox/百度/Cesium/天地图服务前，必须先设置对应的全局 Token
2. **矢量底图无地形**：仅使用矢量提供者时，建议将 `terrainProvider` 设为 `null`（不创建栅格表面）
3. **材质管理器生命周期**：`materialManager` 的 `engine` 属性在 Surface 初始化时自动赋值；`onInit()` 在首次使用时被调用
4. **建筑物材质**：`fill_extrusion_*` 类型的 key 接受数组 `[侧面材质, 顶面材质]`，而非单个材质
5. **路况自动刷新**：`refreshInterval` 最小值为 60000ms（60秒），设置更小的值会自动校正
6. **DuGIS 离线模式**：百度矢量和百度路况都支持离线模式，通过 `isOffline: true` 和 `url` 参数配置
7. **GeoJSON 矢量瓦片**：数据在本地使用 Web Worker 切片处理，适合中小规模数据集
8. **Cesium 地形**：默认从 Cesium Ion 加载全球地形数据，需要有效的 `accessToken`；支持 `requestVertexNormals` 获取更好的光照效果
9. **投影兼容**：矢量瓦片提供者通常支持所有投影（`_supportAllProjections = true`），系统会自动处理投影转换
