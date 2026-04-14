# mapv-three 项目搭建指南

## 交互判断

用户请求搭建 mapv-three 项目时，需要确认网络环境以决定安装方式。

如果用户已明确说明（如"我是内网"、"我在外网"），直接按其环境给出方案，不再询问。否则通过交互确认询问用户一次，提供两个选项（推荐项放在第一个）：
- `百度内网环境 — 可直接 npm install（推荐）`
- `互联网环境 — 需手动拷贝 @baidu/mapv-three 包`

两种环境的区别：
- **百度内网**：能访问 `registry.npm.baidu-int.com`，直接 `npm install -S @baidu/mapv-three`
- **互联网**：无法访问百度内网 registry，需从内网获取 `@baidu/mapv-three` 的 tgz 包或目录，拷贝到项目中通过 `"file:./libs/@baidu/mapv-three"` 本地依赖安装

两种环境安装完成后，包名和 API 完全一致，都是 `@baidu/mapv-three`。

## 快速开始

以下是一个最小可工作示例：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>mapv-three 快速开始</title>
    <style>
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
        }
        #map_container {
            width: 100%;
            height: 100%;
            position: relative;
        }
    </style>
</head>
<body>
    <div id="map_container"></div>
    <script type="module">
        import * as mapvthree from '@baidu/mapv-three';

        // 创建引擎实例（关闭默认天空，使用自定义天空）
        const engine = new mapvthree.Engine('map_container', {
            rendering: { sky: null },
        });

        // 设置视野中心点（经度, 纬度）
        engine.map.setCenter([116.404, 39.915]);
        engine.map.setZoom(14);

        // 添加动态天空
        const sky = engine.add(new mapvthree.DynamicSky());
    </script>
</body>
</html>
```

## 安装方式

### 百度内网环境

内网可直接通过 npm 安装（registry 已在 `.npmrc` 中配置）：

```bash
# 1. 配置内网 npm 源（项目根目录创建 .npmrc）
echo "registry=http://registry.npm.baidu-int.com" > .npmrc

# 2. 安装 mapv-three 和 three.js
npm install -S @baidu/mapv-three
# 如果 three.js 未自动安装（peerDependency >= 0.179.1）
npm install -S three@0.179.1
```

### 互联网环境

互联网无法访问百度内网 npm registry，需手动获取包：

```bash
# 1. 从百度内网获取 @baidu/mapv-three 的 tgz 包或目录
# 放到项目中，例如 libs/@baidu/mapv-three/

# 2. 在 package.json 中配置本地依赖
```

```json
{
    "dependencies": {
        "@baidu/mapv-three": "file:./libs/@baidu/mapv-three"
    }
}
```

```bash
# 3. 安装依赖
npm install

# 4. 安装 three.js（版本 >= 0.179.1）
npm install -S three
```

### 安装后引入

两种环境安装完成后，引入方式一致：

```javascript
import * as mapvthree from '@baidu/mapv-three';
import * as THREE from 'three';
```

| 导入路径 | 说明 |
|---------|------|
| `@baidu/mapv-three` | 核心库 |
| `@baidu/mapv-three/addons` | 扩展组件（资产管理、飞线等） |
| `@baidu/mapv-three/adapters` | 适配器（百度地图 JSAPI 兼容层） |

### 静态资源配置

mapv-three 初始化时需要加载静态资源（着色器、纹理等）。未正确配置会报错：
> "Unable to determine base URL automatically, try defining a global variable called MAPV_BASE_URL."

**第 1 步：在 `index.html` 中声明资源路径**

```html
<script>
    window.MAPV_BASE_URL = 'mapvthree/';
</script>
```

**第 2 步：配置构建工具复制资源**

Vite/Rollup：

```javascript
// vite.config.js
import copy from 'rollup-plugin-copy';

export default {
    plugins: [
        copy({
            targets: [
                { src: 'node_modules/@baidu/mapv-three/dist/assets', dest: 'public/mapvthree' }
            ],
            verbose: true,
            hook: 'buildStart',
        }),
    ]
};
```

Webpack：

```javascript
// webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    plugins: [
        new CopyWebpackPlugin({
            patterns: [
                { from: 'node_modules/@baidu/mapv-three/dist/assets', to: 'mapvthree/assets' }
            ],
        }),
    ]
};
```

## 项目结构

一个典型的 mapv-three 项目结构如下：

```
my-project/
├── public/
│   └── mapvthree/        # mapv-three 静态资源（构建工具自动拷贝）
├── src/
│   ├── main.js           # 入口文件
│   └── map.js            # 地图初始化逻辑
├── index.html
├── package.json
└── vite.config.js        # 或 webpack.config.js
```

## 核心模块概览

mapv-three 导出的模块按功能划分为以下几个大类：

| 类别 | 主要导出 | 说明 |
|------|---------|------|
| 核心引擎 | `Engine` | 渲染引擎入口类 |
| 天空环境 | `DynamicSky`, `StaticSky`, `EmptySky`, `DefaultSky`, `VerticalGradientSky` 等 | 天空和环境光照 |
| 地图底图 | `MapView`, `BaiduVectorTileProvider`, `BingImageryTileProvider`, `OSMImageryTileProvider` 等 | 地图瓦片底图 |
| 3D 瓦片 | `Default3DTiles`, `Cesium3DTileset` 等 | 3DTiles 数据加载 |
| 可视化对象 | `SimplePoint`, `Polyline`, `Polygon`, `Heatmap`, `Wall` 等 | 点线面可视化元素 |
| 模型 | `SimpleModel`, `AnimationModel`, `LODModel` | 3D 模型加载 |
| 数据源 | `DataSource`, `GeoJSONDataSource`, `CSVDataSource`, `JSONDataSource` | 数据管理 |
| 坐标系 | `PROJECTION_GEO`, `PROJECTION_WEB_MERCATOR`, `PROJECTION_ECEF` | 坐标系常量 |

### 坐标系常量

```javascript
import {
    PROJECTION_GEO,           // 'EPSG:4326' - WGS84经纬度
    PROJECTION_WEB_MERCATOR,  // 'EPSG:3857' - Web墨卡托
    PROJECTION_ECEF,          // 'EPSG:4978' - 地心地固坐标系
    PROJECTION_BD_MERCATOR,   // 'BD:MERCATOR' - 百度墨卡托
    PROJECTION_SCREEN_PIXEL,  // 'SCREEN_PIXEL' - 屏幕像素坐标系
} from '@baidu/mapv-three';
```

## 常见场景

### 场景1：使用 Vite 搭建项目

```bash
# 创建项目
npm create vite@latest my-map-app -- --template vanilla
cd my-map-app

# 安装依赖（百度内网）
npm install
npm install -S @baidu/mapv-three three@0.179.1
# 互联网环境需先将包拷贝到 libs/ 目录，配置 file: 依赖后再 npm install
```

```javascript
// src/main.js
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine('map_container', {
    rendering: {
        sky: null,                 // 关闭默认天空
        enableAnimationLoop: true, // 开启动画循环
    },
});

engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(14);

// 添加动态天空（需先关闭默认天空）
const sky = engine.add(new mapvthree.DynamicSky());

// 添加矢量底图
const mapView = engine.add(new mapvthree.MapView({
    vectorProvider: new mapvthree.BaiduVectorTileProvider({}),
}));
```


## 注意事项

1. **Node.js 版本要求**：需要 Node.js >= 20。
2. **three.js 版本**：需要 `three >= 0.179.1`（peerDependency），安装时会自动提示。
3. **包名统一**：无论内网还是外网，包名都是 `@baidu/mapv-three`。区别仅在于安装方式（内网 npm install vs 外网手动拷贝包）。
4. **静态资源**：必须配置 `MAPV_BASE_URL` 全局变量并拷贝 assets 资源到输出目录，否则会报 "Unable to determine base URL" 错误。
5. **默认天空**：Engine 创建时自带默认天空。如果要使用 `DynamicSky` 等自定义天空，需在配置中设置 `rendering: { sky: null }` 关闭默认天空，否则会出现两个天空叠加。
6. **循环渲染**：默认关闭循环渲染以节省性能。若有动画效果（飞线、粒子等），需设置 `rendering.enableAnimationLoop = true`。
7. **three.js 基础**：mapv-three 底层基于 three.js，建议先了解 `Camera`、`Renderer`、`Object3D`、`Geometry`、`Material` 等基础概念。
