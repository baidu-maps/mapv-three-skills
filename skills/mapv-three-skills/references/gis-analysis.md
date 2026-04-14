# GIS 空间分析

mapv-three 提供了丰富的 GIS 空间分析能力，包括通视分析、天际线分析、淹没分析、坡度分析、可视域分析、体积分析（方量计算）、缓冲区分析以及瓦片掩膜。

---

以下所有分析对象均需通过 `engine.add()` 添加到引擎后才能工作。假设 `engine` 已初始化（需开启 `enableAnimationLoop`）。

---

## 1. 通视分析 SightAnalysis

通视分析用于判断观察点到一组目标点之间是否存在遮挡。分析结果以可见线段（绿色）和不可见线段（红色）在地图上可视化展示。

### 构造函数

```js
const sightline = engine.add(new mapvthree.SightAnalysis());
```

`SightAnalysis` 无构造参数。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `viewPoint` | `number[]` | - | 观察点坐标 `[经度, 纬度, 高度]` |
| `distanceBias` | `number` | `0.1` | 起点附近忽略距离（米），避免射线撞到观察点自身所在的物体 |
| `visibleColor` | `string` | `'green'` | 可见区域连线颜色 |
| `invisibleColor` | `string` | `'red'` | 不可见区域连线颜色 |
| `occlusionPoints` | `number[][]` | (只读) | 遮挡点的经纬度列表 |
| `targetPoints` | `Array<{name, position}>` | (只读) | 目标点列表 |

### 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `addTargetPoint(target)` | `number[]` 或 `{position: number[], name?: string}` | 添加目标点 |
| `removeTargetPoint(nameOrIndex)` | `string \| number` | 按名称或索引移除目标点 |
| `removeAllTargetPoints()` | 无 | 清空所有目标点 |
| `build()` | 无 | 执行通视分析（全量重新检测） |
| `close()` | 无 | 关闭通视分析，清理所有结果 |

### 完整示例

```js
// 添加场景中的建筑物
const polygon = engine.add(new mapvthree.Polygon({
    extrude: true,
    extrudeValue: 50,
    vertexHeights: true,
    vertexColors: true,
}));
let dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
dataSource.defineAttribute('color', () => Math.random() * 0xffffff);
dataSource.defineAttribute('height', () => 10 + Math.random() * 40);
polygon.dataSource = dataSource;

// 创建通视分析
const sightline = engine.add(new mapvthree.SightAnalysis());

// 设置观察点
sightline.viewPoint = [103.756, 29.604, 50];

// 添加多个目标点
sightline.addTargetPoint({ position: [103.757, 29.607, 0], name: '目标1' });
sightline.addTargetPoint({ position: [103.758, 29.605, 0], name: '目标2' });
sightline.addTargetPoint([103.755, 29.606, 10]); // 也可以直接传数组

// 延迟执行，等待几何体生成完成
setTimeout(() => sightline.build(), 1000);

// 获取遮挡点
console.log(sightline.occlusionPoints);

// 动态修改
sightline.visibleColor = '#00ff00';
sightline.invisibleColor = '#ff0000';
sightline.distanceBias = 5;
sightline.build(); // 需要重新 build 以更新结果

// 关闭分析
sightline.close();
```

---

## 2. 天际线分析 SkylineAnalysis

> **已知问题：** SkylineAnalysis 当前存在渲染 bug，天际线可能无法正常显示。

天际线分析从当前视角检测建筑物/地形与天空的分界线，绘制天际线轮廓并支持获取天空遮挡率和天际线轮廓数据。

### 构造函数

```js
const skylineAnalysis = engine.add(new mapvthree.SkylineAnalysis());
```

`SkylineAnalysis` 无构造参数。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | `boolean` | `false` | 是否启用天际线分析 |
| `color` | `string` | `'#ff0000'` | 天际线颜色 |
| `lineWidth` | `number` | `5` | 天际线宽度（像素） |
| `skyThreshold` | `number` | `0.9999` | 天空深度阈值 |
| `skyOcclusionRatio` | `number` | (只读) | 天空遮挡率，值域 [0, 1]。0 = 全天空（无遮挡），1 = 全被遮挡 |
| `skylineProfile` | `Array<{x, y}>` | (只读) | 天际线轮廓数据点数组，x 为归一化屏幕 x 坐标，y 为天际线高度 |

### 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `close()` | 无 | 关闭天际线分析（等同于 `enabled = false`） |
| `readSkylineData()` | 无 | 从 GPU 回读天际线数据，返回 `Float32Array`（每列的天际线 y 坐标，归一化到 [0,1]），无数据返回 `null` |

### 完整示例

```js
const skylineAnalysis = engine.add(new mapvthree.SkylineAnalysis());
skylineAnalysis.color = '#ff0000';
skylineAnalysis.lineWidth = 5;

// 启用分析
skylineAnalysis.enabled = true;

// 获取天空遮挡率
console.log('天空遮挡率:', skylineAnalysis.skyOcclusionRatio); // 如 0.35

// 获取天际线轮廓数据
const profile = skylineAnalysis.skylineProfile;
// [{x: 0.1, y: 0.45}, {x: 0.2, y: 0.52}, ...]

// 从 GPU 回读原始天际线数据
const rawData = skylineAnalysis.readSkylineData();
// Float32Array，每个元素为该列的天际线 y 坐标

// 关闭分析
skylineAnalysis.close();
```

---

## 3. 淹没分析 FloodAnalysis

淹没分析用于模拟水面上涨过程，在指定的多边形区域内，水面从最小高度逐步上升至最大高度。

### 构造函数

```js
const floodAnalysis = new mapvthree.FloodAnalysis();
engine.add(floodAnalysis);
```

`FloodAnalysis` 无构造参数。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `dataSource` | `DataSource` | `null` | 淹没区域范围（面类型 DataSource） |
| `maxVisibleValue` | `number` | `0` | 最大淹没高度 |
| `minVisibleValue` | `number` | `0` | 最小淹没高度（起始高度） |
| `floodSpeed` | `number` | `50` | 淹没速度（米/秒） |
| `enabled` | `boolean` | `false` | 是否启用淹没分析 |
| `color` | `string` | `'#007be6'` | 淹没区颜色 |
| `opacity` | `number` | `0.5` | 淹没区透明度 |
| `floodState` | `number` | (只读) | 淹没状态：0-未开始，1-正在淹没，2-淹没结束 |

### 方法

| 方法 | 说明 |
|------|------|
| `reset()` | 重置淹没分析，回到初始状态 |
| `destroy()` | 销毁淹没分析对象 |

### 完整示例

```js
// 创建淹没分析
const floodAnalysis = new mapvthree.FloodAnalysis();
engine.add(floodAnalysis);

// 设置淹没区域
floodAnalysis.dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    geometry: {
        type: 'Polygon',
        coordinates: [[
            [105.478, 29.384],
            [105.732, 29.349],
            [105.832, 29.449],
            [105.478, 29.384],
        ]],
    },
}]);

// 设置淹没参数
floodAnalysis.maxVisibleValue = 450;
floodAnalysis.minVisibleValue = 250;
floodAnalysis.floodSpeed = 50;
floodAnalysis.color = '#007be6';
floodAnalysis.opacity = 0.5;

// 点击地图开始淹没
engine.map.addEventListener('click', () => {
    floodAnalysis.enabled = true;
});
```

### 注意事项

- `maxVisibleValue` 必须大于 `minVisibleValue`。
- 设置 `enabled = false` 会自动重置淹没状态。
- 淹没速度受渲染帧率影响，实际速度 = `floodSpeed * frameTime / 1000`。

---

## 4. 坡度分析 SlopeAnalysis

坡度分析用于对指定区域内的地形进行坡度计算和颜色可视化，不同坡度范围用不同颜色表示。

### 构造函数

```js
const slopeAnalysis = new mapvthree.SlopeAnalysis(options?);
engine.add(slopeAnalysis);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.colorStop` | `Object` | 见下方 | 坡度颜色分段，key 为坡度值(0-90)，value 为颜色 |

默认 colorStop：
```js
{
    0.5: 'rgb(85,182,43)',   // 平坦
    2: 'rgb(135,211,43)',
    5: 'rgb(204,244,44)',
    15: 'rgb(245,233,44)',
    35: 'rgb(255,138,43)',
    55: 'rgb(255,84,43)',
    90: 'rgb(255,32,43)',    // 陡峭
}
```

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | `boolean` | `false` | 是否启用坡度分析 |
| `coverageArea` | `number[][]` | `[]` | 分析区域坐标数组（至少3个点） |
| `colorStop` | `Object` | 见上方 | 坡度颜色分段（getter/setter） |

### 方法

| 方法 | 说明 |
|------|------|
| `build()` | 执行坡度分析 |
| `close()` | 结束坡度分析，释放资源 |

### 完整示例

```js
const slopeAnalysis = new mapvthree.SlopeAnalysis({
    colorStop: {
        0.5: 'rgb(85,182,43)',
        15: 'rgb(245,233,44)',
        55: 'rgb(255,84,43)',
        90: 'rgb(255,32,43)',
    },
});
engine.add(slopeAnalysis);

// 指定分析区域
slopeAnalysis.coverageArea = [
    [113.073, 25.221],
    [113.095, 25.230],
    [113.103, 25.212],
    [113.072, 25.203],
];

// 启用并构建
slopeAnalysis.enabled = true;
slopeAnalysis.build();

// 结束分析
// slopeAnalysis.close();
```

---

## 5. 可视域分析 ViewShedAnalysis

可视域分析基于 Shadow Map 技术，从观察点出发计算视锥范围内哪些区域可见（绿色）、哪些被遮挡（红色），并在屏幕上叠加显示。

### 构造函数

```js
const viewshed = new mapvthree.ViewShedAnalysis();
engine.add(viewshed);
```

`ViewShedAnalysis` 无构造参数。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `viewPosition` | `number[]` | - | 观察点 `[经度, 纬度, 高度]` |
| `targetPosition` | `number[]` | - | 目标点 `[经度, 纬度, 高度]`，用于确定视线方向 |
| `distance` | `number` | (自动) | 可视域分析距离，默认自动根据观察点和目标点计算 |
| `fov` | `number` | `45` | 视场角（度） |
| `visibleColor` | `number \| string` | `0x00ff00` | 可见区域颜色 |
| `invisibleColor` | `number \| string` | `0xff0000` | 不可见区域颜色 |

### 方法

| 方法 | 说明 |
|------|------|
| `build(options?)` | 开始可视域分析。options 支持 `{opacity: number}`（叠加透明度，默认 1） |
| `close()` | 关闭可视域分析 |

### 完整示例

```js
// 添加建筑物
const polygon = engine.add(new mapvthree.Polygon({
    extrude: true,
    extrudeValue: 50,
    vertexHeights: true,
    vertexColors: true,
}));
let dataSource = await mapvthree.GeoJSONDataSource.fromURL('data/geojson/polygon.geojson');
dataSource.defineAttribute('color', () => Math.random() * 0xffffff);
dataSource.defineAttribute('height', () => 10 + Math.random() * 40);
polygon.dataSource = dataSource;

// 创建可视域分析
const viewshed = new mapvthree.ViewShedAnalysis();
engine.add(viewshed);

// 设置观察点和目标点
viewshed.viewPosition = [103.756, 29.604, 50];
viewshed.targetPosition = [103.759, 29.615, 0];

// 开始分析
viewshed.build();

// 动态调整参数
viewshed.fov = 60;
viewshed.distance = 2000;
viewshed.visibleColor = '#00ff00';
viewshed.invisibleColor = '#ff0000';

// 关闭分析
// viewshed.close();
```

### 注意事项

- 可视域分析会在场景中显示视锥体线框辅助元素。
- 设置 `viewPosition` 或 `targetPosition` 后如需刷新结果需重新 `build()`。`fov`、`distance` 等参数可在 `build()` 前后任意时机设置。
- 支持地球模式和平面模式，地球模式下 up 向量会自动设为椭球法线方向。

---

## 6. 体积分析 VolumeLandAnalysis

体积分析（方量计算）用于在指定区域内计算挖方和填方的面积与体积，支持显示采样点。

### 构造函数

```js
const volumeAnalysis = new mapvthree.VolumeLandAnalysis(engine, options);
engine.add(volumeAnalysis);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `engine` | `Engine` | 引擎实例 |
| `options.terrain` | `Object` | 地形对象 |
| `options.bottomImg` | `string` | 开挖底部纹理路径（可选） |
| `options.wallImg` | `string` | 开挖侧壁纹理路径（可选） |
| `options.mapScale` | `number` | 地图缩放比例（可选） |
| `options.terrainClipPlanOpts` | `Object` | 裁剪面参数（可选） |

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `coveragePosition` | `number[][]` | `[]` | 分析区域坐标数组（经纬度，至少3个点） |
| `surfaceHeight` | `number` | `0` | 填挖基准高度 |
| `interpolateNum` | `number` | `30` | 每行采样插值数，值越大精度越高，计算量越大 |
| `showSamplePoint` | `boolean` | `false` | 是否显示采样点（红=挖方，黄=填方） |
| `cutArea` | `number` | (只读) | 挖方面积（平方米） |
| `cutVolume` | `number` | (只读) | 挖方体积（立方米） |
| `fillArea` | `number` | (只读) | 填方面积（平方米） |
| `fillVolume` | `number` | (只读) | 填方体积（立方米） |

### 方法

| 方法 | 说明 |
|------|------|
| `build()` | 执行方量分析 |
| `clear()` | 清除分析结果 |

### 完整示例

```js
// 添加地形
const mapView = engine.add(new mapvthree.MapView({
    terrainProvider: new mapvthree.CesiumTerrainTileProvider({
        url: 'http://terrain-server/dem',
    }),
}));

// 创建体积分析
const volumeAnalysis = new mapvthree.VolumeLandAnalysis(engine, {
    terrain: mapView,
});
engine.add(volumeAnalysis);

// 设置分析区域
volumeAnalysis.coveragePosition = [
    [113.046, 25.241],
    [113.049, 25.245],
    [113.053, 25.241],
    [113.052, 25.235],
    [113.044, 25.235],
];

// 设置基准高度
volumeAnalysis.surfaceHeight = 300;

// 提高精度
volumeAnalysis.interpolateNum = 50;

// 显示采样点
volumeAnalysis.showSamplePoint = true;

// 执行分析
volumeAnalysis.build();

// 获取结果
console.log('挖方面积:', volumeAnalysis.cutArea, '平方米');
console.log('挖方体积:', volumeAnalysis.cutVolume, '立方米');
console.log('填方面积:', volumeAnalysis.fillArea, '平方米');
console.log('填方体积:', volumeAnalysis.fillVolume, '立方米');
```

### 注意事项

- 高度数据通过射线检测获取，请确保地形瓦片在当前视图内以保证准确性。
- `interpolateNum` 越大精度越高，但计算量也越大，建议根据实际需求设置。
- 未开启 `showSamplePoint` 时，默认以填挖可视化模式展示。

---

## 7. 缓冲区分析 BufferAnalysis

缓冲区分析用于对线或面 GeoJSON 数据生成指定半径的缓冲区域。

### 构造函数

```js
const bufferAnalysis = new mapvthree.BufferAnalysis(options?);
engine.add(bufferAnalysis);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.radius` | `number` | `10` | 缓冲距离（米） |
| `options.type` | `string` | `'line'` | 缓冲类型：`'line'` 或 `'polygon'` |
| `options.bufferParameters` | `Object` | `{color: '#007be6', transparent: true, opacity: 0.5}` | 缓冲区面样式参数 |
| `options.lineParameters` | `Object` | `{color: '#5c5dfd'}` | 线条样式参数 |

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `data` | `GeoJSON` | 输入 GeoJSON 数据 |
| `radius` | `number` | 缓冲半径（米） |
| `bufferType` | `string` | 缓冲类型（`'line'` 或 `'polygon'`） |
| `bufferData` | `GeoJSON` | (只读) 缓冲结果 GeoJSON 数据 |

### 方法

| 方法 | 说明 |
|------|------|
| `build()` | 执行缓冲区分析 |
| `clear()` | 清空缓冲区分析数据 |

### 完整示例

```js
const bufferAnalysis = new mapvthree.BufferAnalysis({
    radius: 50,
    type: 'line',
});
engine.add(bufferAnalysis);

// 设置输入数据
bufferAnalysis.data = {
    type: 'FeatureCollection',
    features: [{
        type: 'Feature',
        geometry: {
            type: 'LineString',
            coordinates: [[116.39, 39.90], [116.40, 39.91], [116.41, 39.90]],
        },
    }],
};

// 执行分析
bufferAnalysis.build();

// 获取缓冲结果
console.log(bufferAnalysis.bufferData);

// 修改半径重新分析
bufferAnalysis.radius = 100;
bufferAnalysis.build();

// 清空
bufferAnalysis.clear();
```

---

## 8. 瓦片掩膜 TileMask

TileMask 通过 Stencil Buffer 实现对瓦片图层的区域裁剪渲染，指定一个 GeoJSON 多边形区域作为掩膜范围，只显示该区域内的瓦片内容。支持对 MapView 或单个 TileProvider 进行掩膜。

### 构造函数

```js
const tileMask = engine.add(new mapvthree.TileMask(options));
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `options.targets` | `Array<MapView\|TileProvider>` | 是 | 要应用掩膜的目标列表，支持 MapView 或 TileProvider |
| `options.region` | `Object\|GeoJSONDataSource` | 是 | 掩膜区域，GeoJSON 格式或 GeoJSONDataSource 实例 |

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `region` | `Object\|GeoJSONDataSource` | 掩膜区域（getter/setter），设置后自动更新 |
| `enabled` | `boolean` | 是否启用掩膜 |
| `targets` | `Array<TileProvider>` | (只读) 目标 provider 列表 |

### 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `addTarget(target)` | `MapView\|TileProvider` | 添加掩膜目标 |
| `removeTarget(target)` | `TileProvider` | 移除掩膜目标 |
| `dispose()` | 无 | 清理资源 |

### 完整示例

```js
// 创建掩膜，只显示指定区域的地图内容
const geojson = {
    type: 'FeatureCollection',
    features: [{
        type: 'Feature',
        geometry: {
            type: 'Polygon',
            coordinates: [[[116.1, 39.7], [116.7, 39.7], [116.7, 40.1], [116.1, 40.1], [116.1, 39.7]]],
        },
    }],
};

const tileMask = engine.add(new mapvthree.TileMask({
    targets: [mapView],
    region: geojson,
}));

// 动态更新掩膜区域
tileMask.region = newGeoJSON;

// 临时禁用掩膜
tileMask.enabled = false;

// 重新启用
tileMask.enabled = true;

// 移除掩膜
engine.remove(tileMask);
```

### 注意事项

- 目标 TileProvider 需要设置 `supportsMask = true` 才能被掩膜（如 BaiduVectorTileProvider）。
- 传入 MapView 时会自动从所有 surface 中提取支持掩膜的 provider。

---

## 注意事项汇总

1. 通视分析和可视域分析依赖场景中的几何体，请确保场景加载完成后再调用 `build()`。
2. 使用完毕后，务必调用 `close()` 或 `engine.remove()` 释放资源。
3. VolumeLandAnalysis 构造时需传入 `engine` 实例。
4. TileMask 的目标 TileProvider 需要支持 `supportsMask = true`。
