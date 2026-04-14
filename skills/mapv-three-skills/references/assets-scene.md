# AssetsScene 资产场景管理

通过 `trafficAddon.AssetsScene` 可以连接资产管理平台，实现设备模型的加载、添加、编辑、删除等完整生命周期管理。

## 交互判断

AssetsScene 需要以下信息才能工作：
- **projectId 或 sceneId**（二选一，必填）
- **host**（资产平台服务地址，必填）
- **auth.ak**（鉴权 AK，可选）

用户已提供的直接使用，缺少的直接在回复中列出让用户补充。只列缺少的字段。

例如用户说"帮我加载设备资产"，回复：

```
请提供以下信息：
1. projectId 或 sceneId（如：418690084255052706）
2. host 资产平台服务地址（如：http://10.24.22.223）
3. auth.ak 鉴权 AK
```

用户说"帮我加载设备资产，projectId 是 xxx，host 是 http://10.0.0.1"，只需问：

```
请提供鉴权 AK：
```

用户补充后直接生成代码。

**默认关闭自动加载图层**：`useDefault3DTileLayer` 和 `useDefaultLodTileLayer` 默认为 `true`，会自动加载平台配置的 3DTiles 和 LOD 瓦片。生成代码时应将这两项设为 `false`，避免与用户已有的图层冲突。除非用户明确要求加载这些默认图层。

## 构造函数

```javascript
import { trafficAddon } from '@baidu/mapv-three/addons';

new trafficAddon.AssetsScene(options)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `options.sceneId` | `number` | 否 | 场景 ID（与 projectId 二选一） |
| `options.projectId` | `string` | 否 | 项目 ID（与 sceneId 二选一） |
| `options.host` | `string` | 是 | 资产平台服务地址 |
| `options.renderOption.anchor` | `string` | 否 | 数据查询中心，`'cameraPosition'` 或 `'mapCenter'` |
| `options.renderOption.radius` | `number` | 否 | 区域加载半径（米），默认 300 |
| `options.renderOption.ceilRender` | `boolean` | 否 | 是否使用网格查询算法 |
| `options.renderOption.useDefaultSky` | `boolean` | 否 | 是否使用默认天空，默认不加载 |
| `options.renderOption.useDefaultMVTLayer` | `boolean` | 否 | 是否加载平台配置的 MVT 图层，默认 `false` |
| `options.renderOption.useDefault3DTileLayer` | `boolean` | 否 | 是否加载平台配置的 3DTiles 图层，默认 `true` |
| `options.renderOption.useDefaultLodTileLayer` | `boolean` | 否 | 是否加载平台配置的 LOD 瓦片图层，默认 `true` |
| `options.renderOption.useParade` | `boolean` | 否 | 是否获取巡游路线数据（需资产平台配置），默认 `false` |
| `options.auth.ak` | `string` | 否 | 鉴权 AK |

## 方法列表

| 方法 | 参数 | 说明 |
|------|------|------|
| `changeSceneMode(mode)` | `'readOnly'\|'add'\|'edit'\|'delete'` | 切换场景交互模式 |
| `addDevice(point, element, enableEditor?)` | `point: Array`, `element: Object` | 在指定位置添加设备模型 |
| `deleteDevice(device)` | `device: Object3D` | 删除设备 |
| `editDeviceByParams(device, params)` | `device, params` | 编辑设备参数 |
| `submitDeviceInfo(deviceInfo)` | `deviceInfo: Object` | 提交设备信息到平台 |
| `handleActiveDevice(modelInfo, type, options)` | 见 [设备状态控制](./device-control.md) | 触发设备状态变化（情报板/信号灯/路灯等） |
| `openAddPreview(model, target)` | `model, target` | 开启模型添加预览 |
| `getDeviceByUUID(uuid)` | `uuid: string` | 通过设备 UUID 从平台查询设备信息，返回 `Promise<Array<DeviceInfo>>`，见下方数据结构 |
| `setDeviceVisible(key, value, status)` | `key: string, value: any, status: boolean` | 设置单个设备显隐。`key` 为属性名（如 `'uuid'`），`value` 为属性值，`status` 为显隐状态 |
| `lightModelColor(info, color, status?)` | `info: Object3D, color: string, status?: boolean` | 高亮模型。`status=true` 高亮，`false` 取消高亮。支持普通模型和 InstancedMesh |
| `changeModelVisibleByLayer(layer, showStatus)` | `layer: string, showStatus: boolean` | 按图层 ID 批量控制设备显隐，返回受影响的模型数组 |
| `changeAllModelVisible(status)` | `status: boolean` | 全局设备显隐开关 |
| `switchViewRender(status)` | `status: boolean` | 开关区域动态加载（关闭后不再随相机移动加载新设备） |
| `resetEditDevice(deviceList?)` | `deviceList?: Array<string>` | 重置未保存的编辑设备到初始状态。不传则重置全部 |
| `getParadeList()` | - | 获取巡游路线列表，返回 `[{id, name}]` |
| `startParade(id, pathTracker, option?)` | `id: number, pathTracker: PathTracker, option?: Object` | 启动巡游，见下方详细说明 |
| `resetView()` | - | 重置视角到初始位置（AssetsScene 创建时的视角） |

## 巡游（Parade）

AssetsScene 支持接入资产平台配置的巡游路线，实现相机沿预设路径自动巡航浏览场景。

**前提条件：**
- 构造 AssetsScene 时需设置 `renderOption.useParade: true`
- 资产平台中已配置巡游路线数据

**使用流程：**

```javascript
// 1. 创建 AssetsScene，开启巡游
const assetsScene = engine.add(
    new trafficAddon.AssetsScene({
        sceneId: 849,
        host: 'http://10.24.22.223',
        renderOption: {
            useParade: true,  // 开启巡游数据获取
        },
        auth: { ak: 'your_ak' },
    })
);

// 2. 监听 getParadeList 事件获取巡游列表
// 注意：必须在 engine.add() 之后立即注册事件监听，否则可能因数据已加载完成而错过事件
assetsScene.addEventListener('getParadeList', e => {
    console.log('巡游列表:', e.paradeList);
    // e.paradeList 是 API 原始数据（snake_case），包含完整信息：
    // [{id: 1, parade_name: '全线巡游', confs: '{"time":60,"sameSpeed":true}', parade_images: '[...]'}, ...]
});

// 3. 获取巡游列表（简化格式，字段名经过 camelCase 转换）
const paradeList = assetsScene.getParadeList();
// 返回: [{id: 1, name: '全线巡游'}, {id: 2, name: '隧道巡游'}]
// 注意：getParadeList() 返回简化结构（只有 id 和 name），
// 如需完整巡游数据（confs、parade_images 等），使用 assetsScene.paradeInfo 访问原始数据

// 4. 创建 PathTracker
const pathTracker = engine.add(new mapvthree.PathTracker());

// 5. 启动巡游
assetsScene.startParade(paradeList[0].id, pathTracker, {
    time: 1500,        // 每段路径的过渡时间（毫秒），默认 1500
    speed: undefined,  // 速度（传入后 time 失效）
    resetFov: true,    // 巡游结束后是否恢复原始 FOV
    setOriginFov: true,// 巡游开始时是否设置 FOV 为 60
});

// 6. 巡游结束回调
pathTracker.onFinish = () => {
    console.log('巡游结束');
    assetsScene.resetView();
};

// 7. 暂停巡游
pathTracker.pause();

// 8. 恢复巡游
pathTracker.start();

// 9. 停止巡游并重置视角
pathTracker.stop();
assetsScene.resetView();
```

**巡游数据 confs 字段：**

巡游路线的 `confs` 是一个 JSON 字符串，包含以下配置：

| 字段 | 类型 | 说明 |
|------|------|------|
| `time` | `number` | 巡游总时长（秒） |
| `sameSpeed` | `boolean` | 是否匀速巡游（true 时会根据总距离和总时间计算统一速度） |

**匀速巡游：**

当 `confs.sameSpeed` 为 `true` 时，需要根据路径总距离和总时间计算速度：

```javascript
const paradeInfo = assetsScene.paradeInfo.find(p => p.id === targetId);
const confs = JSON.parse(paradeInfo.confs);
const pointList = JSON.parse(paradeInfo.parade_images);

if (confs.sameSpeed) {
    // 计算路径总距离
    let totalDistance = 0;
    for (let i = 1; i < pointList.length; i++) {
        const dx = pointList[i].x - pointList[i - 1].x;
        const dy = pointList[i].y - pointList[i - 1].y;
        const dz = (pointList[i].z || 0) - (pointList[i - 1].z || 0);
        totalDistance += Math.sqrt(dx * dx + dy * dy + dz * dz);
    }
    // 匀速 = 总距离 / 总时间
    const speed = totalDistance / confs.time;
    assetsScene.startParade(targetId, pathTracker, { speed });
} else {
    // 按分段时间
    const time = confs.time * 1000 / pointList.length;
    assetsScene.startParade(targetId, pathTracker, { time });
}
```

**startParade 参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `id` | `number` | - | 巡游路线 ID（从 `getParadeList()` 获取） |
| `pathTracker` | `PathTracker` | - | 路径追踪器实例，需要先 `engine.add()` |
| `option.time` | `number` | `1500` | 每段路径的过渡时间（毫秒） |
| `option.speed` | `number` | - | 速度（世界坐标单位/秒），传入后 `time` 失效。通常通过 `总距离 / 总时间` 计算得出 |
| `option.resetFov` | `boolean` | `true` | 巡游结束后是否恢复原始相机 FOV |
| `option.setOriginFov` | `boolean` | `true` | 巡游开始时是否将相机 FOV 设为 60 度 |

## DeviceInfo 设备数据结构

`subModelClick` 事件的 `e.assetsInfo` 以及内部 `_assetsInfo.deviceList` 中每个设备均为以下结构（经 camelCase 转换后）：

> **注意：** `getDeviceByUUID` 返回的是 API 原始数据，字段名为 snake_case 格式（如 `class_type`、`model_name`），与事件中的 camelCase 格式不同。

| 字段 | 类型 | 说明 |
|------|------|------|
| `uuid` | `string` | 设备唯一标识 |
| `projectId` | `string` | 所属项目 ID |
| `name` | `string` | 设备名称 |
| `x` | `number` | X 坐标（投影坐标） |
| `y` | `number` | Y 坐标（投影坐标） |
| `z` | `number` | Z 坐标（高度） |
| `rotateX` | `number` | X 轴旋转（度） |
| `rotateY` | `number` | Y 轴旋转（度） |
| `rotateZ` | `number` | Z 轴旋转（度） |
| `scaleX` | `number` | X 轴缩放，默认 1 |
| `scaleY` | `number` | Y 轴缩放，默认 1 |
| `scaleZ` | `number` | Z 轴缩放，默认 1 |
| `classType` | `string` | 设备分类类型 |
| `modelName` | `string` | 模型名称 |
| `layerId` | `number` | 所属图层 ID |
| `layerName` | `string` | 图层名称 |
| `deviceId` | `string` | 外部设备 ID |
| `isTips` | `number` | 是否显示提示（0/1） |
| `isVisible` | `number` | 是否可见（0/1） |
| `isClick` | `number` | 是否可点击（0/1） |
| `isBloom` | `number` | 是否开启泛光（0/1） |
| `status` | `number` | 设备状态 |
| `groupInfo` | `string` | 组合模型信息（JSON 字符串） |

**使用示例：**

```javascript
// 通过 UUID 查询设备信息
const devices = await assetsScene.getDeviceByUUID('692302028113879040');
console.log('设备名称:', devices[0].name);
console.log('设备位置:', devices[0].x, devices[0].y, devices[0].z);
console.log('设备类型:', devices[0].class_type);

// 在 subModelClick 事件中获取设备信息
assetsScene.addEventListener('subModelClick', e => {
    console.log('设备 UUID:', e.assetsInfo.uuid);
    console.log('设备名称:', e.assetsInfo.name);
    console.log('设备分类:', e.assetsInfo.classType);
});
```

## handleActiveDevice

控制设备状态（情报板文字、信号灯、路灯开关、风机旋转、卷帘门升降等）。支持 8 种设备类型，详细参数和示例见 [设备状态控制](./device-control.md)。

## 事件列表

| 事件名 | 触发时机 | 回调参数 |
|--------|----------|----------|
| `subModelClick` | 点击设备子模型 | `{ model, device, assetsInfo }` |
| `modelClick` | 点击设备模型 | `{ model, config }` |
| `3DTilesClick` | 点击 3DTiles 路面 | `{ value: { point } }` |
| `getModelList` | 模型列表加载完成 | `{ modelList }` |
| `assetsLibModelLoaded` | 初始化时设备模型加载完成（只触发一次） | `{ models }` |
| `sceneViewAdd` | 相机移动后新区域的设备模型加载完成（每次区域加载触发） | `{ models }` |
| `mapLoaded` | 地图加载完成 | - |
| `update` | 设备位置更新 | `{ deviceInfo }` |
| `getParadeList` | 巡游列表加载完成 | `{ paradeList }` |

## 常见场景

```javascript
import { trafficAddon } from '@baidu/mapv-three/addons';

// 假设 engine 已初始化（需开启 enableAnimationLoop）

// 创建资产场景
const assetsScene = engine.add(
    new trafficAddon.AssetsScene({
        projectId: '418690084255052706',
        host: 'http://10.24.22.223',
        renderOption: {
            anchor: 'cameraPosition',
            radius: 300,
            ceilRender: true,
            useDefaultSky: true,
            useDefaultMVTLayer: false,
            useDefault3DTileLayer: false,
            useDefaultLodTileLayer: false,
        },
        auth: {
            ak: '你的鉴权AK',
        },
    })
);

// 监听模型点击
assetsScene.addEventListener('modelClick', e => {
    console.log('点击模型:', e.model, '资产信息:', e.config);
});

assetsScene.addEventListener('subModelClick', e => {
    console.log('子模型:', e.model, '主模型:', e.device, '资产信息:', e.assetsInfo);
});

// 切换模式
assetsScene.changeSceneMode('add'); // 进入新增模式

// 在 3DTiles 点击位置添加设备
assetsScene.addEventListener('3DTilesClick', async e => {
    const point = e.value.point;
    const { model, deviceInfo } = await assetsScene.addDevice(point, { file_name: 'device_id' }, false);
    // 保存到平台
    await assetsScene.submitDeviceInfo(deviceInfo);
});

// 显示/隐藏
assetsScene.visible = false;
assetsScene.visible = true;

// 销毁
engine.remove(assetsScene);
```

## 注意事项

1. **来源包**：AssetsScene 来自 addons 包（`@baidu/mapv-three/addons`），需要额外引入 `trafficAddon`。

2. **区域加载半径**：`radius` 参数控制区域加载范围，设置过大会导致设备过多引起性能问题。

3. **默认图层冲突**：`useDefault3DTileLayer` 和 `useDefaultLodTileLayer` 默认为 `true`，会自动加载平台配置的图层。如果用户已有自己的 3DTiles 或 LOD 图层，应设为 `false` 避免冲突。

4. **事件监听时机**：`assetsLibModelLoaded` 和 `getParadeList` 事件必须在 `engine.add()` 之后立即注册，否则可能因数据已加载完成而错过事件。

5. **设备状态控制**：`handleActiveDevice` 的详细用法见 [设备状态控制](./device-control.md)，支持情报板、信号灯、路灯、风机、卷帘门、升降杆、隧道指示灯、防火门等 8 种设备类型。
