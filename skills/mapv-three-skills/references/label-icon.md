# Label 标签、IconPoint 图标点、Icon 图标与 Text 文字

## 如何选择

- **大部分场景使用 `Label`**：支持文本、图标、图文组合，功能最全面，是首选组件。Label 可以覆盖 Text 的所有功能。
- **纯图标展示使用 `Icon`**：支持贴地、跳动动画、旋转等特性，适合不需要文字的纯图标场景。
- **海量图标且追求性能使用 `IconPoint`**：基于 GL_POINTS 渲染，性能开销最低，只面向屏幕显示，适合需要渲染大量简单图标且对性能敏感的场景。
- **纯文字标注可使用 `Text`**：基于 Canvas 绘制文字纹理，支持自定义字体、描边。但 Label 已覆盖 Text 的所有功能且使用 SDF 渲染效果更好，推荐优先使用 Label。

---

## Label

`Label` 继承自 `GeoMesh`，用于在地图上渲染文本标签、图标标签或两者组合。支持 SDF 文字渲染、图标雪碧图合并、淡入淡出动画、碰撞检测等特性。适合批量管理大量标签数据（如整体显隐控制）。对于少量标签，推荐使用 `RenderingLabel`。

### 构造参数

通过 `new mapvthree.Label(parameters)` 创建实例，`parameters` 为配置对象，支持以下属性：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `string` | `'icon'` | 绘制类型，支持 `'icon'`、`'text'`、`'icontext'` |
| `vertexIcons` | `boolean` | `false` | 数据中是否携带图标 URL 资源 |
| `flat` | `boolean` | `false` | 是否贴地显示 |
| `enableFade` | `boolean` | `false` | 是否启用淡入淡出效果（数据需指定 `id`） |
| `fadeDuration` | `number` | - | 淡入淡出持续时间 |
| `mapSrc` | `string` | - | 统一的图标 URL（当 `vertexIcons` 为 `false` 时使用） |
| `transparent` | `boolean` | `false` | 是否启用透明 |
| `depthTest` | `boolean` | - | 是否进行深度测试 |
| `depthWrite` | `boolean` | - | 是否写入深度缓冲 |
| `opacity` | `number` | - | 整体透明度 |
| `keepSize` | `boolean` | - | 是否保持像素大小（不随缩放变化） |
| `pixelOffset` | `Array<number>` | `[0, 0]` | 像素偏移 |
| `positionOffset` | `Array<number>` | `[0, 0]` | 坐标偏移 |
| `jointLine` | `boolean` | `false` | 是否启用引线（自动创建 JointLine 组件） |

#### 文字样式参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `textSize` | `number` | `16` | 文字大小 |
| `textFamily` | `string` | `'sans-serif'` | 文字字体 |
| `textFillStyle` | `string \| Array<number>` | `[255, 255, 255]` | 文字颜色，支持 CSS 颜色字符串或 RGBA 数组 |
| `textStrokeStyle` | `string \| Array<number>` | `[0, 0, 0]` | 文字描边颜色 |
| `textStrokeWidth` | `number` | `0` | 文字描边宽度 |
| `textAnchor` | `string` | `'center'` | 文字锚点位置，见下方锚点说明 |
| `textOffset` | `Array<number>` | `[0, 0]` | 文字偏移量 `[x, y]` |
| `textWeight` | `string` | `'400'` | 文字粗细 |
| `textPadding` | `Array<number>` | `[0, 2]` | 文字内边距 `[x, y]` |
| `textAlign` | `string` | `'center'` | 文本对齐方式，支持 `'left'`、`'right'`、`'center'` |
| `padding` | `Array<number>` | `[2, 2]` | 文字与图标间距 `[x, y]` |

**textAnchor 锚点取值：**

`'center'`（默认）、`'left'`、`'right'`、`'top'`、`'bottom'`、`'top-left'`、`'top-right'`、`'bottom-left'`、`'bottom-right'`

锚点决定了图标相对于文字的位置。例如 `'left'` 表示图标在左、文字在右，`'right'` 表示图标在右、文字在左，`'top'` 表示图标在上、文字在下，`'bottom'` 表示图标在下、文字在上。

#### 图标参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `iconWidth` | `number` | `40` | 图标宽度（当 `type` 为 `'text'` 时自动设为 0） |
| `iconHeight` | `number` | `40` | 图标高度（当 `type` 为 `'text'` 时自动设为 0） |
| `useIconScale` | `boolean` | `false` | 是否使用图片自身的宽高比（高度由宽度和图片比例计算） |
| `rotateZ` | `number` | `0` | 旋转角度（弧度） |

### 数据级自定义样式

`Label` 支持通过数据源 `properties` 携带逐点自定义样式，使用 `defineAttribute` 方法声明需要的属性映射：

| 数据属性 | 类型 | 说明 |
|----------|------|------|
| `icon` | `string` | 图标 URL 路径（需要 `vertexIcons: true`） |
| `iconSize` | `Array<number>` | 图标尺寸 `[width, height]`，覆盖全局 `iconWidth`/`iconHeight` |
| `text` | `string \| Array<string>` | 文字内容。传入数组时，每个元素渲染为独立文字段，支持不同样式 |
| `textSize` | `number \| Array<number>` | 文字大小 |
| `textFillStyle` | `string \| Array<string>` | 文字颜色 |
| `textStrokeStyle` | `string \| Array<string>` | 文字描边颜色 |
| `textStrokeWidth` | `number \| Array<number>` | 文字描边宽度 |
| `textOffset` | `Array<number>` | 文字偏移 `[x, y]` |
| `textAnchor` | `string \| Array<string>` | 文字锚点 |
| `textWeight` | `string \| Array<string>` | 文字粗细 |
| `rotateZ` | `number` | 旋转角度（弧度） |
| `textDrawOnIcon` | `boolean` | 文字是否绘制在图标上（用于地铁线、道路名等场景） |
| `iconFixedWidth` | `number` | 当 `textDrawOnIcon` 为 `true` 时，图标左右预留宽度，默认 `4` |
| `iconOpacity` | `number` | 图标透明度 |
| `iconUvs` | `Array<number>` | 自定义图标 UV 坐标（8 个值） |
| `offset` | `Array<number>` | 图标偏移 `[x, y]` |
| `tolerance` | `number` | 碰撞检测容差 |

> **多文本段支持：** 当 `text` 为数组时，`textSize`、`textFillStyle`、`textStrokeStyle`、`textStrokeWidth`、`textOffset`、`textWeight`、`textAnchor` 也可传入对应长度的数组，实现每段文字独立样式。

### 代理属性

以下属性可在实例创建后动态修改：

- `mapSrc` - 图标 URL
- `transparent` - 透明开关
- `depthTest` - 深度测试
- `depthWrite` - 深度写入
- `opacity` - 透明度
- `flat` - 贴地模式
- `keepSize` - 保持像素大小
- `vertexColors` - 顶点颜色
- `positionOffset` - 坐标偏移
- `pixelOffset` - 像素偏移

### 使用示例

#### 纯文本标签

```javascript
const textLabel = engine.add(new mapvthree.Label({
    type: 'text',
    textSize: 16,
    textFillStyle: 'rgb(255, 8, 8)',
    textStrokeStyle: 'rgba(0, 0, 0, 1)',
    textStrokeWidth: 2,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.404, 39.915] },
    properties: { text: '北京市' }
}]);
data.defineAttribute('text', 'text');
textLabel.dataSource = data;
```

#### 图标标签

```javascript
const iconLabel = engine.add(new mapvthree.Label({
    type: 'icon',
    vertexIcons: true,
    iconWidth: 40,
    iconHeight: 40,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.404, 39.915] },
    properties: { icon: 'path/icon.png' }
}]);
data.defineAttribute('icon', 'icon');
iconLabel.dataSource = data;
```

#### 图标 + 文字组合标签

```javascript
const label = engine.add(new mapvthree.Label({
    type: 'icontext',
    vertexIcons: true,
    textSize: 16,
    textFillStyle: 'rgb(255, 8, 8)',
    textStrokeStyle: 'rgba(0, 0, 0, 1)',
    textStrokeWidth: 2,
    iconWidth: 40,
    iconHeight: 40,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.404, 39.915] },
    properties: {
        icon: 'path/icon.png',
        text: '北京市',
        iconSize: [40, 40],
        textAnchor: 'left',
    }
}]);
data.defineAttribute('icon', 'icon');
data.defineAttribute('text', 'text');
data.defineAttribute('iconSize', 'iconSize');
data.defineAttribute('textAnchor', 'textAnchor');
label.dataSource = data;
```

#### 自定义逐点样式

```javascript
const label = engine.add(new mapvthree.Label({
    type: 'icontext',
    vertexIcons: true,
    textSize: 16,
    iconWidth: 40,
    iconHeight: 40,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.404, 39.915] },
    properties: {
        icon: 'path/icon.png',
        iconSize: [40, 40],
        text: '北京市',
        textSize: 16,
        textAnchor: 'center',
        textFillStyle: 'rgb(255, 8, 8)',
        textStrokeStyle: 'rgba(0, 0, 0, 1)',
        textStrokeWidth: 2,
        textOffset: [0, 0],
        rotateZ: 0,
    }
}]);
// 声明属性映射
data.defineAttribute('icon', 'icon');
data.defineAttribute('iconSize', 'iconSize');
data.defineAttribute('text', 'text');
data.defineAttribute('textSize', 'textSize');
data.defineAttribute('textFillStyle', 'textFillStyle');
data.defineAttribute('textStrokeStyle', 'textStrokeStyle');
data.defineAttribute('textStrokeWidth', 'textStrokeWidth');
data.defineAttribute('textOffset', 'textOffset');
data.defineAttribute('textAnchor', 'textAnchor');
data.defineAttribute('rotateZ', 'rotateZ');
label.dataSource = data;
```

#### 事件绑定

```javascript
label.addEventListener('click', e => {
    console.log('点击了标签', e);
});

label.addEventListener('mouseenter', e => {
    console.log('鼠标进入', e.entity.index);
});

label.addEventListener('mouseleave', e => {
    console.log('鼠标离开', e.entity.index);
});
```

---

## IconPoint

`IconPoint` 继承自 `GeoPoints`，是一个轻量级的图标点组件，使用 GL_POINTS 方式渲染。适合渲染大量简单的图标标记点。

### 构造参数

通过 `new mapvthree.IconPoint(parameters)` 创建实例：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `string` | - | 点颜色 |
| `vertexColors` | `boolean` | `false` | 是否通过数据携带颜色配置 |
| `size` | `number` | `30` | 点尺寸 |
| `vertexSizes` | `boolean` | `false` | 是否通过数据携带尺寸配置 |
| `mapSrc` | `string` | - | 统一的图标 URL 路径 |
| `vertexIcons` | `boolean` | `false` | 是否通过数据携带图标 URL 配置 |
| `transparent` | `boolean` | - | 是否透明 |
| `opacity` | `number` | `1` | 透明度整体系数 |
| `boxSize` | `number` | `100` | 雪碧图最大图标数量 |

### 单图标模式

所有数据点使用同一张图标图片，通过 `mapSrc` 指定：

```javascript
const iconPoint = engine.add(new mapvthree.IconPoint({
    mapSrc: 'path/to/icon.png',
    size: 30,
    color: 'rgba(255, 255, 255, 0)',
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON(pointData);
iconPoint.dataSource = data;
```

### 多图标模式（vertexIcons）

每个数据点可指定不同的图标，组件内部自动生成雪碧图纹理：

```javascript
const iconPoint = engine.add(new mapvthree.IconPoint({
    vertexSizes: true,
    vertexIcons: true,
    color: 'rgba(255, 255, 255, 0)',
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([
    {
        type: 'Feature',
        geometry: { type: 'Point', coordinates: [116.404, 39.915] },
        properties: {
            icon: 'https://example.com/icon-a.png',
            color: 'rgba(0, 250, 0, 0.7)',
            size: 20,
        },
    },
    {
        type: 'Feature',
        geometry: { type: 'Point', coordinates: [116.414, 39.915, 10] },
        properties: {
            icon: 'https://example.com/icon-b.png',
            color: 'yellow',
            size: 50,
        },
    },
]);
data.defineAttribute('color').defineAttribute('size').defineAttribute('icon');
iconPoint.dataSource = data;
```

### 数据属性

| 数据属性 | 类型 | 说明 | 需要启用 |
|----------|------|------|---------|
| `icon` | `string` | 图标 URL | `vertexIcons: true` |
| `color` | `string` | 点颜色 | `vertexColors: true` |
| `size` | `number` | 点尺寸 | `vertexSizes: true` |
| `offset` | `number` | 偏移量 | `vertexOffsets: true` |

### 代理属性

以下属性可在实例创建后动态修改：

- `color` - 点颜色
- `size` - 点尺寸
- `offset` - 偏移
- `transparent` - 透明开关
- `opacity` - 透明度
- `vertexColors` / `vertexSizes` / `vertexIcons` / `vertexOffsets` - 顶点数据开关
- `mapSrc` / `mapTexture` - 图标纹理

---

## Icon

`Icon` 继承自 `GeoMesh`，是 `IconPoint` 的增强版本，使用 Mesh 方式渲染。支持贴地/面向屏幕、跳动动画、旋转等高级特性，并具备更精确的射线拾取（raycast）能力。

### 构造参数

通过 `new mapvthree.Icon(parameters)` 创建实例：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `mapSrc` | `string` | `''` | 图标图片 URL |
| `vertexIcons` | `boolean` | `false` | 是否通过数据携带图标 URL |
| `vertexColors` | `boolean` | `false` | 是否通过数据携带颜色 |
| `width` | `number` | `12` | 图标宽度（像素） |
| `height` | `number` | `12` | 图标高度（像素） |
| `offset` | `Array<number>` | `[0, 0]` | 图标偏移量 `[x, y]` |
| `transparent` | `boolean` | `true` | 是否启用透明 |
| `opacity` | `number` | `1` | 整体透明度 |
| `keepSize` | `boolean` | `true` | 是否保持像素大小（不随地图缩放变化） |
| `flat` | `boolean` | `false` | 是否贴地显示 |
| `color` | `string` | - | 图标颜色叠加 |
| `rotateZ` | `number` | `0` | 旋转弧度（0~2Pi），仅 `flat: true` 时生效 |
| `vertexRotateZs` | `boolean` | `false` | 是否通过数据携带旋转角度 |
| `animationJump` | `boolean` | `false` | 是否启用跳动动画 |
| `jumpHeight` | `number` | `20` | 跳动高度 |
| `jumpSpeed` | `number` | `1` | 跳动速度 |
| `fallback` | `string` | - | 图标加载失败时的备用图片 URL |
| `boxSize` | `number` | `100` | 雪碧图最大图标数量 |
| `padding` | `Array<number>` | `[2, 2]` | 雪碧图内边距 |

### 使用示例

#### 基础图标（面向屏幕）

```javascript
const icon = engine.add(new mapvthree.Icon({
    width: 100,
    height: 100,
    mapSrc: 'path/to/marker.png',
    keepSize: true,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.4256, 39.9166, 0] },
    properties: {},
}]);
icon.dataSource = data;
```

#### 贴地图标（支持旋转）

```javascript
const icon = engine.add(new mapvthree.Icon({
    width: 100,
    height: 100,
    flat: true,
    mapSrc: 'path/to/marker.png',
    rotateZ: Math.PI,
    vertexRotateZs: true,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.4256, 39.9166, 0] },
    properties: { rotateZ: Math.PI / 2 },
}]);
data.defineAttribute('rotateZ');
icon.dataSource = data;
```

#### 多图标模式

```javascript
const icon = engine.add(new mapvthree.Icon({
    width: 100,
    height: 100,
    vertexIcons: true,
    flat: false,
    offset: [0, -30],
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.4252, 39.9162, 21] },
    properties: {
        icon: 'path/to/marker.png',
        color: 'red',
        size: 30,
    },
}]);
data.defineAttribute('color').defineAttribute('size').defineAttribute('icon');
icon.dataSource = data;
```

#### 跳动动画

```javascript
const icon = engine.add(new mapvthree.Icon({
    width: 100,
    height: 100,
    mapSrc: 'path/to/marker.png',
    animationJump: true,
    jumpHeight: 30,
    jumpSpeed: 1,
}));
```

#### 事件交互

```javascript
icon.receiveRaycast = true;

icon.addEventListener('click', e => {
    console.log('点击了图标', e);
});
```

### 代理属性

以下属性可在实例创建后动态修改：

- `width` / `height` - 图标尺寸
- `offset` - 偏移量
- `mapSrc` - 图标 URL
- `transparent` - 透明开关
- `opacity` - 透明度
- `flat` - 贴地模式
- `keepSize` - 保持像素大小
- `color` - 颜色
- `vertexColors` - 顶点颜色
- `animationJump` - 跳动动画
- `jumpHeight` - 跳动高度
- `jumpSpeed` - 跳动速度
- `fallback` - 备用图片

---

## Text 文字

> **注意：** Label 已覆盖 Text 的所有功能，且 Label 使用 SDF 渲染效果更好。推荐优先使用 Label。

Text 基于 Canvas 绘制文字纹理，适用于简单的纯文字标注场景。

### 构造参数

```javascript
const text = engine.add(new mapvthree.Text(parameters));
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `fontSize` | `number` | `16` | 字号大小 |
| `fontFamily` | `string` | `'Microsoft Yahei'` | 字体 |
| `fillStyle` | `string` | `'#ff0'` | 文字填充颜色 |
| `strokeStyle` | `string` | - | 文字描边颜色 |
| `lineWidth` | `number` | - | 文字描边宽度 |
| `padding` | `number[]` | `[2, 2]` | 文字内边距 `[x, y]` |
| `keepSize` | `boolean` | `false` | 是否保持像素大小（不随地图缩放变化） |
| `opacity` | `number` | `1` | 整体透明度 |
| `flat` | `boolean` | `false` | 是否贴地显示 |

### 属性

以下属性支持创建后动态修改：

| 属性 | 类型 | 说明 |
|------|------|------|
| `fontSize` | `number` | 字号大小 |
| `fontFamily` | `string` | 字体 |
| `fillStyle` | `string` | 文字填充颜色 |
| `strokeStyle` | `string` | 文字描边颜色 |
| `lineWidth` | `number` | 文字描边宽度 |
| `padding` | `number[]` | 文字内边距 |
| `keepSize` | `boolean` | 保持像素大小 |
| `opacity` | `number` | 透明度 |
| `flat` | `boolean` | 贴地显示 |
| `dataSource` | `DataSource` | 数据源 |

### 使用示例

```javascript
const text = engine.add(new mapvthree.Text({
    fontSize: 16,
    fontFamily: 'Arial',
    fillStyle: '#ff0000',
    strokeStyle: '#ffffff',
    lineWidth: 2,
    padding: [4, 4],
    keepSize: true,
}));

const data = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: { type: 'Point', coordinates: [116.404, 39.915] },
    properties: { text: '北京市' },
}]);
data.defineAttribute('text', 'text');
text.dataSource = data;
```

---

## 组件对比

| 特性 | Label | IconPoint | Icon | Text |
|------|-------|-----------|------|------|
| 继承 | GeoMesh | GeoPoints | GeoMesh | GeoMesh |
| 渲染方式 | Mesh（SDF） | GL_POINTS | Mesh（四边形） | Mesh（Canvas 纹理） |
| 文字支持 | 支持（SDF 渲染） | 不支持 | 不支持 | 支持（Canvas 渲染） |
| 图标支持 | 支持 | 支持 | 支持 | 不支持 |
| 图文组合 | 支持 | 不支持 | 不支持 | 不支持 |
| 贴地模式 | 支持 | 不支持 | 支持 | 支持 |
| 跳动动画 | 不支持 | 不支持 | 支持 | 不支持 |
| 碰撞检测 | 支持 | 支持 | 支持 | 支持 |
| 射线拾取 | 支持 | 不支持 | 支持（精确） | 支持 |
| 地球模式 | 支持 | 不支持 | 支持 | 支持 |
| 适用场景 | 大批量文本/图标标注 | 大量简单图标标记 | 需要动画/贴地的图标 | 简单纯文字标注（建议用 Label 替代） |

## 碰撞检测

Label、IconPoint、Icon 均支持碰撞检测，通过 `engine.rendering.collision.add(component)` 启用。详细用法见 [引擎初始化与地图绑定 - 碰撞检测](./engine-bindmap.md#碰撞检测)。

---

## 数据源说明

所有组件均使用 `GeoJSONDataSource` 作为数据源，通过 `defineAttribute` 方法建立 GeoJSON `properties` 到组件属性的映射：

```javascript
// 创建数据源
const data = mapvthree.GeoJSONDataSource.fromGeoJSON(geojsonData);

// 属性映射（属性名与 properties 键名相同时可省略第二个参数）
data.defineAttribute('icon');          // 等同于 data.defineAttribute('icon', 'icon');
data.defineAttribute('text', 'text');
data.defineAttribute('size');

// 支持函数映射
data.defineAttribute('text', p => {
    return `速度: ${p.speed} km/h`;
});
data.defineAttribute('color', p => {
    return p.size > 30 ? 'red' : 'green';
});

// 链式调用
data.defineAttribute('color').defineAttribute('size').defineAttribute('icon');

// 绑定到组件
component.dataSource = data;
```
