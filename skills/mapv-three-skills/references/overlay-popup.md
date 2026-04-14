# 覆盖物与弹窗

MapV-Three 提供了一套完整的覆盖物系统，包括 DOM 覆盖物（DOMOverlay）、信息弹窗（Popup）、图片标注（Marker）和地面覆盖物（GroundOverlay）。覆盖物可以在地图上的指定位置渲染 HTML 元素、图片或纹理，并支持丰富的事件交互。

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 初始化引擎
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        enableAnimationLoop: true,
    },
});
engine.map.setCenter([116.404, 39.915]);
engine.map.setZoom(14);

// 添加一个 Marker 标注
const marker = engine.add(new mapvthree.Marker({
    point: [116.404, 39.915],
    icon: 'path/to/icon.png',
    width: 32,
    height: 32,
}));

// 添加一个 Popup 信息弹窗
const popup = engine.add(new mapvthree.Popup({
    point: [116.404, 39.915, 100],
    title: '位置信息',
    content: '北京市天安门广场',
}));

// 添加一个自定义 DOM 覆盖物
const node = document.createElement('div');
node.innerHTML = '<span style="color:#fff;background:rgba(0,0,0,0.7);padding:4px 8px;">自定义DOM</span>';
const overlay = engine.add(new mapvthree.DOMOverlay({
    point: [116.414, 39.915],
    dom: node,
    enableDragging: true,
}));
```

---

## DOMOverlay - DOM 覆盖物

DOM 覆盖物基类，用于在地图上添加任意 HTML 元素。继承自 `THREE.Object3D`。

### 构造函数

```javascript
new mapvthree.DOMOverlay(parameters)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `parameters.dom` | `HTMLElement \| string` | **必填** | DOM 元素或 HTML 字符串 |
| `parameters.point` | `Array<number>` | `[]` | 位置坐标 `[经度, 纬度, 高度]` |
| `parameters.offset` | `Array<number>` | `[0, 0]` | 像素偏移量 `[x, y]` |
| `parameters.visible` | `boolean` | `true` | 是否可见 |
| `parameters.enableDragging` | `boolean` | `false` | 是否启用拖拽 |
| `parameters.className` | `string` | `''` | CSS 类名 |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `dom` | `HTMLElement` | 获取/设置 DOM 元素，支持 HTMLElement 或 HTML 字符串 |
| `point` | `Array<number>` | 获取/设置坐标位置 `[lng, lat, alt]`，也支持 `THREE.Vector2/Vector3` |
| `visible` | `boolean` | 获取/设置是否可见 |
| `offset` | `Array<number>` | 获取/设置像素偏移量 `[x, y]` |
| `enableDragging` | `boolean` | 获取/设置是否启用拖拽 |
| `stopPropagation` | `boolean` | 获取/设置是否阻止事件冒泡 |
| `className` | `string` | 获取/设置 CSS 类名 |

### 方法

| 方法 | 说明 |
| --- | --- |
| `dispose()` | 释放资源，移除 DOM 元素和事件监听 |
| `addEventListener(type, listener)` | 继承自 Object3D，监听事件（如 `dragstart`、`dragmove`、`dragend`） |
| `removeEventListener(type, listener)` | 移除事件监听 |

### 拖拽事件

启用 `enableDragging` 后，DOMOverlay 会派发以下事件：

| 事件名 | 触发时机 | 回调参数 |
| --- | --- | --- |
| `dragstart` | 开始拖拽时 | `{ type, target, point }` |
| `dragmove` | 拖拽移动时 | `{ type, target, point }` |
| `dragend` | 拖拽结束时 | `{ type, target, point }` |

### 完整示例

```javascript
// 创建自定义样式的 DOM 覆盖物
const node = document.createElement('div');
node.style.width = '200px';
node.style.height = '100px';
node.style.background = 'rgba(100, 48, 44, 0.8)';
node.style.borderTop = '4px solid rgba(162, 52, 50, 0.8)';
node.style.color = '#fff';
node.style.fontSize = '12px';
node.style.padding = '10px';
node.innerText = '您好，我是自定义DOM！';

const overlay = engine.add(new mapvthree.DOMOverlay({
    point: [116.44, 39.915],
    dom: node,
}));

// 启用拖拽
overlay.enableDragging = true;

// 监听拖拽事件
overlay.addEventListener('dragstart', e => {
    console.log('拖拽开始', e.point);
});
overlay.addEventListener('dragmove', e => {
    console.log('拖拽中', e.point);
});
overlay.addEventListener('dragend', e => {
    console.log('拖拽结束', e.point);
});
```

也可以使用 HTML 字符串创建：

```javascript
const htmlOverlay = engine.add(new mapvthree.DOMOverlay({
    point: [116.404, 39.915],
    dom: `
        <div class="dom-overlay-custom">
            <div class="overlay-body">
                <div class="overlay-title">Hello World</div>
            </div>
        </div>
    `,
}));
```

---

## Popup - 信息弹窗

信息弹窗覆盖物，自带标题、内容和关闭按钮。继承自 `DOMOverlay`，锚点位于底部中心。

### 构造函数

```javascript
new mapvthree.Popup(parameters)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `parameters.point` | `Array<number>` | `[]` | 位置坐标 `[经度, 纬度, 高度]` |
| `parameters.title` | `string \| HTMLElement` | `'title'` | 弹窗标题，支持 HTML 字符串或 HTMLElement |
| `parameters.content` | `string \| HTMLElement` | `'content'` | 弹窗内容，支持 HTML 字符串或 HTMLElement |
| `parameters.offset` | `Array<number>` | `[0, 0]` | 像素偏移量 |
| `parameters.visible` | `boolean` | `true` | 是否可见 |
| `parameters.className` | `string` | `''` | 自定义 CSS 类名 |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `title` | `string \| HTMLElement` | 获取/设置标题 |
| `content` | `string \| HTMLElement` | 获取/设置内容 |

继承自 DOMOverlay 的所有属性（`point`、`visible`、`offset`、`className` 等）。

### 事件

| 事件名 | 触发时机 | 说明 |
| --- | --- | --- |
| `close` | 点击关闭按钮时 | `{ type: 'close', target }` |

### 完整示例

```javascript
// 创建弹窗
const popup = engine.add(new mapvthree.Popup({
    point: [116.404, 39.915, 100],
    title: '位置信息',
    content: '<p>北京市天安门广场</p><p>坐标: 116.404, 39.915</p>',
    offset: [0, -20], // 向上偏移 20 像素
    className: 'my-popup',
}));

// 阻止事件冒泡（避免点击弹窗触发地图事件）
popup.stopPropagation = true;

// 动态更新内容
popup.title = '新标题';
popup.content = '新内容';

// 监听关闭事件
popup.addEventListener('close', e => {
    console.log('弹窗已关闭');
});

// 使用 HTMLElement 作为内容
const customContent = document.createElement('div');
customContent.innerHTML = '<strong>自定义HTML内容</strong>';
popup.content = customContent;
```

---

## Marker - 图片标注

图片标注覆盖物，用于在地图上添加图标。继承自 `DOMOverlay`，锚点位于图片中心。

### 构造函数

```javascript
new mapvthree.Marker(parameters)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `parameters.point` | `Array<number>` | `[0, 0, 0]` | 位置坐标 `[经度, 纬度, 高度]` |
| `parameters.icon` | `string` | 默认红色标注图片 | 图标 URL |
| `parameters.width` | `number` | `25` | 图标宽度（像素） |
| `parameters.height` | `number` | `25` | 图标高度（像素） |
| `parameters.offset` | `Array<number>` | `[0, 0]` | 像素偏移量 |
| `parameters.enableDragging` | `boolean` | `false` | 是否可拖拽 |
| `parameters.title` | `string` | `''` | 鼠标悬停提示文字 |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `icon` | `string` | 获取/设置图片路径 |
| `width` | `number` | 获取/设置图片宽度 |
| `height` | `number` | 获取/设置图片高度 |
| `title` | `string` | 获取/设置鼠标悬停提示文字 |

继承自 DOMOverlay 的所有属性和拖拽事件。

### 完整示例

```javascript
// 创建标注
const marker = engine.add(new mapvthree.Marker({
    point: [116.404, 39.915],
    icon: 'path/to/icon.png',
    width: 32,
    height: 32,
    offset: [0, -16], // 锚点设置到图片底部中心
    enableDragging: true,
}));

// 动态修改图标
marker.icon = 'path/to/new-icon.png';
marker.width = 48;
marker.height = 48;

// 监听拖拽
marker.addEventListener('dragend', e => {
    console.log('标注拖拽到:', e.point);
});
```

---

## GroundOverlay - 地面覆盖物

在地图表面指定地理范围内渲染图片、Canvas 或视频。继承自 `THREE.Mesh`，球面模式下自动进行自适应三角细分以贴合椭球面。

### 构造函数

```javascript
new mapvthree.GroundOverlay(parameters)
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `parameters.bounds` | `Array<Array<number>>` | `null` | 地理范围 `[[minLng, minLat], [maxLng, maxLat]]` |
| `parameters.url` | `string \| HTMLCanvasElement` | `''` | 图片URL、Canvas元素或视频URL |
| `parameters.type` | `string` | `'image'` | 类型：`'image'`、`'canvas'`、`'video'` |
| `parameters.opacity` | `number` | `1` | 透明度，范围 0-1 |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `bounds` | `Array<Array<number>>` | 获取/设置地理范围 |
| `url` | `string \| HTMLCanvasElement` | 获取/设置资源 URL |
| `opacity` | `number` | 获取/设置透明度 |

### 完整示例

```javascript
// 图片地面覆盖物
const groundOverlay = engine.add(new mapvthree.GroundOverlay({
    bounds: [[117.19635, 36.24093], [117.20350, 36.24764]],
    url: '/path/to/satellite-image.png',
    type: 'image',
    opacity: 0.8,
}));

// 动态修改透明度
groundOverlay.opacity = 0.5;

// 动态修改范围
groundOverlay.bounds = [[117.200, 36.245], [117.210, 36.255]];

// Canvas 覆盖物
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
canvas.width = 256;
canvas.height = 256;
ctx.fillStyle = 'rgba(255, 0, 0, 0.5)';
ctx.fillRect(0, 0, 256, 256);

const canvasOverlay = engine.add(new mapvthree.GroundOverlay({
    bounds: [[117.19, 36.24], [117.20, 36.25]],
    url: canvas,
    type: 'canvas',
}));

// 视频覆盖物
const videoOverlay = engine.add(new mapvthree.GroundOverlay({
    bounds: [[117.19, 36.24], [117.20, 36.25]],
    url: '/path/to/video.mp4',
    type: 'video',
    opacity: 1,
}));
```

---

## 事件系统

MapV-Three 提供了统一的事件系统，通过 `object.addEventListener` / `object.removeEventListener` 对物体绑定事件。

### 支持的事件类型

| 事件名 | 说明 |
| --- | --- |
| `click` | 鼠标单击 |
| `dblclick` | 鼠标双击 |
| `rightclick` | 鼠标右键单击 |
| `rightdblclick` | 鼠标右键双击 |
| `mousemove` | 鼠标移动 |
| `mouseenter` | 鼠标移入物体 |
| `mouseleave` | 鼠标移出物体 |
| `pointerdown` | 指针按下 |
| `pointerup` | 指针抬起 |

### 事件回调参数

所有事件（click、dblclick、mousemove 等）的回调参数 `e` 是 `EventParams` 对象，包含以下字段：

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `e.point` | `Array<number>` | 点击位置的**经纬度坐标** `[lng, lat, alt]`，由 `position` 自动反投影得到 |
| `e.position` | `Array<number>` | 点击位置的**投影坐标** `[x, y, z]`，从射线交点 `intersection.point` 获取 |
| `e.pixel` | `Array<number>` | 屏幕像素坐标 `[x, y]` |
| `e.event` | `Event` | 原始浏览器 DOM 事件 |
| `e.target` | `Object3D` | 射线命中的最底层物体（实际被点击的子网格） |
| `e.currentTarget` | `Object3D` | 绑定事件监听器的物体（事件冒泡的当前层） |
| `e.entity` | `Object` | 数据层实体信息（如 SimplePoint 中被点击的数据项，包含 `index`、`value` 等） |
| `e.intersection` | `Object` | Three.js 射线交点信息，包含 `point`（Vector3）、`distance`、`face` 等 |
| `e.stopPropagation()` | `Function` | 阻止事件继续冒泡 |

**常用取值方式：**

```javascript
engine.map.addEventListener('click', e => {
    // 经纬度坐标（最常用）
    console.log('经纬度:', e.point);  // [116.404, 39.915, 0]

    // 投影坐标
    console.log('投影坐标:', e.position);  // [12958056, 4853597, 0]

    // 屏幕像素坐标
    console.log('像素:', e.pixel);  // [512, 384]

    // 被点击的物体
    console.log('目标:', e.target);

    // 数据层实体（如点击了 SimplePoint 中的某个点）
    if (e.entity) {
        console.log('数据索引:', e.entity.index);
        console.log('数据值:', e.entity.value);
    }
});
```

### 事件绑定示例

```javascript
// 给地图绑定点击事件
engine.map.addEventListener('click', e => {
    console.log('点击位置:', e.point);
});

// 给 3D 物体绑定点击事件
const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
engine.add(mesh);
mesh.addEventListener('click', e => {
    console.log('点击了物体:', e.target);
});

// 鼠标移入/移出
mesh.addEventListener('mouseenter', e => {
    mesh.material.color.set(0xff0000);
    engine.requestRender();
});
mesh.addEventListener('mouseleave', e => {
    mesh.material.color.set(0x00ff00);
    engine.requestRender();
});

```

### 数据层事件

```javascript
// 给点图层绑定点击事件
const pointLayer = engine.add(new mapvthree.SimplePoint({ size: 20 }));
pointLayer.dataSource = dataSource;

pointLayer.addEventListener('click', e => {
    console.log('点击的数据项:', e.entity);
    // e.entity.value 包含数据项的属性
    const id = e.entity.value.id;
    // 动态修改被点击数据项的颜色
    dataSource.setAttributeValues(id, {
        color: 0xff0000,
    });
});

pointLayer.addEventListener('rightclick', e => {
    // 右键删除数据项
    dataSource.remove(e.entity.value.id);
});
```

---

## 注意事项

1. **覆盖物锚点差异**：DOMOverlay 和 Marker 的锚点在元素中心，Popup 的锚点在底部中心。设置 `offset` 时需注意这一区别。

2. **拖拽与事件冒泡**：启用 `enableDragging` 后，覆盖物的滚轮事件会自动转发到地图 canvas 上，避免覆盖物遮挡地图缩放。如需阻止其他事件冒泡，设置 `stopPropagation = true`。

3. **HTML 字符串安全**：通过 `dom` 属性传入 HTML 字符串时，内部使用 `DOMParser` 解析。请确保传入的 HTML 内容安全可信。

4. **GroundOverlay 球面模式**：在 ECEF（球面）投影模式下，GroundOverlay 会自动进行三角细分以贴合椭球面，大范围覆盖物可能产生较多三角面。

5. **视频覆盖物**：GroundOverlay 的 `video` 类型会自动循环播放并静音，内部使用 `requestAnimationFrame` 驱动纹理更新。切换 `url` 或移除时会自动停止视频。

6. **事件系统**：使用 `object.addEventListener` 绑定事件。事件会沿场景图向上冒泡，可通过 `e.stopPropagation()` 阻止。

7. **内存释放**：覆盖物从引擎移除时会自动调用 `dispose()` 释放 DOM 和事件监听。如手动创建覆盖物后不再使用，建议调用 `engine.remove(overlay)` 确保资源释放。
