# EffectModelPoint 效果模型点

特效模型点组件，支持加载自定义 3D 模型作为点类数据可视化元素。提供旋转动画和跳跃动画配置，可实现丰富的动态效果。

> 注意：动画效果需要在初始化引擎时设置 `rendering.enableAnimationLoop = true`。

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';

// 1. 创建引擎，启用动画循环
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        enableAnimationLoop: true,
    },
    map: {
        is3DControl: true,
    },
});
engine.map.setCenter([110.590777, 19.956398]);
engine.map.setZoom(17);
engine.map.setPitch(70);

// 2. 加载模型并创建 EffectModelPoint
const loader = new GLTFLoader();
loader.load('your_model.glb', function (gltf) {
    const effectModelPoint = engine.add(new mapvthree.EffectModelPoint({
        normalize: true,
        rotateToZUp: true,
        keepSize: true,
    }));
    effectModelPoint.model = gltf.scene;
    effectModelPoint.size = 30;

    // 3. 设置数据源
    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON([
        {
            geometry: {
                type: 'Point',
                coordinates: [110.590777, 19.956398],
            },
        },
    ]);
    effectModelPoint.dataSource = dataSource;
});
```

## 构造函数

```javascript
new mapvthree.EffectModelPoint(parameters)
```

### 参数 `parameters`

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `normalize` | `boolean` | `true` | 是否归一化模型大小。开启后模型会被缩放到统一的基础尺寸 |
| `rotateToZUp` | `boolean` | `true` | 将模型调整至 Z 轴朝上。用于兼容不同建模软件导出的坐标系 |
| `keepSize` | `boolean` | `true` | 是否保持固定大小。开启后模型大小不随地图缩放变化 |
| `animationRotate` | `boolean` | `false` | 是否启用旋转动画 |
| `animationRotatePeriod` | `number` | `3000` | 旋转一周的时间，单位毫秒 |
| `animationJump` | `boolean` | `false` | 是否启用跳跃动画 |
| `animationJumpPeriod` | `number` | `3000` | 跳跃一次的时间周期，单位毫秒 |
| `animationJumpHeight` | `number` | `30` | 跳跃高度 |
| `vertexColors` | `boolean` | `false` | 是否使用逐顶点颜色。开启后可通过数据源的 `color` 属性为每个点设置不同颜色 |
| `vertexSizes` | `boolean` | `false` | 是否使用逐顶点大小。开启后可通过数据源的 `size` 属性为每个点设置不同大小 |

## 属性

以下属性均支持通过 getter/setter 动态读写。修改后即时生效。

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `model` | `Object3D \| null` | `null` | 绑定的 3D 模型。可设置为 GLTF 加载后的 `gltf.scene`。不设置时使用内置默认钻石模型 |
| `size` | `number` | `1` | 模型统一缩放大小 |
| `size3` | `Array<number>` | `[1, 1, 1]` | 模型三轴独立缩放，格式为 `[x, y, z]`。需配合 `useSize3 = true` 使用 |
| `useSize3` | `boolean` | `false` | 是否使用三轴独立缩放（`size3`）替代统一缩放（`size`） |
| `height` | `number` | `0` | 模型距地面的高度偏移 |
| `keepSize` | `boolean` | `true` | 是否保持固定大小，不随地图缩放变化 |
| `animationRotate` | `boolean` | `false` | 是否启用旋转动画 |
| `animationRotatePeriod` | `number` | `3000` | 旋转一周的时间（毫秒） |
| `animationJump` | `boolean` | `false` | 是否启用跳跃动画 |
| `animationJumpPeriod` | `number` | `3000` | 跳跃一次的时间（毫秒） |
| `animationJumpHeight` | `number` | `30` | 跳跃高度 |
| `dataSource` | `DataSource` | - | 数据源，用于提供点位坐标数据 |
| `visible` | `boolean` | `true` | 是否可见（继承自 Three.js Object3D） |
| `name` | `string` | `''` | 对象名称（继承自 Three.js Object3D） |
| `receiveRaycast` | `boolean` | `false` | 是否接收射线检测，用于事件交互。需手动设置为 `true` |
| `isEventEntitySupported` | `boolean` | `true` | 是否支持事件实体（只读） |

## 方法

| 方法名 | 说明 |
|--------|------|
| `dispose()` | 销毁组件，释放模型、几何体和材质资源。调用 `engine.remove(object)` 时会自动执行，通常无需手动调用 |
| `addAttributeRename(key, value)` | 添加属性重命名映射。`key` 为组件属性名，`value` 为数据源属性名 |
| `removeAttributeRename(key)` | 移除指定的属性重命名映射 |
| `clearAttributeRename()` | 清空所有属性重命名映射 |

## 常见场景

### 场景一：带旋转和跳跃动画的模型点

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        enableAnimationLoop: true,
    },
    map: {
        is3DControl: true,
    },
});
engine.map.setCenter([110.590777, 19.956398]);
engine.map.setZoom(17);
engine.map.setPitch(70);

const loader = new GLTFLoader();
loader.load('your_model.glb', function (gltf) {
    const effectModelPoint = engine.add(new mapvthree.EffectModelPoint({
        normalize: true,
        rotateToZUp: true,
        keepSize: true,
        animationJump: true,
        animationRotate: true,
        animationRotatePeriod: 2000,
        animationJumpPeriod: 2000,
        animationJumpHeight: 6,
    }));
    effectModelPoint.model = gltf.scene;
    effectModelPoint.size = 30;

    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON([
        {geometry: {type: 'Point', coordinates: [110.590777, 19.956398]}},
        {geometry: {type: 'Point', coordinates: [110.595, 19.960]}},
    ]);
    effectModelPoint.dataSource = dataSource;
});
```

### 场景二：大量点位 + 事件交互 + Bloom 辉光效果

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';

const center = [110.590777, 19.956398];
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        enableAnimationLoop: true,
    },
    map: {
        is3DControl: true,
    },
});
engine.map.setCenter(center);
engine.map.setZoom(17);
engine.map.setPitch(70);

// 开启 Bloom 辉光
engine.rendering.bloom.enabled = true;

const loader = new GLTFLoader();
loader.load('your_model.glb', function (gltf) {
    const effectModelPoint = engine.add(new mapvthree.EffectModelPoint({
        normalize: true,
        rotateToZUp: true,
        keepSize: true,
    }));
    effectModelPoint.receiveRaycast = true;
    effectModelPoint.model = gltf.scene;
    effectModelPoint.size = 30;

    // 生成随机点数据
    const data = mapvthree.GeoJSONDataSource.fromGeoJSON(
        randomPoints(center, 0.05, 1000)
    );
    // 为每个点定义独立大小
    data.defineAttribute('size', p => Math.random() * 10 + 10);
    effectModelPoint.dataSource = data;

    // 绑定点击事件
    effectModelPoint.addEventListener('click', e => {
        console.log('点击了点:', e);
    });

    // 绑定鼠标进入/离开事件
    effectModelPoint.addEventListener('mouseenter', e => {
        console.log('鼠标进入, 索引:', e.entity.index);
    });
    effectModelPoint.addEventListener('mouseleave', e => {
        console.log('鼠标离开, 索引:', e.entity.index);
    });
});
```

### 场景三：在 PointGroup 中组合使用

EffectModelPoint 可以作为 `PointGroup` 的子组件与其他点类组件（如 Text、Icon、SimplePoint）组合使用，共享同一数据源。

```javascript
import * as mapvthree from '@baidu/mapv-three';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';

const center = [116.414, 39.915];
const engine = new mapvthree.Engine(document.getElementById('map_container'));
engine.map.setCenter(center);
engine.map.setZoom(16);
engine.map.setPitch(50);

const pointGroup = engine.add(
    new mapvthree.PointGroup({vertexSizes: true, vertexIcons: true})
);

// 添加文本组件
pointGroup.addComponent(new mapvthree.Text({}));

// 添加图标组件
pointGroup.addComponent(new mapvthree.Icon({
    mapSrc: 'https://example.com/icon.png',
    width: 50,
    height: 100,
}));

// 添加模型点组件
const loader = new GLTFLoader();
loader.load('your_model.glb', function (gltf) {
    const effectModelPoint = pointGroup.addComponent(
        new mapvthree.EffectModelPoint({
            normalize: true,
            rotateToZUp: true,
            keepSize: true,
            size: 30,
        })
    );
    effectModelPoint.model = gltf.scene;
});

// 设置共享数据源
const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(points);
dataSource
    .defineAttribute('text', p => '标注文字')
    .defineAttribute('size');
pointGroup.dataSource = dataSource;
```

### 场景四：使用默认模型（不加载自定义 GLB）

不设置 `model` 属性时，组件会自动加载内置的钻石模型。

```javascript
const effectModelPoint = engine.add(new mapvthree.EffectModelPoint({
    normalize: true,
    rotateToZUp: true,
    keepSize: true,
    animationRotate: true,
    animationJump: true,
}));
effectModelPoint.size = 30;

const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON([
    {geometry: {type: 'Point', coordinates: [116.414, 39.915]}},
]);
effectModelPoint.dataSource = dataSource;
```

## 注意事项

1. **必须启用动画循环**：使用旋转或跳跃动画时，必须在引擎初始化时设置 `rendering.enableAnimationLoop = true`，否则动画不会播放。

2. **模型加载是异步的**：通过 GLTFLoader 加载模型是异步操作，需要在回调或 Promise 中设置 `model` 属性。内置默认模型也是异步加载的。

3. **添加到引擎的时机**：应先通过 `engine.add()` 将 EffectModelPoint 添加到引擎，再设置 `model` 和 `dataSource`。

4. **事件交互**：要启用点击、鼠标悬停等事件，需手动设置 `receiveRaycast = true`。支持的事件类型包括 `click`、`mousemove`、`mouseenter`、`mouseleave`。

5. **`size` 与 `size3` 的关系**：默认使用 `size` 进行统一缩放。若需要三轴独立缩放，需同时设置 `useSize3 = true` 和 `size3 = [x, y, z]`。

6. **`keepSize` 的作用**：设置为 `true` 时模型在屏幕上保持固定像素大小，不随地图缩放而变化；设置为 `false` 时模型大小会随地图缩放等比变化。

7. **逐顶点属性**：启用 `vertexColors` 或 `vertexSizes` 后，可通过数据源的 `defineAttribute` 为每个点定义独立的颜色或大小值。

8. **资源释放**：调用 `engine.remove(effectModelPoint)` 时会自动释放几何体和材质资源，无需手动调用 `dispose()`。

9. **Bloom 辉光效果**：如需辉光效果，需开启 `engine.rendering.useMrt = true` 和 `engine.rendering.bloom.enabled = true`。