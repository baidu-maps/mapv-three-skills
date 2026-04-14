# 模型加载

mapv-three 提供了多种 3D 模型加载方式，包括简单模型（SimpleModel）、动画模型（AnimationModel）、LOD 模型（LODModel）和 GLTF 场景（GLTFScene）。对于大规模同类物体渲染，提供了 GeoInstancedMesh、InstancedModel 和 DynamicInstancedMesh 三种实例化方案。

## 快速开始

```javascript
// 假设 engine 已初始化（需开启 enableAnimationLoop）
const model = engine.add(new mapvthree.SimpleModel({
    name: '汽车',
    url: 'assets/models/car.glb',     // 支持 glb/gltf 格式
    point: [113.306, 23.487, 0],       // [经度, 纬度, 高度]
    scale: [10, 10, 10],               // 缩放
    rotation: [0, 0, 0],               // 旋转 [roll, pitch, heading]（弧度）
}));
```

---

## SimpleModel 简单模型

加载单个 3D 模型，支持 URL 加载 glb/gltf 文件或直接传入 Object3D 实例。自动处理坐标投影和 Y-up 到 Z-up 的轴向转换。

### 构造函数

```javascript
new mapvthree.SimpleModel(parameters)
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `parameters.name` | `string` | 否 | `''` | 模型名称 |
| `parameters.url` | `string` | 否 | - | 模型文件路径（glb/gltf 格式） |
| `parameters.object` | `Object3D` | 否 | - | 直接传入 three.js Object3D 实例（与 url 二选一） |
| `parameters.point` | `Array<number>\|Vector3` | 否 | `[0,0,0]` | 位置 `[经度, 纬度, 高度]` |
| `parameters.scale` | `Array<number>\|Vector3\|number` | 否 | `[1,1,1]` | 缩放比例 |
| `parameters.rotation` | `Array<number>\|Vector3` | 否 | `[0,0,0]` | 旋转 `[roll, pitch, heading]`，单位弧度 |
| `parameters.autoYUpToZUp` | `boolean` | 否 | `true` | 是否自动将 Y-up 转为 Z-up（仅 URL 加载） |

### 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `setTransform(transform)` | `{ point?, rotation?, scale? }` | 设置位置、旋转、缩放 |
| `point = value` | `Array\|Vector3` | 设置模型位置（setter） |

### 事件列表

| 事件名 | 触发时机 | 说明 |
|--------|----------|------|
| `loaded` | 模型加载完成 | `e.value` 为模型实例 |

### 常见场景

```javascript
import { Vector3, Mesh, BoxGeometry, MeshBasicMaterial } from 'three';

// 假设 engine 已初始化
const center = [113.306, 23.487];

// 方式一：通过 URL 加载 glb 模型
const model = engine.add(new mapvthree.SimpleModel({
    name: '汽车模型',
    url: 'assets/models/car.glb',
    point: center,
    scale: new Vector3(10, 10, 10),
}));

// 监听加载完成
model.addEventListener('loaded', e => {
    console.log('模型加载完成:', e.value);
});

// 方式二：传入 Object3D 实例
const box = new Mesh(new BoxGeometry(1, 1, 1), new MeshBasicMaterial({ color: 'red' }));
const model2 = engine.add(new mapvthree.SimpleModel({
    object: box,
    point: [center[0] + 0.001, center[1], 0],
}));

// 动态更新变换
model.setTransform({
    point: [113.307, 23.488, 100],
    rotation: [Math.PI / 2, 0, 0],
    scale: [5, 5, 5],
});
```

---

## AnimationModel 动画模型

继承自 SimpleModel，增加了骨骼动画播放和控制能力。适用于加载带动画的 glb/gltf 模型。

### 构造函数

```javascript
new mapvthree.AnimationModel(parameters)
```

继承 SimpleModel 的所有参数，额外支持：

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `parameters.autoPlay` | `boolean` | 否 | `false` | 是否自动播放第一个动画 |

### 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `play(actionIndex?)` | `actionIndex: number` | 播放指定动画，默认第 0 个 |
| `stop(actionIndex?)` | `actionIndex: number` | 停止指定动画，默认第 0 个 |
| `playAll()` | - | 播放所有动画 |
| `stopAll()` | - | 停止所有动画 |
| `setSpeed(speed, actionIndex?)` | `speed: number` | 设置播放速度，1 为正常。不传 actionIndex 则设置所有动画 |
| `setLoop(loop, actionIndex?)` | `loop: boolean` | 设置循环模式。不传 actionIndex 则设置所有动画 |

### 常见场景

```javascript
// 假设 engine 已初始化（需开启 enableAnimationLoop）
const center = [113.306, 23.487];

// 加载带动画的行人模型
const model = engine.add(new mapvthree.AnimationModel({
    name: '行人',
    url: 'assets/models/pedestrian_man_animate.glb',
    point: center,
    scale: new Vector3(10, 10, 10),
    // autoPlay: true, // 自动播放
}));

// 手动控制动画
model.play(0);          // 播放第一个动画
model.setSpeed(2);      // 2倍速
model.setLoop(true);    // 循环播放

// 停止
// model.stop(0);
// model.stopAll();
```

---

## LODModel 多细节层级模型

根据相机距离自动切换不同精度的模型，优化渲染性能。

### 构造函数

```javascript
new mapvthree.LODModel(parameters)
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `parameters.hysteresis` | `number` | 否 | `0.1` | 切换缓冲系数（0-1），防止在临界距离处抖动 |
| `parameters.levels` | `Array<Object>` | 否 | `[]` | 层级配置数组 |
| `parameters.levels[].distance` | `number` | 是 | - | 切换距离（世界坐标单位） |
| `parameters.levels[].file` | `string` | 是 | - | 模型文件路径 |
| `parameters.levels[].hysteresis` | `number` | 否 | - | 该层级的缓冲系数，覆盖全局设置 |

### 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `addLevel(file, distance, hysteresis?)` | `file: string`, `distance: number` | 添加层级，返回 this（可链式调用） |
| `removeLevel(file)` | `file: string` | 移除指定层级 |
| `getCurrentLevel()` | - | 获取当前层级索引 |
| `getCurrentModel()` | - | 获取当前显示的模型 |
| `getModel(level)` | `level: number` | 获取指定层级的模型（异步） |

### 属性列表

| 属性 | 类型 | 说明 |
|------|------|------|
| `transform` | `Object` | 读写 `{ translate, rotation, scale }` |
| `levels` | `Array` | 读写层级配置 |
| `hysteresis` | `number` | 读写缓冲系数 |

### 事件列表

| 事件名 | 触发时机 | 说明 |
|--------|----------|------|
| `loaded` | 单个层级模型加载完成 | `e.value` 为模型 |
| `complete` | 所有层级加载完成 | `e.value` 为 LODModel 实例 |

### 常见场景

```javascript
// 假设 engine 已初始化
const center = [113.306, 23.487];
const position = engine.map.projectArrayCoordinate(center);

// 方式一：构造时传入 levels
const lod = engine.add(new mapvthree.LODModel({
    hysteresis: 0.2,
    levels: [
        { distance: 500, file: 'assets/models/car_high.glb' },
        { distance: 1000, file: 'assets/models/car_low.glb' },
    ],
}));
lod.position.set(...position);
lod.scale.setScalar(10);
lod.rotateX(Math.PI / 2);

// 方式二：链式添加层级
const lod2 = engine.add(new mapvthree.LODModel({ hysteresis: 0.2 }));
lod2.position.set(...position);
lod2.scale.setScalar(10);
lod2.rotateX(Math.PI / 2);
lod2.addLevel('assets/models/car_high.glb', 500)
    .addLevel('assets/models/car_low.glb', 1000);

// 监听事件
lod.addEventListener('loaded', e => {
    console.log('层级模型加载完成:', e.value);
});
lod.addEventListener('complete', e => {
    console.log('所有层级加载完成');
});

// 绑定点击事件
lod.addEventListener('click', e => {
    if (e.object._currentModel) {
        console.log('点击了 LOD 模型');
    }
});
```

---

## GLTFScene GLTF 场景

加载包含多个模型的完整 GLTF 场景，根据 `scene_desc.json` 描述文件自动处理单体模型和实例化模型。适用于加载编辑器导出的复杂场景。

### 构造函数

```javascript
new mapvthree.GLTFScene(options)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `options.url` | `string` | 是 | 场景根目录 URL，目录下需包含 `scene_desc.json` |

### 常见场景

```javascript
// 假设 engine 已初始化

// 加载整个场景
const gltfScene = engine.add(new mapvthree.GLTFScene({
    url: 'http://example.com/scenes/my_scene/', // 场景目录，需含 scene_desc.json
}));
```

**场景描述文件格式** (`scene_desc.json`)：
```json
{
    "models": [
        {
            "name": "building",
            "path": "building.glb",
            "transform": {
                "translation": [0, 0, 0],
                "rotation": [0, 0, 0],
                "scale": [1, 1, 1]
            },
            "instances": [
                {
                    "translation": [0, 0, 0],
                    "rotation": [0, 0, 0],
                    "scale": [1, 1, 1]
                }
            ]
        }
    ]
}
```

- 当 `instances` 数组长度为 1 时，加载为普通静态模型
- 当 `instances` 数组长度大于 1 时，自动使用 InstancedMesh 批量渲染

---

## GeoInstancedMesh 地理实例化网格

将大量相同几何体和材质的物体通过实例化渲染，极大减少 draw call。使用 GeoJSON 数据源驱动，每个实例的位置由数据源中的坐标确定。

### 构造函数

```javascript
new mapvthree.GeoInstancedMesh(geometry, material)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `geometry` | `BufferGeometry` | 是 | 几何体实例 |
| `material` | `Material` | 是 | 材质实例 |

### 关键属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `dataSource` | `GeoJSONDataSource` | 数据源，设置后自动更新渲染 |
| `enableInstanceColor` | `boolean` | 是否启用实例颜色 |
| `getInstanceLocalMatrix` | `Function` | 自定义每个实例的本地变换矩阵 |

### 常见场景

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

// 假设 engine 已初始化
const center = [110.590, 19.956];

// 加载模型几何体
const loader = new GLTFLoader();
loader.load('assets/models/palm_tree.glb', gltf => {
    const tree = gltf.scene.children[0];

    // 创建实例化网格
    const instancedMesh = engine.add(
        new mapvthree.GeoInstancedMesh(tree.geometry, tree.material)
    );

    // 随机生成 10000 个点作为数据源
    const dataSource = mapvthree.GeoJSONDataSource.fromGeoJSON(
        randomPoints(center, 0.05, 10000) // 自定义的随机点生成函数
    );

    // 定义颜色属性
    dataSource.defineAttribute('color', p => Math.random() * 0xffffff);

    // 启用实例颜色
    instancedMesh.enableInstanceColor = true;
    instancedMesh.dataSource = dataSource;

    // 自定义每个实例的本地变换矩阵（可选）
    instancedMesh.getInstanceLocalMatrix = (coordinates, dataItem, index) => {
        const matrix = new THREE.Matrix4();
        const scale = dataItem.attributes.count;
        matrix.makeScale(scale, scale, scale);
        matrix.makeRotationZ(Math.PI * Math.random());
        return matrix;
    };

    // 绑定事件
    instancedMesh.addEventListener('click', e => {
        console.log('点击实例:', e.entity.index);
    });
    instancedMesh.addEventListener('mouseenter', e => {
        console.log('鼠标进入实例:', e.entity.index);
    });
    instancedMesh.addEventListener('mouseleave', e => {
        console.log('鼠标离开实例:', e.entity.index);
    });
});
```

---

## InstancedModel 静态批量模型

将多个 Mesh（如 GLTF 模型中的多个子网格）进行实例化渲染。通过 `setBufferData` 设置所有实例的位置、旋转和缩放。适用于静态批量物体。

### 构造函数

```javascript
new mapvthree.InstancedModel(meshes, count)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `meshes` | `Array<Mesh>` | 否 | Mesh 数组，不传则使用默认白色方块 |
| `count` | `number` | 是 | 预分配的实例数量 |

### 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `setBufferData(data)` | 见下表 | 设置所有实例的变换数据 |
| `has(id)` | `id: string` | 检查指定 ID 的实例是否存在 |
| `clear()` | - | 清空所有实例 |

### bufferData 格式

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | `Array<string>` | 每个实例的唯一 ID |
| `translation` | `Float32Array` | 位置数据，每 3 个元素为一组 `[x, y, z]` |
| `rotation` | `Float32Array` | 旋转数据，每 3 个元素为一组 |
| `scale` | `Float32Array` | 缩放数据，每 3 个元素为一组 |
| `instanceMatrix` | `Float32Array` | （可选）直接传入 4x4 矩阵数组，跳过 translation/rotation/scale 计算 |

### 属性列表

| 属性 | 类型 | 说明 |
|------|------|------|
| `needsUpdate` | `boolean`（setter） | 设为 true 触发更新渲染 |
| `count` | `number` | 实例数量 |
| `meshes` | `Array<Mesh>` | 读写基础 Mesh 数组 |

### 常见场景

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

// 假设 engine 已初始化
const position = engine.map.projectArrayCoordinate([0, 0]);
const count = 100;

// 加载模型并创建实例化
const loader = new GLTFLoader();
loader.load('assets/models/palm_tree.glb', gltf => {
    const meshes = gltf.scene.children; // 获取所有子 Mesh

    // 创建实例化模型
    const instancedModel = engine.add(new mapvthree.InstancedModel(meshes, count));

    // 准备 bufferData
    const ids = [];
    const translations = new Float32Array(count * 3);
    const scales = new Float32Array(count * 3);
    for (let i = 0; i < count; i++) {
        ids.push('tree_' + i);
        translations[i * 3] = position[0] + (Math.random() - 0.5) * 1000;
        translations[i * 3 + 1] = position[1] + (Math.random() - 0.5) * 1000;
        translations[i * 3 + 2] = 0;
        const s = 1 + Math.random();
        scales[i * 3] = s;
        scales[i * 3 + 1] = s;
        scales[i * 3 + 2] = s;
    }

    // 设置数据并触发更新
    instancedModel.setBufferData({
        id: ids,
        translation: translations,
        scale: scales,
    });
    instancedModel.needsUpdate = true;
});
```

---

## DynamicInstancedMesh 动态实例化网格

支持高效的动态增删实例，适用于需要频繁更新的场景（如实时车辆、动态植被等）。内部使用动态缓冲区自动扩容。

### 构造函数

```javascript
new mapvthree.DynamicInstancedMesh(meshes?)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `meshes` | `Array<Mesh>` | 否 | Mesh 数组，不传则使用默认白色方块 |

### 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `setBufferData(data)` | `bufferData` | 替换所有实例数据 |
| `addInstances(data)` | `bufferData` | 追加实例 |
| `removeInstance(id)` | `id: string` | 移除单个实例 |
| `removeInstances(data)` | `{ id: Array }` | 批量移除实例 |
| `setColor(id, color)` | `id, color: Color` | 设置单个实例颜色 |
| `has(id)` | `id: string` | 检查实例是否存在 |
| `clear()` | - | 清空所有实例 |

### 属性列表

| 属性 | 类型 | 说明 |
|------|------|------|
| `needsUpdate` | `boolean`（setter） | 设为 true 触发更新 |
| `meshes` | `Array<Mesh>` | 读写基础 Mesh |
| `enableColor` | `boolean` | 是否启用实例颜色 |
| `enableColorList` | `Array<string>` | 指定启用颜色的子 Mesh 名称列表 |
| `keepSize` | `boolean` | 是否根据相机距离保持视觉大小 |

### 常见场景

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

// 假设 engine 已初始化
const position = engine.map.projectArrayCoordinate([116, 40]);

const loader = new GLTFLoader();
loader.load('assets/models/tree.glb', gltf => {
    const meshes = gltf.scene.children;

    // 创建动态实例化
    const dynamicMesh = engine.add(new mapvthree.DynamicInstancedMesh(meshes));
    dynamicMesh.enableColor = true;

    // 初始数据
    const ids = ['a', 'b', 'c'];
    const translations = new Float32Array([
        position[0], position[1], 0,
        position[0] + 10, position[1], 0,
        position[0] + 20, position[1], 0,
    ]);
    dynamicMesh.setBufferData({
        id: ids,
        translation: translations,
        color: ['#ff0000', '#00ff00', '#0000ff'],
    });
    dynamicMesh.needsUpdate = true;

    // 动态添加
    dynamicMesh.addInstances({
        id: ['d'],
        translation: new Float32Array([position[0] + 30, position[1], 0]),
    });
    dynamicMesh.needsUpdate = true;

    // 动态移除
    dynamicMesh.removeInstance('b');
    dynamicMesh.needsUpdate = true;

    // 修改颜色
    dynamicMesh.setColor('a', new THREE.Color('#ffff00'));
    dynamicMesh.needsUpdate = true;
});
```

---

## 注意事项

1. **动画循环**：使用 AnimationModel 或 DynamicInstancedMesh 等需要逐帧更新的对象时，引擎初始化时需开启 `rendering.enableAnimationLoop: true`。

2. **坐标系统**：
   - SimpleModel 的 `point` 参数接受地理坐标 `[经度, 纬度, 高度]`，内部自动投影。
   - LODModel 和直接添加的 Object3D 需要手动调用 `engine.map.projectArrayCoordinate()` 获取投影坐标，再设置 `position`。
   - 实例化对象（InstancedModel、DynamicInstancedMesh）的 `translation` 使用投影后的坐标。

3. **Y-up 到 Z-up**：glb/gltf 模型默认为 Y-up 坐标系，mapv-three 使用 Z-up。SimpleModel 和 AnimationModel 会自动转换（`autoYUpToZUp` 默认 true）。手动加载模型时需添加 `model.rotateX(Math.PI / 2)`。

4. **实例化选择**：
   - **GeoInstancedMesh**：使用 GeoJSON 数据源驱动，适合地理分布的大量同类物体（如 POI 图标、树木），内置碰撞检测和事件支持。
   - **InstancedModel**：静态批量渲染，通过 bufferData 一次性设置所有实例，适合初始化后不再变化的场景。
   - **DynamicInstancedMesh**：支持动态增删，内置自动扩容，适合实时更新的场景（如车辆追踪）。

5. **性能优化**：LODModel 在超出最远距离时会自动隐藏模型释放渲染资源。使用 `hysteresis` 参数可防止在距离临界点频繁切换。

6. **模型缓存**：SimpleModel 和 AnimationModel 内部使用 `PromisedSingleton` 缓存已加载的模型，相同 URL 不会重复加载。

7. **资源清理**：通过 `engine.remove()` 移除模型时会自动调用 `dispose()` 清理资源。LODModel 的 `destroyModel()` 方法会遍历所有层级释放几何体、材质和纹理。
