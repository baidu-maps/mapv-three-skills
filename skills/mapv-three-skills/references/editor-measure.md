# Editor 编辑器与 Measure 测量工具

---

## 一、Editor 编辑器

`Editor` 是统一的地图要素绘制与编辑组件，支持多边形、线、点、圆、矩形五种几何类型的绘制、编辑、样式管理和数据导入导出。

### 1.1 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 创建引擎
const engine = new mapvthree.Engine(container, {
    rendering: { enableAnimationLoop: true },
});

// 创建编辑器
const editor = engine.add(new mapvthree.Editor({
    type: mapvthree.Editor.DrawerType.POLYGON,
    enableMidpointHandles: true,
}));

// 设置样式
editor.setStyle({
    fillColor: '#ff0000',
    fillOpacity: 0.5,
    strokeColor: '#ffffff',
    strokeWidth: 2,
});

// 开始绘制
editor.start();

// 监听绘制完成事件
editor.addEventListener('created', () => {
    const data = editor.exportData();
    console.log('绘制完成', data);
});
```

### 1.2 构造函数

```javascript
new mapvthree.Editor(options?)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `string` | `'polygon'` | 默认绘制类型，见 DrawerType |
| `showLabel` | `boolean` | `false` | 是否显示测量标签（`true` 时为测量模式） |
| `singleMode` | `boolean` | `false` | 单要素模式，绘制新要素时自动清除旧要素 |
| `enableMidpointHandles` | `boolean` | `false` | 编辑时是否显示中点控制点（用于在边上添加顶点） |
| `continuousDrawing` | `boolean` | `false` | 是否启用连续绘制模式 |
| `renderOptions.depthTest` | `boolean` | `true` | 是否启用深度测试 |
| `renderOptions.transparent` | `boolean` | `true` | 是否启用透明度 |
| `renderOptions.renderOrder` | `number` | `0` | 渲染顺序 |
| `controlPoints.vertex.color` | `string` | `'#ffffff'` | 顶点控制点颜色 |
| `controlPoints.vertex.size` | `number` | `10` | 顶点控制点大小 |
| `controlPoints.midpoint.color` | `string` | `'#ffff00'` | 中点控制点颜色 |
| `controlPoints.midpoint.size` | `number` | `8` | 中点控制点大小 |

### 1.3 DrawerType 绘制类型常量

通过 `mapvthree.Editor.DrawerType` 或直接导入 `mapvthree.DrawerType` 访问。

| 常量 | 值 | 说明 |
|------|----|------|
| `POLYGON` | `'polygon'` | 多边形 |
| `LINE` | `'line'` | 线段/折线 |
| `POINT` | `'point'` | 点 |
| `CIRCLE` | `'circle'` | 圆形 |
| `RECTANGLE` | `'rectangle'` | 矩形 |

### 1.4 属性

| 属性 | 类型 | 读写 | 说明 |
|------|------|------|------|
| `type` | `string` | 读写 | 当前绘制类型 |
| `showLabel` | `boolean` | 读写 | 是否显示测量标签 |
| `singleMode` | `boolean` | 只读 | 是否为单要素模式 |
| `isDrawing` | `boolean` | 只读 | 当前是否正在绘制 |
| `isEditing` | `boolean` | 只读 | 当前是否正在编辑 |
| `renderOptions` | `Object` | 只读 | 渲染选项配置 |

### 1.5 方法

#### 绘制控制

##### `start(options?)`

开始绘制操作。

```javascript
editor.type = mapvthree.Editor.DrawerType.LINE;
editor.start({ continuous: true }); // 连续绘制模式
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `options.continuous` | `boolean` | 是否启用连续绘制，绘制完一个要素后自动开始下一个 |

##### `stop()`

停止当前绘制操作。

```javascript
editor.stop();
```

#### 编辑控制

##### `enableEdit(featureIdOrFilter?, createMeasureLabels?)`

启用要素编辑模式。

```javascript
// 编辑所有要素
editor.enableEdit();

// 编辑指定 ID 的要素
editor.enableEdit('feature-123');

// 通过过滤函数编辑
editor.enableEdit(feature => feature.properties.shape === 'polygon');
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `featureIdOrFilter` | `string \| Function` | 要素 ID 或过滤函数，不传则编辑全部 |
| `createMeasureLabels` | `boolean` | 是否为编辑要素创建测量标签，默认 `false` |

##### `disableEdit()`

退出编辑模式。

```javascript
editor.disableEdit();
```

#### 数据操作

##### `exportData(type?)`

导出数据为 GeoJSON FeatureCollection 格式。

```javascript
// 导出所有数据
const allData = editor.exportData();

// 按类型导出
const polygons = editor.exportData(mapvthree.Editor.DrawerType.POLYGON);
const lines = editor.exportData(mapvthree.Editor.DrawerType.LINE);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `DrawerType` | 要导出的要素类型，不传则导出全部 |

**返回值：** `Object` -- GeoJSON FeatureCollection

##### `importData(data, options?)`

导入 GeoJSON 数据。

```javascript
const geojson = {
    type: 'FeatureCollection',
    features: [
        {
            type: 'Feature',
            id: 'my-polygon',
            geometry: {
                type: 'Polygon',
                coordinates: [[[116.51, 39.79], [116.52, 39.79], [116.52, 39.80], [116.51, 39.80], [116.51, 39.79]]],
            },
            properties: { name: '测试多边形', color: '#ff0000' },
        },
    ],
};

const importedFeatures = editor.importData(geojson, { clear: true });
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `data` | `Object` | -- | GeoJSON FeatureCollection 或 Feature 数据 |
| `options.clear` | `boolean` | `false` | 是否清除现有数据 |

**返回值：** `string[]` -- 导入的要素 ID 数组

##### `delete(type?)`

删除要素。

```javascript
// 删除所有要素
editor.delete();

// 删除指定类型
editor.delete(mapvthree.Editor.DrawerType.POLYGON);
```

**返回值：** `number` -- 删除的要素数量

##### `deleteById(featureIdOrIds)`

根据 ID 删除要素。

```javascript
// 删除单个
editor.deleteById('feature-1');

// 删除多个
editor.deleteById(['feature-1', 'feature-2', 'feature-3']);
```

**返回值：** `number` -- 成功删除的要素数量

#### 样式管理

##### `getStyle(type?)`

获取指定类型的默认样式。

```javascript
const style = editor.getStyle(mapvthree.Editor.DrawerType.POLYGON);
```

##### `setStyle(style, type?)`

设置样式。正在绘制时会实时更新当前绘制要素的样式。

```javascript
// 设置全局默认样式
editor.setStyle({
    fillColor: '#ff0000',
    fillOpacity: 0.5,
    strokeColor: '#ffffff',
    strokeWidth: 2,
});

// 设置特定类型样式
editor.setStyle({ strokeWidth: 5 }, mapvthree.Editor.DrawerType.LINE);
```

##### `updateFeatureStyle(featureId, style, replace?)`

更新已有要素的样式。

```javascript
editor.updateFeatureStyle('feature-123', {
    fillColor: '#00ff00',
    fillOpacity: 0.8,
});

// 完全替换样式（而非合并）
editor.updateFeatureStyle('feature-123', { fillColor: '#0000ff' }, true);
```

#### 显示/隐藏

##### `show(type?)`

显示要素。

```javascript
editor.show();                                          // 显示全部
editor.show(mapvthree.Editor.DrawerType.POLYGON);       // 按类型显示
```

##### `hide(type?)`

隐藏要素。

```javascript
editor.hide();                                          // 隐藏全部
editor.hide(mapvthree.Editor.DrawerType.LINE);          // 按类型隐藏
```

### 1.6 事件

通过 `addEventListener` / `removeEventListener` 监听事件。

| 事件名 | 触发时机 | 事件对象属性 |
|--------|----------|-------------|
| `start` | 调用 `start()` 时 | `{ drawerType, options }` |
| `drawStart` | 每次开始绘制一个新要素时 | `{ drawerType, options }` |
| `created` | 一个要素绘制完成时 | -- |
| `update` | 要素被修改时（绘制中、编辑中） | -- |
| `delete` | 要素被删除时 | -- |
| `featureClick` | 点击已有要素时 | `{ featureId }` |

```javascript
editor.addEventListener('created', () => {
    console.log('要素绘制完成');
    const data = editor.exportData();
});

editor.addEventListener('featureClick', (e) => {
    console.log('点击了要素:', e.featureId);
    editor.enableEdit(e.featureId);
});

editor.addEventListener('update', () => {
    console.log('要素已更新');
});

editor.addEventListener('delete', () => {
    console.log('要素已删除');
});
```

### 1.7 样式配置项

```javascript
{
    fillColor: '#3388ff',       // 填充颜色（多边形、圆、矩形）
    fillOpacity: 0.2,           // 填充透明度
    strokeColor: '#3388ff',     // 边框/线条颜色
    strokeWidth: 3,             // 边框/线条宽度（像素）
    strokeOpacity: 1,           // 边框/线条透明度
    pointRadius: 5,             // 点半径（仅 point 类型）
    radius: 100,                // 圆半径（仅 circle 类型，单位：米）
}
```

### 1.8 Feature 数据结构

导出的数据遵循 GeoJSON 规范，不同类型的要素对应关系如下：

| 绘制类型 | geometry.type | properties 特有字段 |
|----------|--------------|---------------------|
| `polygon` | `Polygon` | `shape: 'polygon'` |
| `line` | `LineString` | `shape: 'line'` |
| `point` | `Point` | `shape: 'point'` |
| `circle` | `Point` | `shape: 'circle', radius: number` |
| `rectangle` | `Polygon` | `shape: 'rectangle'` |

导入数据时，可通过 `properties.shape` 字段标记要素类型。圆形要素需在 `properties` 中提供 `radius`（单位：米）。

### 1.9 生命周期

```javascript
// 添加到引擎
const editor = engine.add(new mapvthree.Editor({ ... }));

// 从引擎移除（自动调用 dispose 清理资源）
engine.remove(editor);
```

---

## 二、Measure 测量工具

`Measure` 继承自 `Editor`，专门用于地图测量，默认开启标签显示（`showLabel: true`），支持距离、面积、点坐标三种测量类型。

### 2.1 快速开始

```javascript
// 创建测量工具
const measure = engine.add(new mapvthree.Measure({
    type: mapvthree.MeasureType.DISTANCE,
}));

// 设置样式
measure.setStyle({
    strokeColor: '#1890ff',
    strokeWidth: 2,
});

// 开始测量
measure.start({ continuous: false });

// 监听测量完成
measure.addEventListener('created', () => {
    const results = measure.getMeasurements();
    console.log('测量结果:', results);
});
```

### 2.2 构造函数

```javascript
new mapvthree.Measure(options?)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `MeasureType` | `'distance'` | 测量类型 |
| `continuousDrawing` | `boolean` | `false` | 是否连续测量 |
| `enableMidpointHandles` | `boolean` | `true` | 是否启用中点标记 |
| `renderOptions.depthTest` | `boolean` | `false` | 深度测试（默认关闭） |
| `renderOptions.transparent` | `boolean` | `true` | 透明度 |
| `renderOptions.renderOrder` | `number` | `100` | 渲染顺序 |

> 注意：`Measure` 自动设置 `showLabel: true` 和 `singleMode: true`。

### 2.3 MeasureType 测量类型常量

通过 `mapvthree.MeasureType` 或 `mapvthree.Measure.MeasureType` 访问。

| 常量 | 值 | 说明 | 对应绘制类型 |
|------|----|------|-------------|
| `DISTANCE` | `'distance'` | 距离测量 | `line` |
| `AREA` | `'area'` | 面积测量 | `polygon` |
| `POINT` | `'point'` | 坐标测量 | `point` |

### 2.4 方法

`Measure` 继承了 `Editor` 的所有方法（`start`、`stop`、`setStyle`、`exportData` 等），并扩展以下测量专用方法：

##### `setType(measureType)`

动态切换测量类型。

```javascript
measure.setType(mapvthree.MeasureType.AREA);
```

##### `getMeasurements()`

获取所有测量结果。

```javascript
const results = measure.getMeasurements();
```

##### `getMeasurementsByType(type)`

获取指定类型的测量结果。

```javascript
const distances = measure.getMeasurementsByType(mapvthree.MeasureType.DISTANCE);
```

##### `clear()`

清除所有测量结果（包括要素和标签）。

```javascript
measure.clear();
```

##### `clearByType(type)`

清除指定类型的测量结果。

```javascript
measure.clearByType(mapvthree.MeasureType.DISTANCE);
```

##### `enableEdit(featureIdOrFilter?)`

启用测量结果编辑。

```javascript
measure.enableEdit();                       // 编辑全部
measure.enableEdit('measurement-123');       // 编辑指定要素
```

##### `setLabelRenderer(renderFunction)`

自定义测量标签 DOM 渲染。

```javascript
measure.setLabelRenderer((value) => {
    const div = document.createElement('div');
    div.innerText = value.attributes.text;
    div.style.cssText = 'background: rgba(0,0,0,0.7); color: #fff; padding: 4px 8px; border-radius: 4px; font-size: 12px;';
    return div;
});
```

| 参数 | 说明 |
|------|------|
| `value.attributes.text` | 标签文本内容 |
| `value.attributes.isMain` | 是否为主标签（总距离/总面积标签） |

##### `setDistanceFormatter(formatter)`

自定义距离显示格式。

```javascript
measure.setDistanceFormatter((meters) => {
    if (meters < 1000) {
        return `${meters.toFixed(1)} 米`;
    }
    return `${(meters / 1000).toFixed(2)} 公里`;
});
```

##### `setAreaFormatter(formatter)`

自定义面积显示格式。

```javascript
measure.setAreaFormatter((sqMeters) => {
    if (sqMeters < 10000) {
        return `${sqMeters.toFixed(1)} 平方米`;
    } else if (sqMeters < 1000000) {
        return `${(sqMeters / 10000).toFixed(2)} 公顷`;
    }
    return `${(sqMeters / 1000000).toFixed(2)} 平方公里`;
});
```

##### `setPointFormatter(formatter)`

自定义坐标显示格式。

```javascript
measure.setPointFormatter((point) => {
    return `经度: ${point[0].toFixed(6)}, 纬度: ${point[1].toFixed(6)}`;
});
```

### 2.5 测量结果数据结构

#### 距离测量

```javascript
{
    featureId: 'string',
    type: 'distance',
    result: {
        totalDistance: 1234.56,                  // 总距离（米）
        formattedTotalDistance: '1.23 km',
        segments: [
            { distance: 500.12, formattedDistance: '500.1 m' },
            { distance: 734.44, formattedDistance: '734.4 m' },
        ],
        coordinates: [[lng1, lat1], [lng2, lat2], ...]
    }
}
```

#### 面积测量

```javascript
{
    featureId: 'string',
    type: 'area',
    result: {
        area: 50000.0,                          // 面积（平方米）
        formattedArea: '5.00 公顷',
        perimeter: 1000.0,                      // 周长（米）
        formattedPerimeter: '1.00 km',
        coordinates: [[lng1, lat1], [lng2, lat2], ...]
    }
}
```

#### 坐标测量

```javascript
{
    featureId: 'string',
    type: 'point',
    result: {
        longitude: 116.516303,
        latitude: 39.799170,
        altitude: 0,
        formattedPoint: '116.516303, 39.799170',
        coordinates: [116.516303, 39.799170]
    }
}
```

### 2.6 静态计算方法

> **注意：** 静态计算方法定义在内部的旧版 `Measure` 类（`src/threemap/controls/measure/Measure.js`）上，当前导出的新版 `mapvthree.Measure`（继承自 `Editor`）**不包含**这些静态方法。`MeasureManager` 内部使用它们进行计算，用户无需直接调用。
>
> 如果需要在业务代码中进行距离/面积计算，建议使用 `Measure` 实例的 `getMeasurements()` 获取测量结果。

### 2.7 测量样式参考

不同测量类型建议的样式配置：

```javascript
// 点测量样式
measure.setStyle({
    fillColor: '#ff4d4f',
    pointRadius: 10,
    fillOpacity: 1,
});

// 距离测量样式
measure.setStyle({
    strokeColor: '#1890ff',
    strokeWidth: 2,
    strokeOpacity: 1,
});

// 面积测量样式
measure.setStyle({
    fillColor: '#52c41a',
    fillOpacity: 0.3,
    strokeColor: '#52c41a',
    strokeWidth: 2,
    strokeOpacity: 1,
});
```

---

## 三、完整示例

### 3.1 编辑器完整示例

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 初始化引擎
const engine = new mapvthree.Engine(document.getElementById('map'), {
    rendering: { enableAnimationLoop: true, sky: null },
    map: { projection: 'ecef' },
});
engine.map.setCenter([116.516, 39.799]);
engine.map.setZoom(20);

// 添加底图和天空
engine.add(new mapvthree.MapView({
    imageryProvider: new mapvthree.BingImageryTileProvider(),
}));
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 3600 * 16.5;

// 创建编辑器
const editor = engine.add(new mapvthree.Editor({
    type: mapvthree.Editor.DrawerType.POLYGON,
    enableMidpointHandles: true,
    showLabel: true,            // 启用测量标签
}));

// 设置样式
editor.setStyle({
    fillColor: '#ff0000',
    fillOpacity: 0.5,
    strokeColor: '#ffffff',
    strokeWidth: 2,
});

// 监听事件
editor.addEventListener('created', () => {
    console.log('绘制完成', editor.exportData());
});

editor.addEventListener('featureClick', (e) => {
    editor.enableEdit(e.featureId);
});

editor.addEventListener('update', () => {
    console.log('数据更新', editor.exportData());
});

editor.addEventListener('delete', () => {
    console.log('数据删除');
});

// 开始绘制多边形
editor.start({ continuous: false });

// 切换为线段绘制
// editor.type = mapvthree.Editor.DrawerType.LINE;
// editor.start();

// 停止绘制
// editor.stop();

// 导入预设数据
// editor.importData(geojsonData, { clear: true });

// 导出数据
// const data = editor.exportData();
// const polygonOnly = editor.exportData(mapvthree.Editor.DrawerType.POLYGON);

// 清除所有
// editor.delete();

// 从引擎移除
// engine.remove(editor);
```

### 3.2 测量工具完整示例

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map'), {
    rendering: { enableAnimationLoop: true },
});
engine.map.setCenter([116.516, 39.799]);
engine.map.setZoom(18);

// 创建测量工具
const measure = engine.add(new mapvthree.Measure({
    type: mapvthree.MeasureType.DISTANCE,
}));

// 设置样式
measure.setStyle({
    strokeColor: '#1890ff',
    strokeWidth: 2,
    strokeOpacity: 1,
});

// 自定义距离格式化
measure.setDistanceFormatter((meters) => {
    return meters > 1000
        ? `${(meters / 1000).toFixed(2)} km`
        : `${meters.toFixed(0)} m`;
});

// 自定义标签渲染
measure.setLabelRenderer((value) => {
    const div = document.createElement('div');
    div.innerText = value.attributes.text;
    div.style.cssText = `
        background: rgba(0,0,0,0.75);
        color: #fff;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
    `;
    return div;
});

// 监听测量完成
measure.addEventListener('created', () => {
    const results = measure.getMeasurements();
    console.log('测量结果:', results);
});

// 开始距离测量
measure.start({ continuous: false });

// 切换到面积测量
// measure.setType(mapvthree.MeasureType.AREA);
// measure.setStyle({ fillColor: '#52c41a', fillOpacity: 0.3, strokeColor: '#52c41a', strokeWidth: 2 });
// measure.start();

// 切换到点测量
// measure.setType(mapvthree.MeasureType.POINT);
// measure.start();

// 停止测量
// measure.stop();

// 清除指定类型结果
// measure.clearByType(mapvthree.MeasureType.DISTANCE);

// 清除全部结果
// measure.clear();

// 从引擎移除
// engine.remove(measure);
```

### 3.3 单要素模式示例

```javascript
// 创建单要素模式编辑器（每次只保留最新绘制的要素）
const editor = engine.add(new mapvthree.Editor({
    type: mapvthree.Editor.DrawerType.POLYGON,
    enableMidpointHandles: true,
    singleMode: true,   // 启用单要素模式
}));

editor.addEventListener('created', () => {
    const data = editor.exportData();
    console.log('当前要素:', data);
});

editor.start();
```

### 3.4 连续绘制模式示例

```javascript
const editor = engine.add(new mapvthree.Editor({
    type: mapvthree.Editor.DrawerType.POLYGON,
}));

// 开启连续绘制：完成一个要素后自动开始绘制下一个
editor.start({ continuous: true });

// 手动停止连续绘制
// editor.stop();
```

### 3.5 数据导入导出示例

```javascript
// 导入 GeoJSON 数据
const geojson = {
    type: 'FeatureCollection',
    features: [
        {
            type: 'Feature',
            id: 'polygon-1',
            geometry: {
                type: 'Polygon',
                coordinates: [[[116.51, 39.79], [116.52, 39.79], [116.52, 39.80], [116.51, 39.80], [116.51, 39.79]]],
            },
            properties: { name: '多边形', color: '#ff0000', opacity: 0.5 },
        },
        {
            type: 'Feature',
            id: 'circle-1',
            geometry: {
                type: 'Point',
                coordinates: [116.515, 39.795],
            },
            properties: { shape: 'circle', radius: 100, color: '#00ff00' },
        },
    ],
};

editor.importData(geojson, { clear: true });

// 进入编辑模式
editor.enableEdit();

// 导出修改后的数据
const result = editor.exportData();
console.log(JSON.stringify(result, null, 2));
```

