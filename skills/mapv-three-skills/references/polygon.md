# Polygon 多边形

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 引擎初始化（略）

// 创建多边形
const polygon = engine.add(
    new mapvthree.Polygon({
        color: 'red',
        opacity: 0.8,
    })
);

// 加载 GeoJSON 数据
const dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
polygon.dataSource = dataSource;
```

---

## 构造函数

```javascript
new mapvthree.Polygon(parameters?)
```

### 参数列表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `extrude` | `boolean` | `false` | 是否沿 Z 轴拉伸为柱状体 |
| `extrudeValue` | `number` | `0` | 拉伸高度（单位：米） |
| `vertexHeights` | `boolean` | `false` | 是否通过数据携带高度配置 |
| `enableBottomFace` | `boolean` | `false` | 拉伸时是否封闭底面 |
| `color` | `string` | `'#ffffff'` | 填充颜色 |
| `vertexColors` | `boolean` | `false` | 是否通过数据携带颜色配置 |
| `opacity` | `number` | `1` | 整体透明度系数 |
| `mapSrc` | `string` | `''` | 纹理贴图路径 |
| `mapScale` | `number` | `1` | 贴图缩放大小，仅在使用纹理贴图时有效 |
| `zOffset` | `number` | `0` | 整体高度抬升（单位：米），在几何体构建时叠加到所有顶点的高度上 |
| `perPositionHeight` | `boolean` | `false` | 使用顶点高度；若为 `false`，即使数据中设置了高度也按 0 处理 |
| `renderOrder` | `number` | `0` | 渲染顺序，值越大越晚渲染 |
| `border` | `boolean` | `false` | 是否显示边框 |
| `borderColor` | `string` | `'#000000'` | 边框颜色 |
| `borderWidth` | `number` | `2` | 边框线宽（像素） |
| `borderOpacity` | `number` | `1` | 边框透明度 |

---

## 属性（getter / setter）

以下属性均支持在实例创建后动态修改。

### 几何属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `extrude` | `boolean` | 是否拉伸多边形 |
| `extrudeValue` | `number` | 拉伸高度值 |
| `vertexHeights` | `boolean` | 是否通过数据携带高度 |
| `enableBottomFace` | `boolean` | 是否封闭底面 |
| `zOffset` | `number` | 整体高度抬升 |
| `perPositionHeight` | `boolean` | 是否使用顶点高度 |

### 材质属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `color` | `string` | 填充颜色 |
| `vertexColors` | `boolean` | 是否通过数据携带颜色 |
| `opacity` | `number` | 整体透明度系数 |
| `transparent` | `boolean` | 是否开启透明 |
| `mapScale` | `number` | 贴图缩放大小 |
| `mapSrc` | `string` | 纹理贴图路径（设置后自动启用 UV） |
| `side` | `number` | 渲染面（THREE.FrontSide / THREE.DoubleSide 等） |
| `depthWrite` | `boolean` | 是否写入深度缓冲 |

### 边框属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `border` | `boolean` | 是否显示边框（支持运行时动态开关） |
| `borderColor` | `string` | 边框颜色 |
| `borderWidth` | `number` | 边框线宽（像素） |
| `borderOpacity` | `number` | 边框透明度 |

### 其他属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `dataSource` | `DataSource` | 数据源对象 |
| `visible` | `boolean` | 是否可见（会自动同步到边框线） |
| `renderOrder` | `number` | 渲染顺序 |
| `needsUpdate` | `boolean` | 设置为 `true` 后下一帧更新数据 |

---

## 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `dispose()` | `void` | 销毁多边形，释放几何体、材质和边框线资源。通常调用 `engine.remove(polygon)` 会自动触发，不需要手动调用 |
| `addEventListener(type, listener)` | `void` | 添加事件监听器（继承自 Three.js Object3D） |
| `removeEventListener(type, listener)` | `void` | 移除事件监听器（继承自 Three.js Object3D） |

---

## 常见场景

### 1. 基础多边形 + 纹理贴图

从 GeoJSON 文件加载多边形数据，设置拉伸和纹理贴图。

```javascript
// 引擎初始化（略）

const polygon = engine.add(
    new mapvthree.Polygon({
        color: 'red',
        extrude: true,
        extrudeValue: 10,
        mapScale: 0.05,
    })
);

// 设置纹理贴图（mapSrc 会自动开启 UV）
polygon.mapSrc = 'website_assets/images/traffic.png';

// 加载数据
let dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
dataSource.defineAttribute('color', p => Math.random() * 0xffffff);
polygon.dataSource = dataSource;

// 绑定事件
polygon.addEventListener('click', e => {
    console.log('click', e.entity);
});
```

### 2. 拉伸柱状体（行政区划）

为每个要素设置不同高度，实现拉伸的 3D 行政区划效果。

```javascript
// 引擎初始化（略，需 enableAnimationLoop: true）

// 加载行政区 GeoJSON
const response = await fetch('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json');
const json = await response.json();

json.features.forEach(feature => {
    const height = Math.random() * 4000 + 4000;
    feature.properties.height = height;

    const polygon = engine.add(new mapvthree.Polygon({
        vertexHeights: true,
        color: Math.random() * 0xffffff,
        opacity: 0.8,
        extrude: true,
        extrudeValue: 10,
    }));

    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON({
        type: 'FeatureCollection',
        features: [feature],
    });
    dataSource.defineAttribute('height');
    polygon.dataSource = dataSource;
});
```

### 3. 带边框的多边形

使用 `border` 系列参数为多边形添加描边效果，支持普通多边形、拉伸体和带洞多边形。

```javascript
// 引擎初始化（略）

// 普通多边形 + 边框
const polygon = engine.add(
    new mapvthree.Polygon({
        vertexColors: true,
        opacity: 0.6,
        transparent: true,
        border: true,
        borderColor: '#333333',
        borderWidth: 2,
    })
);
let dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
dataSource.defineAttribute('color', () => Math.random() * 0xffffff);
polygon.dataSource = dataSource;

// 拉伸体 + 边框
const extrudePolygon = engine.add(
    new mapvthree.Polygon({
        color: 'rgba(255, 140, 0, 0.8)',
        opacity: 0.8,
        transparent: true,
        extrude: true,
        extrudeValue: 50,
        border: true,
        borderColor: '#000000',
        borderWidth: 2,
    })
);
extrudePolygon.dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: {
        type: 'Polygon',
        coordinates: [[
            [103.740, 29.595], [103.745, 29.595],
            [103.745, 29.600], [103.740, 29.600],
            [103.740, 29.595],
        ]],
    },
}]);

// 运行时动态修改边框属性
polygon.borderColor = '#ff0000';
polygon.borderWidth = 4;
polygon.border = false; // 关闭边框
```

### 4. 自定义材质（侧面/顶面分别设置）

通过材质数组为拉伸体的侧面和顶面指定不同材质。

```javascript
import * as THREE from 'three';

// 引擎初始化（略，需 enableAnimationLoop: true）

const polygon = engine.add(
    new mapvthree.Polygon({
        vertexHeights: true,
        color: 'red',
        opacity: 0.8,
        extrude: true,
        extrudeValue: 10,
    })
);

polygon.geometry.useUV = true;

// 材质数组：[侧面材质, 顶面材质]
const sideMaterial = new mapvthree.ExtendMeshStandardMaterial({
    transparent: true,
    map: new THREE.TextureLoader().load('assets/textures/building/ac1.png'),
});
polygon.material = [
    sideMaterial,
    new THREE.MeshBasicMaterial({color: 0x008800}),
];

let dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
dataSource.defineAttribute('color', p => Math.random() * 0xffffff);
dataSource.defineAttribute('height', p => Math.random() * 40 + 10);
polygon.dataSource = dataSource;
```

---

## 注意事项

1. **数据源**：`Polygon` 必须通过 `dataSource` 设置数据后才能渲染。推荐使用 `GeoJSONDataSource.fromURL()` 或 `GeoJSONDataSource.fromGeoJSON()` 创建数据源。

2. **纹理贴图**：设置 `mapSrc` 时会自动开启 UV 映射，无需手动设置 `polygon.geometry.useUV = true`。但自定义材质（如直接替换 `polygon.material`）时需手动设置 `polygon.geometry.useUV = true`。

3. **顶点颜色**：启用 `vertexColors: true` 时，需在数据源中通过 `defineAttribute('color', ...)` 定义颜色属性。

4. **顶点高度**：启用 `vertexHeights: true` 时，需在数据源中通过 `defineAttribute('height')` 定义高度属性，每个要素在 `properties.height` 中携带高度值。

5. **自定义材质**：拉伸体支持材质数组 `[侧面材质, 顶面材质]`，可分别控制侧面和顶面的渲染效果。

6. **边框性能**：边框通过内部 `PolylineInternal` 实现，每个启用边框的 Polygon 会额外创建线条对象。大量多边形启用边框时需注意性能开销。

7. **坐标闭合**：边框功能会自动闭合坐标环，GeoJSON 数据中不需要手动确保首尾坐标一致。

8. **资源释放**：使用 `engine.remove(polygon)` 移除多边形时会自动调用 `dispose()` 释放资源（包括几何体、材质和边框线），无需手动调用。

9. **ECEF 与 EPSG:3857 模式差异**：`extrudeValue`、`zOffset`、顶点高度等参数在两种模式下方向不同——ECEF（地球模式）沿椭球面法线方向拉伸，EPSG:3857（平面模式）沿 Z 轴方向拉伸。单位均为米，但物理含义不同。
