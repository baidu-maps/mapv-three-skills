# 动画追踪器

MapV-Three 提供了三种动画追踪器，用于实现路径巡游、环绕浏览和对象跟随等相机动画效果。

## 快速开始

```javascript
// 路径追踪
const tracker = engine.add(new mapvthree.PathTracker());
tracker.track = [
    [116.404, 39.915, 0],
    [116.414, 39.925, 0],
    [116.424, 39.920, 0],
];
tracker.start({
    duration: 10000,
    pitch: 60,
    range: 500,
});
```

---

## PathTracker - 路径追踪器

沿指定路径进行相机或对象的动画追踪。继承自 TrackerAbstract。

### 构造函数

```javascript
const tracker = engine.add(new mapvthree.PathTracker());
```

### track 属性 - 设置路径数据

支持三种数据格式：

```javascript
// 格式1：坐标数组
tracker.track = [
    [112.368, 23.177, 38],
    [112.370, 23.179, 40],
    [112.372, 23.181, 42],
];

// 格式2：GeoJSON
tracker.track = {
    geometry: {
        type: 'LineString',
        coordinates: [[112.368, 23.177, 38], [112.370, 23.179, 40]],
    },
    properties: {
        frameInfo: [/* 帧信息数组 */],
    },
};

// 格式3：帧信息数组
tracker.track = [
    { x: 113.34, y: 23.04, z: 32, yaw: 7.69, pitch: 1.44, speed: 72.7 },
    { x: 113.34, y: 23.05, z: 35, yaw: 7.66, pitch: 1.40, speed: 72.7 },
];
```

### start 方法参数

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `options.duration` | `number` | `1000` | 动画持续时间（毫秒） |
| `options.heading` | `number` | `0` | 方位角偏移 |
| `options.pitch` | `number` | `60` | 俯仰角偏移 |
| `options.range` | `number` | `100` | 相机距离 |
| `options.easing` | `string \| Function` | `'linear'` | 缓动函数：`'linear'`、`'ease-in'`、`'ease-out'`、`'ease-in-out'` 或自定义函数 |
| `options.keepRunning` | `boolean` | `false` | 是否持续循环运行 |
| `options.repeatCount` | `number` | `1` | 重复次数，`Infinity` 为无限循环 |
| `options.delay` | `number` | `0` | 延迟开始时间（毫秒） |
| `options.direction` | `string` | `'normal'` | 播放方向：`'normal'`、`'reverse'`、`'alternate'`、`'alternate-reverse'` |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `track` | `Array \| Object` | 获取/设置路径数据 |
| `viewMode` | `string` | 视图模式：`'follow'`（跟随）、`'lock'`（锁定）、`'unlock'`（自由）、`'keyFrame'`、`'activeFrame'` |
| `object` | `Object3D` | 设置跟随路径移动的 3D 对象 |
| `pointHandle` | `string` | 轨迹点插值方式，设为 `'curve'` 启用 CatmullRom 曲线平滑 |
| `interpolateDirectThreshold` | `number` | 插值平滑阈值，值越大拐角过渡越平滑 |
| `isRunning` | `boolean` | 是否正在运行（只读） |
| `isPaused` | `boolean` | 是否已暂停（只读） |
| `currentState` | `Object` | 当前动画状态（只读） |

### 控制方法

| 方法 | 说明 |
| --- | --- |
| `start(options)` | 开始追踪动画 |
| `pause()` | 暂停动画，返回当前状态 |
| `stop()` | 停止动画 |

### 回调

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `onStart` | `Function` | 动画开始时回调 |
| `onFinish` | `Function` | 动画结束时回调 |
| `onUpdate` | `Function` | 每帧更新回调，参数为 `{ point, hpr }` |

### 完整示例

```javascript
// 加载模型
const model = engine.add(new mapvthree.SimpleModel({
    url: 'models/car.glb',
}));

// 创建路径追踪器
const tracker = engine.add(new mapvthree.PathTracker());

// 设置路径
tracker.track = [
    [112.368, 23.177, 38],
    [112.370, 23.179, 40],
    [112.372, 23.181, 42],
    [112.374, 23.180, 40],
];

// 设置视图模式为锁定跟随
tracker.viewMode = 'lock';

// 启用曲线插值使路径更平滑
tracker.pointHandle = 'curve';

// 绑定模型跟随路径移动
tracker.object = model;

// 逐帧回调
tracker.onUpdate = (state) => {
    console.log('当前位置:', state.point);
    console.log('当前朝向:', state.hpr);
};

// 开始动画
tracker.start({
    duration: 10000,  // 10 秒
    pitch: 80,
    range: 100,
    easing: 'ease-in-out',
});

// 暂停
// tracker.pause();

// 继续（再次调用 start）
// tracker.start({ ... });

// 停止
// tracker.stop();
```

### 可视化路径线

```javascript
// 将路径数据同时绘制为线
const lineDS = mapvthree.GeoJSONDataSource.fromGeoJSON([{
    type: 'Feature',
    geometry: {
        type: 'LineString',
        coordinates: tracker.track.map(p => [p[0], p[1], p[2]]),
    },
}]);
const line = engine.add(new mapvthree.Polyline({ flat: true }));
line.dataSource = lineDS;
```

---

## OrbitTracker - 轨道追踪器

围绕指定对象或坐标点进行环绕旋转动画。继承自 TrackerAbstract。

### 构造函数

```javascript
const tracker = engine.add(new mapvthree.OrbitTracker());
```

### start 方法参数

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `options.object` | `Object3D` | `null` | 围绕旋转的 3D 对象（与 center/projectedCenter 三选一） |
| `options.center` | `Array<number>` | `null` | 围绕旋转的地理坐标 `[lng, lat, alt]` |
| `options.projectedCenter` | `Array<number>` | `null` | 围绕旋转的投影坐标 `[x, y, z]` |
| `options.radius` | `number` | `100` | 旋转半径 |
| `options.startAngle` | `number` | `0` | 起始角度（度） |
| `options.endAngle` | `number` | `360` | 结束角度（度） |
| `options.duration` | `number` | `10000` | 动画持续时间（毫秒） |
| `options.heading` | `number` | `0` | 方位角（度） |
| `options.pitch` | `number` | `0` | 俯仰角（度） |
| `options.range` | `number` | `0` | 距离 |
| `options.height` | `number` | `0` | 高度偏移（米） |
| `options.keepRunning` | `boolean` | `true` | 是否持续运行 |
| `options.loopMode` | `string` | `'repeat'` | 循环模式：`'repeat'`（正向循环）、`'alternate'`（往返循环）、`'reverse'`（反向循环） |
| `options.repeatCount` | `number` | `1` | 重复次数 |
| `options.easing` | `string \| Function` | `'linear'` | 缓动函数 |
| `options.useWorldAxis` | `boolean` | `false` | 是否使用世界轴（球面模式下） |

### 属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `viewMode` | `string` | 固定为 `'lock'`，OrbitTracker 仅支持锁定模式 |
| `isRunning` | `boolean` | 是否正在运行（只读） |
| `isPaused` | `boolean` | 是否已暂停（只读） |
| `currentState` | `Object` | 当前动画状态（只读） |

### 控制方法

| 方法 | 说明 |
| --- | --- |
| `start(options)` | 开始环绕动画 |
| `pause()` | 暂停动画，返回当前状态 |
| `stop()` | 停止动画 |

### 回调

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `onStart` | `Function` | 动画开始时回调 |
| `onFinish` | `Function` | 动画结束时回调 |
| `onUpdate` | `Function` | 每帧更新回调，参数为 `{ point, hpr, direction }` |

### 完整示例

```javascript
// 围绕 3D 模型旋转
const model = engine.add(new mapvthree.SimpleModel({
    url: 'models/building.glb',
}));

const orbit = engine.add(new mapvthree.OrbitTracker());
orbit.start({
    object: model,
    radius: 200,
    duration: 15000,
    startAngle: 0,
    endAngle: 360,
    pitch: 30,
    keepRunning: true,
    loopMode: 'repeat',
});

// 围绕地理坐标旋转
const orbit2 = engine.add(new mapvthree.OrbitTracker());
orbit2.start({
    center: [116.404, 39.915, 0],
    radius: 500,
    duration: 20000,
    heading: 45,
    pitch: 60,
    height: 50, // 高度偏移
    keepRunning: true,
});

// 停止
// orbit.stop();
```

---

## ObjectTracker - 对象追踪器

实时追踪指定的 3D 对象或坐标点，相机自动跟随目标移动和旋转。继承自 TrackerAbstract。

### track 方法参数

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `object` | `Object3D \| Object` | **必填** | 追踪的 3D 对象，也支持 `{ instance, instanceIndex }` 格式追踪 InstancedMesh 实例 |
| `config.range` | `number` | `0` | 追踪距离 |
| `config.radius` | `number` | `0` | 追踪距离（`range` 的别名，二者取其一） |
| `config.pitch` | `number` | `0` | 俯仰角 |
| `config.heading` | `number` | `0` | 方位角 |
| `config.height` | `number` | `undefined` | 高度偏移（仅平面/2.5D 模式生效） |
| `config.extraDir` | `number` | `undefined` | 额外方向修正角度（度） |
| `config.duration` | `number` | `0` | 持续时间（0 表示持续追踪） |
| `config.easing` | `string \| Function` | `'linear'` | 缓动函数 |
| `config.lock` | `boolean` | `true` | 是否锁定视角 |

### 控制方法

| 方法 | 说明 |
| --- | --- |
| `track(object, config)` | 开始追踪对象 |
| `stop()` | 停止追踪并重置状态 |
| `pause()` | 暂停追踪，返回当前状态 |

### 回调

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `onStart` | `Function` | 动画开始时回调 |
| `onFinish` | `Function` | 动画结束时回调 |
| `onUpdate` | `Function` | 每帧更新回调，参数为 `{ point, hpr }` |
| `onTrackFrame` | `Function` | 追踪帧回调，参数为 `(lastState, currentState)`，可覆写自定义逻辑 |

### 完整示例

```javascript
// 创建运动的模型（比如车辆）
const car = engine.add(new mapvthree.SimpleModel({
    url: 'models/car.glb',
}));

// 创建对象追踪器
const objTracker = engine.add(new mapvthree.ObjectTracker());

// 开始追踪
objTracker.track(car, {
    range: 100,
    pitch: 60,
    heading: 0,
});

// 逐帧回调
objTracker.onTrackFrame = (lastState, currentState) => {
    console.log('追踪状态:', currentState);
};

// 停止追踪
// objTracker.stop();
```

### 支持追踪 InstancedMesh 中的单个实例

```javascript
// 追踪 InstancedMesh 中的指定实例
objTracker.track({
    instance: instancedMesh,
    instanceIndex: 5,
}, {
    range: 50,
    pitch: 45,
});
```

---

## TrackerAbstract - 追踪器基类

所有追踪器（PathTracker、OrbitTracker、ObjectTracker）的公共基类，提供通用的动画控制能力。

### 视图模式说明

| 模式 | lockView | viewFollow | 说明 |
| --- | --- | --- | --- |
| `follow` | `false` | `true` | 相机跟随但不锁定朝向 |
| `lock` | `true` | `true` | 相机跟随且锁定朝向（默认） |
| `unlock` | `false` | `false` | 相机不跟随，仅对象移动 |
| `keyFrame` | `true` | `true` | 关键帧模式，使用路径数据中的帧信息 |
| `activeFrame` | `true` | `true` | 活跃帧模式，结合帧信息和路径插值 |

---

## 注意事项

1. **追踪器生命周期**：追踪器需要通过 `engine.add` 添加到引擎才能工作。不再使用时调用 `engine.remove(tracker)` 移除。

2. **缓动函数**：内置 `'linear'`、`'ease-in'`、`'ease-out'`、`'ease-in-out'` 四种缓动，也支持传入自定义函数 `(t: number) => number`，其中 t 的范围为 0-1。

3. **暂停与恢复**：调用 `tracker.pause()` 暂停后，再次调用 `tracker.start()` 会从暂停位置恢复，而非重新开始。调用 `tracker.stop()` 则完全停止并重置状态。

4. **球面模式兼容**：OrbitTracker 在球面（ECEF）模式下会自动计算 ENU 局部坐标系以保证旋转轨道正确贴合地球表面。通过 `useWorldAxis: true` 可切换为世界坐标轴旋转。

5. **飞行动画**：使用 `engine.map.flyTo()` 实现视角飞行（见 [引擎初始化](./engine-bindmap.md)），与追踪器是不同的功能。
