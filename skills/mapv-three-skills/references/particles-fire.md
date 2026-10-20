# 火灾效果与粒子系统

普通地理场景优先使用 `FireEffect`。它负责经纬度投影、每帧更新、按需请求渲染和多处火灾管理。`ParticleSystem` 是不认识地图坐标的底层粒子容器，仅在编写自定义粒子预设时直接使用。

`FireEffect`、`ParticleSystem`、`FIRE_PRESET` 和 `PARTICLE_PRESETS` 在完整包与开放版入口均已导出。

## FireEffect 快速开始

```javascript
const fireEffect = engine.add(new mapvthree.FireEffect({
    severity: 'medium',
    wind: [3, 1],
}));

const fire = fireEffect.addFire({
    coordinates: [116.404, 39.915, 0],
    severity: 'large',
    intensity: 1,
});

fire.intensity = 0.5;
fire.wind = [6, 0];
fire.enabled = false;
```

一个 `FireEffect` 可以管理多处火灾。`addFire()` 也可以在加入引擎前调用，参数会先保留，加入引擎后再创建 GPU 资源。

### FireEffect 参数

| 参数 | 类型／默认值 | 说明 |
| --- | --- | --- |
| `severity` | `number\|string`／`'medium'` | 全局严重程度，数字范围 `0～1`，或 `small`、`medium`、`large`、`extreme` |
| `color` | CSS 颜色、数字或颜色数组 | 覆盖严重程度预设的火焰颜色 |
| `fireScale` | `number`／`1` | 全局尺寸倍数；不要写成 `scale` |
| `wind` | `[east,north]`／`[0,0]` | 东向和北向风速，单位为米／秒 |
| `maxParticles` | `Object` | 按 `smoke`、`flame`、`spark` 覆盖粒子池上限 |
| `renderMode` | `'points'\|'instanced'\|Object` | 全层或逐层渲染模式；默认各层使用 `points` |
| `bucketSize` | `number`／`10000` | RTC 分桶边长，普通业务保持默认 |
| `autoRequestRender` | `boolean`／`true` | 是否在存在火灾时持续请求下一帧 |

`points` 带宽较低，适合大量或远景火灾；`instanced` 使用面片，不受 `gl_PointSize` 上限影响，近景火焰方向更自然，但开销更高。

### addFire 与 FireHandle

```javascript
const fire = fireEffect.addFire({
    coordinates: [116.404, 39.915, 5],
    severity: 0.8,
    color: '#ff6a22',
    scale: 1.2,
    wind: [4, -1],
    intensity: 1.5,
    enabled: true,
});
```

| 字段／属性 | 说明 |
| --- | --- |
| `coordinates` | `[经度,纬度,高度]`，高度单位为米；运行时赋值可移动整团火灾 |
| `severity` | 单处覆盖值；设为 `undefined` 后重新跟随全局值 |
| `color` | 单处颜色覆盖；单色会派生火焰渐变 |
| `scale` | 单处尺寸倍数 |
| `wind` | 单处东向、北向风速 |
| `intensity` | 发射速率倍数；设为 `0` 后停止产生新粒子 |
| `enabled` | 关闭后停止发射，已有粒子按生命周期淡出 |
| `remove()` | 删除当前火灾 |

修改 `intensity` 适合连续表现火势变化；修改 `severity` 会重建对应发射器，已有粒子可能立即消失。

### 管理方法和默认值

| API | 说明 |
| --- | --- |
| `addFire(options)` | 新增并返回 `FireHandle` |
| `removeFire(handle)` | 删除指定火灾 |
| `clear()` | 删除全部火灾，但保留效果对象 |
| `fireCount` | 当前火灾数量 |
| `severity`、`color`、`fireScale`、`wind` | 修改所有没有单独覆盖该字段的火灾 |
| `dispose()` | 完全释放粒子资源并清空句柄 |

临时 `engine.remove(fireEffect)` 会释放 GPU 桶但保留火灾句柄，再次添加时自动恢复；`dispose()` 会清空句柄。

## ParticleSystem 高级用法

```javascript
const system = new mapvthree.ParticleSystem({
    preset: mapvthree.FIRE_PRESET,
    onNeedsRender: () => engine.requestRender(),
});

const emitter = system.createEmitter({
    position: new THREE.Vector3(0, 0, 0),
    layerOptions: system.resolveLayerOptions({
        severity: 'small',
        metersToUnits: 1,
    }),
});
```

直接使用 `ParticleSystem` 时，调用方还必须负责场景坐标、把对象加入场景，以及在渲染循环中执行 `system.update(deltaSeconds, renderer, camera)`。需要地理火灾时不要重复实现这些工作，直接使用 `FireEffect`。

`preset` 可传内置名字 `'fire'`，也可传 `{name,layers,resolve}` 自定义预设。`createEmitter()` 接受的是 `resolveLayerOptions()` 生成的逐层参数，不是 `severity` 等业务参数。资源释放使用 `removeEmitter()`、`clear()` 和 `dispose()`。
