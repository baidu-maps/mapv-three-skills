# 3DTiles 加载与管理

MapV-Three 提供了完整的 3DTiles 加载和管理体系，支持从 URL 或 Cesium Ion 加载 3D Tiles 数据集，并提供材质管理器、元素管理器、实例化元素、分组管理等高级功能。

> **重要规则（必须遵守）**：
> 1. 使用 `HDMap3DTiles` 时，**必须**在 `engine.add()` 之后立即配置 `tiles.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2()`，除非用户明确指定了其他材质管理器。
> 2. 树木、路灯、信号灯、POI 标注等**不是**独立的 3DTiles 数据集，而是 `HDMap3DTiles` 的**实例化元素图层**，通过 `instancedElementManager.setInstanceCollectionConfig('tree', {enabled: true})` 来开启。

---

## 交互判断

**关键规则**：当用户提到"树"、"路灯"、"信号灯"、"POI"等元素时，这些是 HDMap3DTiles 的实例化元素图层，必须使用 `HDMap3DTiles` 加载，然后通过 `instancedElementManager` 开启对应图层。即使用户给了一个 tileset.json 的 URL，也应该用 `HDMap3DTiles` 而非 `Default3DTiles` 来加载���

用户要求加载 3DTiles 时，如果用户已明确指定了类或提到了树木/路灯等元素���键词，直接使用对应类，不再询问。否则通过交互确认让用户选择以下方案（推荐选项排在前面并标注「推荐」）：
- `普通 3DTiles（建筑/地形等） — 使用 Default3DTiles（推荐）`
- `高精地图场景（含树木/路灯/信号灯/POI） — 使用 HDMap3DTiles`

**示例**：用户说"帮我加载树的3dtiles，url 是 xxx/tileset.json"，正确做法：
```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'xxx/tileset.json',
}));
// HDMap3DTiles 默认配置 V2 材质管理器
tiles.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2();
// 开启树木图层
tiles.instancedElementManager.setInstanceCollectionConfig('tree', {enabled: true});
```
错误做法：用 `Default3DTiles` 加载（树木图层和高精地图材质不会生效）。

**注意**：使用 `HDMap3DTiles` 时应默认配置 `HDMap3DTilesMaterialManagerV2` 材质管理器，除非用户指定了其他材质管理器。

若用户选择或已指定高精地图，且未明确说明需要哪些附加元素，可询问用户需要哪些。HDMap3DTiles 支持以下附加元素：

**1. 实例化元素（通过 `instancedElementManager.setInstanceCollectionConfig`）**

HDInstanceConfig.json 中默认已配置 26 种元素（护栏、桥墩、坡面、交通锥、摄像头等），默认全部启用。常用的额外控制：
- 树木：`tiles.instancedElementManager.setInstanceCollectionConfig('tree', {enabled: true})`（tree 默认未启用，需手动开启）
- 自定义树木模型：`tiles.instancedElementManager.treeModelPath = 'path/to/tree.glb'`
- 关闭某类元素：`tiles.instancedElementManager.setInstanceCollectionConfig('camera', {enabled: false})`

**2. POI 标注（通过 `instancedElementManager.labelEnabled`）**
- 开启：`tiles.instancedElementManager.labelEnabled = true`


---

## 场景选择

| 场景 | 加载类 | materialManager | 说明 |
|------|--------|-----------------|------|
| 高精地图 | `HDMap3DTiles` | `HDMap3DTilesMaterialManagerV2` | 内置水渲染、实例化元素，推荐默认使用 |
| 地形 | `HDMap3DTiles` | `Terrain3DTilesMaterialManager` | 地形也用 HDMap3DTiles，内置地形材质 |
| 建筑 | `Default3DTiles` 或 `HDMap3DTiles` | `Building3DTilesMaterialManager` | 建筑专用材质，两种加载类均可 |
| 独立水系 | `Default3DTiles` | 自定义 MaterialManager（含 WaterMaterial） | 水面作为独立 3DTiles 层 |
| 多区域分组 | `Default3DTilesGroup` | - | 统一管理多个数据集 |

### 水面渲染

`HDMap3DTilesMaterialManagerV2` 已内置水面材质，加载高精地图时自动渲染水面，无需额外配置：

```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: dataUrl + 'hdmap/tileset.json',
}));
tiles.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2();
```

用户需要渲染水面时，推荐使用 `HDMap3DTilesMaterialManagerV2`，它内置了水面材质、深度计算、菲涅尔反射等效果。

---

## 快速开始

### 基础加载

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 创建引擎
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    map: {
        is3DControl: true,
    },
    rendering: {
        sky: null,
        features: {
            antialias: { enabled: true },
            shadow: { enabled: true },
        },
    },
});

engine.map.setCenter([105.920169, 29.356349]);
engine.map.setZoom(17);
engine.map.setPitch(80);

// 添加天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 3600 * 16.5;

// 加载3DTiles数据
const tiles = engine.add(new mapvthree.Default3DTiles({
    url: 'path/to/tileset.json',
    errorTarget: 16,
}));

// 设置阴影
tiles.receiveShadow = true;
tiles.castShadow = true;
```

### 从 Cesium Ion 加载

```javascript
// 通过 assetId 从 Cesium Ion 加载
const tiles = await mapvthree.Default3DTiles.fromAssetId('your-asset-id', {
    errorTarget: 16,
});
engine.add(tiles);
```

---

## Default3DTiles

默认的 3DTiles 加载器，继承自 `Cesium3DTileset`，是所有 3DTiles 类型的基类。

### 构造函数

```javascript
new mapvthree.Default3DTiles(options)
```

### 参数

| 参数名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| `url` | `string` | - | 3DTiles 的 tileset.json URL 地址 |
| `assetId` | `number` | - | Cesium Ion asset ID（与 url 二选一） |
| `errorTarget` | `number` | `64` | 屏幕空间误差目标值（越小精度越高，加载量越大） |
| `forceUnlit` | `boolean` | `false` | 是否强制使用无光照模式 |
| `cullWithChildrenBounds` | `boolean` | `true` | 是否使用子节点边界进行剔除 |
| `cullRequestsWhileMoving` | `boolean` | `true` | 移动时是否剔除请求 |
| `cullRequestsWhileMovingMultiplier` | `number` | `60` | 移动时剔除请求的乘数 |
| `dynamicScreenSpaceError` | `boolean` | `true` | 是否使用动态屏幕空间误差 |
| `dynamicScreenSpaceErrorHeightFalloff` | `number` | `0.25` | 动态屏幕空间误差高度衰减 |
| `dynamicScreenSpaceErrorDensity` | `number` | `0.00278` | 动态屏幕空间误差密度 |
| `foveatedScreenSpaceError` | `boolean` | `true` | 是否使用注视点屏幕空间误差 |
| `foveatedConeSize` | `number` | `0.05` | 注视点锥体大小 |
| `foveatedMinimumScreenSpaceErrorRelaxation` | `number` | `0.0` | 注视点最小屏幕空间误差松弛值 |
| `progressiveResolutionHeightFraction` | `number` | `0.5` | 渐进式分辨率高度分数 |
| `cacheBytes` | `number` | - | 缓存字节数 |
| `loaders` | `Array` | - | 自定义加载器配置，如 `[[/\.gltf$/, gltfLoader]]` |

### 属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| `errorTarget` | `number` | 屏幕空间误差目标值（可读写） |
| `showDebug` | `boolean` | 是否显示调试边界框（可读写） |
| `castShadow` | `boolean` | 是否投射阴影（可读写） |
| `receiveShadow` | `boolean` | 是否接收阴影（可读写） |
| `raycastMethod` | `number` | 射线检测方法（可读写） |
| `freezeUpdate` | `boolean` | 是否冻结更新（可读写） |
| `materialManager` | `Default3DTilesMaterialManager` | 材质管理器（可读写） |
| `elementsManager` | `ElementsManager` | 元素管理器（只读） |
| `instancedElementManager` | `TileInstancedElementManager` | 实例化元素管理器（只读） |
| `editableElementManager` | `EditableElementManager` | 可编辑元素管理器（只读） |
| `cullWithChildrenBounds` | `boolean` | 是否使用子节点边界剔除（可读写） |
| `cullRequestsWhileMoving` | `boolean` | 移动时是否剔除请求（可读写） |
| `foveatedConeSize` | `number` | 注视点锥体大小（可读写） |
| `loadSiblings` | `boolean` | 是否加载兄弟节点（可读写） |
| `enabledSchedule` | `boolean` | 是否启用调度（可读写） |
| `deferOutSideFrustum` | `boolean` | 是否延迟加载视锥外瓦片（可读写） |

### 射线检测常量

| 常量 | 值 | 说明 |
|------|---|------|
| `Default3DTiles.RAYCAST_NONE` | `0` | 禁用射线检测 |
| `Default3DTiles.RAYCAST_DEFAULT` | `1` | 默认射线检测 |
| `Default3DTiles.RAYCAST_BVH` | `2` | 使用 BVH 加速射线检测 |

### 方法

| 方法名 | 参数 | 返回值 | 说明 |
|-------|------|--------|------|
| `Default3DTiles.fromAssetId(assetId, options)` | `assetId: number, options: Object` | `Promise<Default3DTiles>` | 静态方法，从 Cesium Ion 加载 |
| `lockCameraViewport()` | - | - | 锁定当前相机视角（调试用） |
| `releaseCameraViewport()` | - | - | 释放相机视角锁定 |
| `getBounds()` | - | `Box3` | 获取 3DTiles 的包围盒 |
| `transformFromEcefToPlane(lng, lat, height)` | 经度、纬度、高程 | - | ECEF 坐标转平面坐标 |

### 基础示例

```javascript
const tiles = engine.add(new mapvthree.Default3DTiles({
    url: 'http://example.com/tileset.json',
    errorTarget: 16,     // 精度控制
}));

// 配置渲染属性
tiles.receiveShadow = true;
tiles.castShadow = true;
tiles.raycastMethod = 1;  // 开启射线检测（可点击）
tiles.enabledSchedule = true;
tiles.deferOutSideFrustum = true;
tiles.dynamicScreenSpaceError = true;
tiles.dynamicScreenSpaceErrorFactor = 64;

// 点击事件
tiles.addEventListener('click', e => {
    console.log('3DTiles 被点击', e);
    if (e.entity) {
        console.log('实体ID:', e.entity.id);
        console.log('实体类型:', e.entity.dataType);
    }
});

// 2秒后聚焦到3DTiles范围
setTimeout(() => {
    engine.map.zoomTo(tiles);
}, 2000);
```

---

## Default3DTilesGroup

用于同时加载和管理多个 3DTiles 数据集的分组容器。所有属性操作会自动同步到组内所有 3DTiles 实例。

### 构造函数

```javascript
new mapvthree.Default3DTilesGroup(options)
```

### 参数

| 参数名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| `url` | `string[]` | **必填** | 多个 tileset.json 的 URL 数组 |
| 其他参数 | - | - | 与 `Default3DTiles` 相同，会传递给每个子实例 |

### 代理属性

`Default3DTilesGroup` 将以下属性代理到所有子 3DTiles 实例：

`visible`、`castShadow`、`receiveShadow`、`raycastMethod`、`viewFar`、`errorTarget`、`showDebug`、`materialManager`、`elementsManager`、`instancedElementManager`、`editableElementManager`、`freezeUpdate`、`enabledSchedule`、`maxDepth`、`loadSiblings`、`checkIntersectByBox` 等。

### 方法

| 方法名 | 说明 |
|-------|------|
| `lockCameraViewport()` | 锁定所有子实例的相机视角 |
| `releaseCameraViewport()` | 释放所有子实例的相机视角锁定 |
| `getBounds()` | 获取第一个子实例的包围盒 |

### 分组示例

```javascript
// 加载多个 3DTiles 数据集
const tileGroup = engine.add(new mapvthree.Default3DTilesGroup({
    url: [
        'http://example.com/area1/tileset.json',
        'http://example.com/area2/tileset.json',
        'http://example.com/area3/tileset.json',
    ],
    errorTarget: 64,
    loaders: [
        [/\.gltf$/, mapvthree.gltfLoader],
    ],
}));

// 统一设置属性，自动同步到所有子实例
tileGroup.receiveShadow = true;
tileGroup.castShadow = true;
tileGroup.showDebug = false;

// 事件绑定
tileGroup.addEventListener('click', e => {
    console.log('3DTiles 分组被点击', e);
});
```

---

## HDMap3DTiles

高精地图专用 3DTiles 加载器，继承自 `Default3DTiles`，内置了护栏、隧道、路面柱等实例化元素配置，支持样式文件自动加载和材质分组控制。

### 构造函数

```javascript
new mapvthree.HDMap3DTiles(options)
```

### 参数

在 `Default3DTiles` 参数基础上，增加：

| 参数名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| `enableStyle` | `boolean` | `true` | 是否自动加载 style.json 样式文件 |

### 属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| `isHDMap3DTiles` | `boolean` | 标识为高精地图3DTiles |
| `edit` | `HDMapEdit` | 高精地图编辑器（懒加载） |
| `turn` | `HDMapTurnEdit` | 转弯编辑器（懒加载） |
| `visibility` | `HDMapVisibility` | 可见性控制器（懒加载） |
| `grid` | `HDMapGrid` | 网格控制器（懒加载） |

### 方法

| 方法名 | 参数 | 返回值 | 说明 |
|-------|------|--------|------|
| `setGroupMaterial(groupName, properties)` | 分组名, 材质属性 | `this` | 设置材质分组效果 |
| `resetGroupMaterial(groupName)` | 分组名 | `this` | 重置材质分组为默认状态 |
| `getMaterialGroups()` | - | `string[]` | 获取所有可用的材质分组名称 |
| `isEntityVisible(entity)` | 实体对象 | `boolean` | 检查实体是否可见 |

### 高精地图示例

```javascript
// 加载高精地图3DTiles
const hdmap = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap/tiles-box-plane/top.json',
    errorTarget: 16,
}));

// 设置写实风格材质管理器
hdmap.materialManager = new mapvthree.Realistic3DTilesMaterialManager();

// 阴影和射线检测
hdmap.receiveShadow = true;
hdmap.castShadow = true;
hdmap.raycastMethod = 1;
hdmap.enabledSchedule = true;

// 开启标注和树木
hdmap.instancedElementManager.labelEnabled = true;
hdmap.instancedElementManager
    .setInstanceCollectionConfig('tree', { enabled: true });

// 注册元素管理器
hdmap.elementsManager.subscribedMaxLodLevel = 2;

const roadLight = new mapvthree.RoadLight3DTilesElement();
const trafficLight = new mapvthree.TrafficLight3DTilesElement();
hdmap.elementsManager.registerElement(roadLight);
hdmap.elementsManager.registerElement(trafficLight);
```

### 材质分组控制

HDMap3DTiles 支持按分组控制材质效果，适用于隧道透明、道路高亮等场景。

```javascript
// 设置隧道半透明
hdmap.setGroupMaterial('tunnel', {
    transparent: true,
    opacity: 0.3,
    depthWrite: false,
});

// 重置隧道材质为默认状态
hdmap.resetGroupMaterial('tunnel');

// 获取所有可用的材质分组名称
const groups = hdmap.getMaterialGroups();
console.log(groups); // 例如: ['tunnel', 'road', 'building', 'green', 'water']
```

---

## 材质管理器

材质管理器用于控制 3DTiles 模型的材质表现。通过 `tiles.materialManager` 属性设置，系统内置了多种预设管理器。

### Default3DTilesMaterialManager（基类）

所有材质管理器的基类，提供材质的增删查、分组管理和 PBR 材质创建等基础能力。

#### 方法

| 方法名 | 参数 | 返回值 | 说明 |
|-------|------|--------|------|
| `getMaterialByKey(key)` | 材质 key | `Material` | 根据 key 获取材质，未匹配时回退到通配符 `*` |
| `addMaterialByKey(key, material, group)` | key, 材质, 分组名(可选) | - | 添加材质，可指定分组 |
| `removeMaterialByKey(key)` | key | - | 移除材质 |
| `getMaterialsByGroup(groupName)` | 分组名 | `Material[]` | 获取分组下所有材质 |
| `getGroupNames()` | - | `string[]` | 获取所有分组名 |
| `applyGroupMaterial(groupName, properties)` | 分组名, 属性 | `this` | 批量设置分组材质属性 |
| `resetGroupMaterial(groupName)` | 分组名 | `this` | 重置分组材质 |
| `createPbrMaterial(textureName, channels, repeat, params)` | 纹理名, 通道配置, 重复, 参数 | `MeshStandardMaterial` | 创建 PBR 材质 |
| `dispose()` | - | - | 销毁所有材质和纹理 |

#### 自定义材质管理器示例

```javascript
// 创建基础材质管理器
const materialManager = new mapvthree.Default3DTilesMaterialManager();

// 按 key 添加自定义材质
materialManager.addMaterialByKey('road', new THREE.MeshStandardMaterial({
    color: 0x333333,
    roughness: 0.8,
}));

materialManager.addMaterialByKey('building', new mapvthree.BatchBuildingMaterial());

materialManager.addMaterialByKey('water', new mapvthree.WaterMaterial());

// 添加材质到指定分组
materialManager.addMaterialByKey('tunnel_wall', new THREE.MeshStandardMaterial({
    color: 0xaaaaaa,
}), 'tunnel');  // 属于 'tunnel' 分组

// 应用到3DTiles
tiles.materialManager = materialManager;
```

### 内置材质管理器一览

| 名称 | 类名 | 适用场景 | 说明 |
|------|------|---------|------|
| 写实风格 | `Realistic3DTilesMaterialManager` | 高精地图、城市场景 | PBR 材质，含建筑夜灯效果、天气响应 |
| 自定义风格 | `Custom3DTilesMaterialManager` | 通用场景 | 预设道路/绿地/建筑/水面材质 |
| 建筑风格 | `Building3DTilesMaterialManager` | 建筑模型 | 自动加载建筑贴图，支持夜间亮灯 |
| 高精地图 | `HDMap3DTilesMaterialManager` | HD Map | 基于 typeId 的材质映射系统 |
| 高精地图V2 | `HDMap3DTilesMaterialManagerV2` | HD Map V2 | 新版高精地图材质，细分路面/边坡/隧道等类型 |
| 高清路网 | `HDRoad3DTilesMaterialManager` | 路网渲染 | 道路、绿化、路线标注材质 |
| 地形 | `Terrain3DTilesMaterialManager` | 地形渲染 | 地形分类材质，含水面 |
| 随机颜色 | `RandomColor3DTilesMaterialManager` | 调试用 | 每个材质 key 分配随机颜色 |
| 线框模式 | `Wireframe3DTilesMaterialManager` | 调试用 | 全局线框显示 |
| ID识别 | `Identity3DTilesMaterialManager` | 数据查看 | 按 ID 或 dataType 着色 |

### Realistic3DTilesMaterialManager（写实风格）

写实风格材质管理器，支持自动日夜灯光、天气响应。

```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap/top.json',
    errorTarget: 16,
}));

// 设置写实风格材质
const mm = new mapvthree.Realistic3DTilesMaterialManager();
tiles.materialManager = mm;

// 控制自动日夜灯光
mm.autoTimingLight = true;  // 跟随天空时间自动切换灯光
mm.nightLightDensity = 0.5; // 手动设置夜间灯光强度（autoTimingLight=false 时生效）
```

### Building3DTilesMaterialManager（建筑风格）

专为建筑模型设计，自动加载建筑贴图，支持基于天空时间的夜间亮灯效果。

```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/building/top.json',
    errorTarget: 16,
}));

// 设置建筑材质管理器
tiles.materialManager = new mapvthree.Building3DTilesMaterialManager();
```

### HDMap3DTilesMaterialManagerV2（高精地图V2）

新版高精地图材质管理器，支持更丰富的路面、边坡、隧道、收费站等材质类型，并支持材质分组控制。

```javascript
const tile = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap_v2/tileset.json',
}));

// 使用V2版材质管理器
tile.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2();

// 开启树木实例
tile.instancedElementManager
    .setInstanceCollectionConfig('tree', { enabled: true });

// 控制隧道透明度
tile.setGroupMaterial('tunnel', {
    transparent: true,
    opacity: 0.3,
    depthWrite: false,
});
```

### Custom3DTilesMaterialManager + 自定义替换

```javascript
// 使用自定义材质管理器并覆盖特定材质
tiles.materialManager = new mapvthree.Custom3DTilesMaterialManager();

// 覆盖路面材质为纯色
tiles.materialManager.addMaterialByKey('road', new THREE.MeshBasicMaterial({
    color: 'rgb(84, 97, 128)',
}));

// 覆盖绿地材质
tiles.materialManager.addMaterialByKey('green', new THREE.MeshBasicMaterial({
    color: 'rgb(26, 68, 67)',
}));
```

### RandomColor3DTilesMaterialManager（调试用）

为每个材质 key 自动分配随机颜色，快速区分不同类型的几何体。

```javascript
tiles.materialManager = new mapvthree.RandomColor3DTilesMaterialManager();
```

### Wireframe3DTilesMaterialManager（调试用）

以线框模式渲染整个 3DTiles，用于查看网格结构。

```javascript
tiles.materialManager = new mapvthree.Wireframe3DTilesMaterialManager();
```

### Identity3DTilesMaterialManager（ID识别）

按实体 ID 或 dataType 着色渲染，用于数据可视化和调试。

```javascript
const mm = new mapvthree.Identity3DTilesMaterialManager();
tiles.materialManager = mm;

// type=1 按ID着色，type=2 按dataType着色
mm.type = 1;
```

---

## 元素管理器

### ElementsManager（动态元素管理器）

用于从 3DTiles 的 BatchTable 中提取特定类型的元素（如信号灯、路灯），并将其作为独立的可交互对象管理。

#### 属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| `subscribedMaxLodLevel` | `number` | 关注的最大 LOD 层级（默认 1），超过此层级的瓦片数据不解析 |

#### 方法

| 方法名 | 参数 | 返回值 | 说明 |
|-------|------|--------|------|
| `registerElement(element)` | `AbstractElement` | `AbstractElement` | 注册一个元素处理器 |
| `unregisterElement(element)` | `AbstractElement` | - | 注销一个元素处理器 |

#### 内置元素类型

**TrafficLightElement（信号灯元素）**

订阅 dataType: 110, 111, 112, 113

| 方法名 | 参数 | 说明 |
|-------|------|------|
| `setLightState(id, state, number)` | ID, 状态(1红/2黄/3绿), 倒计时秒数 | 设置信号灯状态 |
| `setTimeTable(id, timetable)` | ID, 配时表对象 | 设置信号灯配时表 |
| `getLightState(id)` | ID | 获取信号灯状态 |
| `getTimeTable(id)` | ID | 获取配时表 |
| `getRealtimeState(id)` | ID | 获取实时灯态和倒计时 |

配时表格式：

```javascript
{
    startTime: 1654863684,   // 起始时间（秒级时间戳）
    period: 60,              // 周期（秒）
    states: [[0, 3], [25, 2], [30, 1]],  // [时间偏移, 灯态]
    // 灯态：1=红灯, 2=黄灯, 3=绿灯
}
```

**RoadLightElement（路灯元素）**

订阅 dataType: 101

| 方法名 | 参数 | 说明 |
|-------|------|------|
| `setPowerState(id, on)` | ID, 开关 | 设置路灯开关 |
| `togglePowerState(id)` | ID | 切换路灯开关 |
| `isPowerOn(id)` | ID | 查询路灯是否开启 |
| `setColor(id, color)` | ID, 颜色值 | 设置路灯颜色 |
| `allPowerOn` | `boolean` | 全部路灯开关（属性） |

#### 动态元素完整示例

```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap/top.json',
    errorTarget: 16,
}));

// 设置元素管理器的LOD层级
tiles.elementsManager.subscribedMaxLodLevel = 2;

// 注册路灯和信号灯元素
const roadLight = new mapvthree.RoadLight3DTilesElement();
const trafficLight = new mapvthree.TrafficLight3DTilesElement();
tiles.elementsManager.registerElement(roadLight);
tiles.elementsManager.registerElement(trafficLight);

// 点击事件处理
tiles.addEventListener('click', e => {
    if (!e.entity) return;

    // 路灯交互（dataType === 7 为示例，实际取决于数据）
    if (e.entity.dataType === 7) {
        // 切换路灯开关
        roadLight.togglePowerState(e.entity.id);
        // 设置随机颜色
        roadLight.setColor(e.entity.id, 0xffffff * Math.random());
    }

    // 信号灯交互
    if (e.entity.dataType === 14) {
        // 方式1：直接设置灯态
        trafficLight.setLightState(e.entity.id, 1, 10); // 红灯，倒计时10秒

        // 方式2：设置配时表（自动循环）
        trafficLight.setTimeTable(e.entity.id, {
            startTime: Date.now() / 1000,
            period: 60,
            states: [[0, 3], [25, 2], [30, 1]], // 25秒绿灯->5秒黄灯->30秒红灯
        });
    }
});

// 动态注销/注册元素
// tiles.elementsManager.unregisterElement(roadLight);
```

### InstancedElementManager（实例化元素管理器）

已废弃（deprecated），请使用 `TileInstancedElementManager`。

### TileInstancedElementManager（瓦片实例化元素管理器）

管理树木、POI、标注等实例化元素，通过配置控制不同类型元素的显示。

#### 属性

| 属性名 | 类型 | 说明 |
|-------|------|------|
| `treeEnabled` | `boolean` | 是否显示树木 |
| `labelEnabled` | `boolean` | 是否显示标注 |
| `treeModelPath` | `string` | 自定义树木模型路径 |

#### 方法

| 方法名 | 参数 | 说明 |
|-------|------|------|
| `setInstanceCollectionConfig(type, config)` | 类型名, 配置对象 | 设置实例集合配置 |

#### 实例化元素示例

```javascript
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap/top.json',
}));

// 开启标注
tiles.instancedElementManager.labelEnabled = true;

// 开启树木显示
tiles.instancedElementManager
    .setInstanceCollectionConfig('tree', { enabled: true });

// 自定义树木模型
tiles.instancedElementManager.treeModelPath =
    'https://example.com/models/custom_tree.glb';
```

### EditableElementManager（可编辑元素管理器）

支持按 ID 隐藏/显示 3DTiles 中的实体，用于编辑场景。

#### 方法

| 方法名 | 参数 | 说明 |
|-------|------|------|
| `addDeletedId(id)` | 实体ID | 隐藏指定 ID 的实体 |
| `addDeletedIds(ids)` | ID 数组 | 批量隐藏实体 |
| `removeDeletedId(id)` | 实体ID | 恢复显示实体 |
| `removeDeletedIds(ids)` | ID 数组 | 批量恢复显示 |
| `hasDeletedId(id)` | 实体ID | 查询是否已隐藏 |
| `requestUpdate()` | - | 手动触发刷新 |

```javascript
// 隐藏指定实体
tiles.editableElementManager.addDeletedId('entity_001');
tiles.editableElementManager.addDeletedIds(['entity_002', 'entity_003']);

// 恢复显示
tiles.editableElementManager.removeDeletedId('entity_001');
```

---

## 自定义元素扩展

通过继承 `AbstractElement`，可以创建自定义的动态元素类型。

### AbstractElement 基类

| 属性/方法 | 说明 |
|----------|------|
| `subscribedDataTypeIds` | 需要订阅的 dataType ID 数组 |
| `createMeshFromGeometry(geometry, elementObject)` | 从几何体创建 Mesh，可重写 |
| `addTileElementObjects(tile, elementObjects)` | 向瓦片添加元素对象 |
| `showTileElementObjects(tile)` | 显示瓦片中的元素 |
| `hideTileElementObjects(tile)` | 隐藏瓦片中的元素 |
| `disposeTileElementObjects(tile)` | 销毁瓦片中的元素 |
| `onEntityShow(id, object)` | 实体显示时的回调 |
| `tickObject(object, time)` | 每帧更新回调 |
| `dispose()` | 销毁元素 |

```javascript
// 自定义元素示例
class MyCustomElement extends mapvthree.AbstractElement {
    // 订阅的 dataType
    subscribedDataTypeIds = [200, 201];

    createMeshFromGeometry(geometry, elementObject) {
        const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
        return new THREE.Mesh(geometry, material);
    }

    onEntityShow(id, object) {
        // 实体显示时的逻辑
        console.log('实体显示:', id);
    }

    tickObject(object, time) {
        // 每帧更新（如动画）
        object.rotation.z = time * 0.001;
    }
}

// 注册自定义元素
const myElement = new MyCustomElement();
tiles.elementsManager.registerElement(myElement);
```

---

## 与百度地图集成

### 在 BMapGL 上加载 3DTiles

```javascript
// 创建百度地图
const map = new BMapGL.Map('map_container', {
    maxZoom: 25,
});
map.centerAndZoom(new BMapGL.Point(105.93, 29.36), 21);
map.enableScrollWheelZoom();

// 在百度地图上创建引擎
const engine = new mapvthree.Engine(map);

// 加载3DTiles
const tiles = engine.add(new mapvthree.Default3DTiles({
    url: 'http://example.com/tileset.json',
}));

// 点击事件
tiles.addEventListener('click', e => {
    console.log('3DTiles 被点击', e);
});
```

---

## 常见场景

### 场景一：城市建筑渲染

```javascript
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    map: { is3DControl: true },
    rendering: {
        sky: null,
        features: {
            antialias: { enabled: true },
            shadow: { enabled: true, method: 'default', shadowMapSize: 2048 },
            bloom: { enabled: true },
        },
        enableAnimationLoop: true,
    },
});

engine.map.setCenter([117.31, 40.04]);
engine.map.setZoom(15);
engine.map.setPitch(60);

// 天空和天气
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 3600 * 14;

// 地面
const position = engine.map.projectArrayCoordinate([117.31, 40.04]);
const ground = new THREE.Mesh(
    new THREE.CircleGeometry(1, 16),
    new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 1 })
);
ground.position.set(position[0], position[1], -0.1);
ground.scale.setScalar(100000);
ground.receiveShadow = true;
engine.add(ground);

// 加载建筑3DTiles
const tiles = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/building/top.json',
    errorTarget: 16,
}));
tiles.receiveShadow = true;
tiles.castShadow = true;
tiles.raycastMethod = 1;
tiles.materialManager = new mapvthree.Building3DTilesMaterialManager();

// 自动聚焦
setTimeout(() => engine.map.zoomTo(tiles), 3000);
```

### 场景二：高精地图 + 树木 + POI

```javascript
// 高精地图底图
const hdmap = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap/top.json',
}));
hdmap.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2();
hdmap.instancedElementManager.labelEnabled = true;
hdmap.instancedElementManager
    .setInstanceCollectionConfig('tree', { enabled: true });
```

### 场景三：多区域数据分组加载

```javascript
const tileGroup = engine.add(new mapvthree.Default3DTilesGroup({
    url: [
        'http://example.com/region1/tileset.json',
        'http://example.com/region2/tileset.json',
        'http://example.com/region3/tileset.json',
    ],
    errorTarget: 64,
}));

// 统一配置，自动同步
tileGroup.receiveShadow = true;
tileGroup.materialManager = new mapvthree.Realistic3DTilesMaterialManager();
tileGroup.elementsManager.registerElement(
    new mapvthree.RoadLight3DTilesElement()
);
```

### 场景四：HDMap V2 + 隧道透明控制

```javascript
const tile = engine.add(new mapvthree.HDMap3DTiles({
    url: 'http://example.com/hdmap_v2/tileset.json',
}));

// 使用V2版材质管理器
tile.materialManager = new mapvthree.HDMap3DTilesMaterialManagerV2();

// 开启树木
tile.instancedElementManager
    .setInstanceCollectionConfig('tree', { enabled: true });

// 隧道透明控制
tile.setGroupMaterial('tunnel', {
    transparent: true,
    opacity: 0.3,
    depthWrite: false,
});

// 重置隧道
tile.resetGroupMaterial('tunnel');
```

---

## 注意事项

1. **errorTarget 参数调优**：`errorTarget` 值越小加载的瓦片精度越高，但同时加载量和内存消耗也越大。建议远景用 64，近景用 16。

2. **射线检测模式**：默认 `raycastMethod` 为 1（默认模式）。如果不需要点击交互，设为 0 (`RAYCAST_NONE`) 可提升性能。BVH 模式（2）精度最高但开销最大。

3. **材质管理器设置时机**：`materialManager` 应在 `engine.add(tiles)` 之后尽早设置，以确保后续加载的瓦片都使用新材质。

4. **元素管理器 LOD 控制**：`elementsManager.subscribedMaxLodLevel` 用于控制提取动态元素的层级范围。值越大解析的瓦片越多，建议设为 1~2。

5. **实例化元素配置**：树木、路灯、信号灯、POI 标注等是 `HDMap3DTiles` 的实例化元素图层，**不是独立的 3DTiles 数据集**。通过 `instancedElementManager.setInstanceCollectionConfig('tree', {enabled: true})` 开启树木，通过 `instancedElementManager.labelEnabled = true` 开启 POI 标注。`HDMap3DTiles` 已内置护栏（fence）、隧道（tunnel）、路面柱（roadpillar）等元素配置，可通过 `instancedElementManager.config` 覆盖。

6. **内存管理**：
   - 使用 `cacheBytes` 参数控制瓦片缓存上限。
   - 不再需要 3DTiles 时，使用 `engine.remove(tiles)` 移除并释放资源。
   - 材质管理器通过 `dispose()` 方法释放纹理和材质。

7. **分组加载**：当需要加载多个相邻区域的 3DTiles 时，优先使用 `Default3DTilesGroup` 而非手动创建多个实例，分组可以统一管理属性。

8. **坐标系统**：3DTiles 数据可能使用 ECEF 坐标，通过 `transformFromEcefToPlane(lng, lat, height)` 方法转换到平面坐标系。

9. **enableStyle 控制**：`HDMap3DTiles` 默认会尝试加载 tileset.json 同目录下的 `style.json`。如果不存在会静默忽略。可通过 `enableStyle: false` 禁用。

10. **动态元素生命周期**：通过 `registerElement` 注册的元素会随瓦片的显示/隐藏自动管理。使用 `unregisterElement` 注销后，元素会被自动清理。
