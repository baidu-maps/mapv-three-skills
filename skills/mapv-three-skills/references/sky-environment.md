# 天空环境系统

mapv-three 提供了多种天空类型，从轻量级的渐变天空到高级的物理大气散射天空，满足不同性能和视觉需求。所有天空类型均继承自 `EmptySky`，共享基础的光照控制能力。

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 创建引擎
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        sky: null, // 不使用默认天空，手动创建
    },
});

// 方式一：使用动态天空（高品质大气散射 + 体积云）
const sky = engine.add(new mapvthree.DynamicSky());

// 设置时间（秒，3600 * 17.5 = 下午5:30）
sky.time = 3600 * 17.5;
```

## 天空类型总览

| 类名 | 说明 | 性能开销 | 适用场景 |
|------|------|---------|---------|
| `EmptySky` | 空天空，仅提供光照系统 | 最低 | 不需要天空背景，仅需光照 |
| `DefaultSky` | 默认渐变天空 | 低 | 一般场景，平衡性能与效果 |
| `HazeSky` | 雾霾天空 | 低 | 需要雾化渐变效果的场景 |
| `StaticSky` | 静态贴图天空 | 低 | 需要基于时间/天气自动切换纹理 |
| `CustomStaticSky` | 自定义静态天空 | 低 | 使用自定义天空贴图 |
| `VerticalGradientSky` | 垂直渐变天空 | 低 | 自定义垂直方向颜色渐变 |
| `DynamicSky` | 动态天空（大气散射 + 体积云） | 高 | 高品质实时天空效果 |

---

## EmptySky - 空天空

基础天空类，所有天空的父类。仅提供太阳光源（DirectionalLight）和环境光（AmbientLight），不渲染天空背景。

### 构造函数

```javascript
const sky = new mapvthree.EmptySky(options);
engine.add(sky);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.time` | `number` | `-1` | 初始时间，单位秒。例如 `3600 * 15.5` 表示15:30 |

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `time` | `number` | 当前时间（秒），如 `3600 * 17` 表示下午5点 |
| `sunLightIntensity` | `number` | 太阳光强度，默认 `2.2` |
| `skyLightIntensity` | `number` | 天空环境光强度，默认 `1.2` |
| `sunIntensityBias` | `number` | 太阳光强度偏移 |
| `sunIntensityScale` | `number` | 太阳光强度缩放，默认 `0.8` |
| `envLightIntensity` | `number` | 环境光贴图强度，默认 `0.2` |
| `sunLight` | `DirectionalLight` | 太阳光源对象（只读） |
| `skyLight` | `AmbientLight` | 环境光对象（只读） |
| `sunDirection` | `Vector3` | 太阳方向向量（只读） |
| `affectWorld` | `boolean` | 天空颜色是否影响场景环境，默认 `false` |

### 方法

| 方法 | 说明 |
|------|------|
| `addTimeChangedListener(listener)` | 添加时间变化监听器，回调参数为当前时间（秒） |
| `removeTimeChangedListener(listener)` | 移除时间变化监听器 |
| `dispose()` | 释放资源 |

### 示例

```javascript
// 创建空天空
const sky = engine.add(new mapvthree.EmptySky());

// 调整光照
sky.sunLightIntensity = 3.0;
sky.skyLightIntensity = 0.8;

// 监听时间变化
sky.addTimeChangedListener((time) => {
    console.log('当前时间（秒）:', time);
});
```

---

## DefaultSky - 默认天空

基于后处理的渐变天空效果，性能与效果平衡。支持设置天空基础颜色和高处颜色。

### 构造函数

```javascript
const sky = new mapvthree.DefaultSky();
engine.add(sky);
```

无额外构造参数，继承 `EmptySky` 的 `options.time`。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `Color` | `Color(1, 1, 1)` | 天空基础颜色（地平线附近） |
| `highColor` | `Color` | `Color(0.6, 0.8, 1.0)` | 天空高处颜色（天顶方向） |
| `enablePostPass` | `boolean` | `true` | 是否启用天空后处理效果 |

### 示例

```javascript
import * as THREE from 'three';

const sky = engine.add(new mapvthree.DefaultSky());
sky.color = new THREE.Color(0x87ceeb);     // 设置地平线颜色
sky.highColor = new THREE.Color(0x2255aa); // 设置天顶颜色
```

---

## HazeSky - 雾霾天空

轻量级后处理天空，在平面模式下以屏幕空间地平线为分界做天空渐变和远端雾化，在地球模式下基于射线角度计算大气渐变。

### 构造函数

```javascript
const sky = new mapvthree.HazeSky(options);
engine.add(sky);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.fogBlendStart` | `number` | `0.7` | 雾化起始比例（0~1），相对于最远视距 |
| `options.groundRadiusMeters` | `number` | `6371000` | 地球半径（米），用于球体模式计算 |

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `color` | `Color` | `Color(0.9, 0.95, 1.0)` | 地平线/近处天空颜色 |
| `highColor` | `Color` | `Color(0.32, 0.52, 0.88)` | 天顶颜色 |
| `fogBlendStart` | `number` | `0.7` | 雾化起始比例 |
| `groundRadiusMeters` | `number` | `6371000` | 地球半径 |
| `enablePostPass` | `boolean` | `true` | 是否启用后处理效果 |

### 示例

```javascript
const sky = engine.add(new mapvthree.HazeSky({
    fogBlendStart: 0.6, // 从最远距离的 60% 处开始雾化
}));
sky.color = new THREE.Color(0x87ceeb);
sky.highColor = new THREE.Color(0x2255aa);
```

---

## StaticSky - 静态天空

预置了常见天气和时段的天空贴图，根据时间和天气自动切换纹理。继承自 `CustomStaticSky`。

### 构造函数

```javascript
const sky = new mapvthree.StaticSky();
engine.add(sky);
```

无额外构造参数。

### 时间段与天气切换规则

时间段根据秒数自动判断：

| 时间段 | 时间范围 | 标识 |
|--------|----------|------|
| 夜晚 | 18:00 ~ 06:00 | `night` |
| 黄昏 | 17:00 ~ 18:00 | `dusk` |
| 下午 | 15:30 ~ 17:00 | `afternoon` |
| 默认（白天） | 06:00 ~ 15:30 | `default` |

天气类型：`clear`（晴天）、`cloudy`（多云）等，加载对应路径的纹理贴图。

### 属性

继承 `CustomStaticSky` 的所有属性，加上：

| 属性 | 类型 | 说明 |
|------|------|------|
| `weather` | `string` | 当前天气类型 |

### 示例

```javascript
// 通过引擎配置创建
const engine = new mapvthree.Engine(container, {
    rendering: { skyType: 'static' },
});
const sky = engine.rendering.sky;

// 或者手动创建
const sky = engine.add(new mapvthree.StaticSky());

// 通过引擎时钟设置时间
sky.time = 3600 * 17.5;
```

---

## CustomStaticSky - 自定义静态天空

使用自定义纹理贴图作为天空背景，支持等距柱状投影贴图（全景图）和立方体贴图两种格式，支持 HDR 和 LDR 纹理。

### 构造函数

```javascript
const sky = new mapvthree.CustomStaticSky(options);
engine.add(sky);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.isVerticalTexture` | `boolean` | `false` | 是否为垂直纹理（竖向全景图） |

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `textureURL` | `string` | `null` | 天空贴图 URL，设置后自动加载 |
| `textureIsCube` | `boolean` | `false` | 纹理是否为立方体贴图 |
| `isVerticalTexture` | `boolean` | `false` | 是否为垂直纹理 |
| `affectWorld` | `boolean` | `true` | 天空是否影响场景环境反射 |

### 示例 - 全景图天空

```javascript
const sky = engine.add(new mapvthree.CustomStaticSky());
// 设置全景图 URL
sky.textureURL = 'https://example.com/sky_panorama.jpg';
// 天空将自动加载纹理并生成环境贴图
```

### 示例 - 立方体贴图天空

```javascript
const sky = engine.add(new mapvthree.CustomStaticSky());
sky.textureIsCube = true;
// 设置立方体贴图目录路径（包含 px.jpg, nx.jpg 等6个面）
sky.textureURL = 'https://example.com/sky_cube/';
```

### 示例 - HDR 贴图天空

```javascript
const sky = engine.add(new mapvthree.CustomStaticSky());
sky._textureIsHDR = true; // 启用 HDR 加载
sky.textureURL = 'https://example.com/sky.hdr';
```

---

## VerticalGradientSky - 垂直渐变天空

通过设置颜色渐变节点来自定义天空垂直方向的颜色渐变。

### 构造函数

```javascript
const sky = new mapvthree.VerticalGradientSky();
engine.add(sky);
```

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `gradients` | `Array<{stop: number, color: Color}>` | 渐变节点数组，`stop` 范围 0~1 |

### 示例

```javascript
const sky = engine.add(new mapvthree.VerticalGradientSky());
sky.gradients = [
    { stop: 0.0, color: new THREE.Color(1/255, 1/255, 1/255) },
    { stop: 0.01, color: new THREE.Color(31/255/4, 43/255/4, 93/255/4) },
    { stop: 0.03, color: new THREE.Color(31/255/8, 43/255/8, 93/255/8) },
    { stop: 0.09, color: new THREE.Color(31/255/32, 43/255/32, 93/255/32) },
    { stop: 0.15, color: new THREE.Color(1/255, 1/255, 1/255) },
];
```

---

## DynamicSky - 动态天空

高级天空效果，提供大气散射渲染、体积云、环境反射捕获，根据时间自动变化光照和天空外观。性能开销较高，适用于需要高品质天空的场景。

### 构造函数

```javascript
const sky = new mapvthree.DynamicSky(options);
engine.add(sky);
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.time` | `number` | `-1` | 初始时间，单位秒 |

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `affectWorld` | `boolean` | `true` | 天空是否影响场景环境反射 |
| `useVolumetricClouds` | `boolean` | `true` | 是否使用体积云 |
| `mixGrayFactor` | `number` | `0` | 灰度混合因子（0=正常色彩, 1=完全灰色），用于模拟阴天 |
| `cloudsCoverage` | `number` | `0` | 云层覆盖率（0~1） |
| `cloudDensity` | `number` | `0` | 云层密度 |
| `cloudsSpeed` | `number` | `1` | 云层移动速度 |
| `cloudShapeBaseScale` | `number` | `1` | 云层形状基础缩放 |
| `cloudShapeDetailScale` | `number` | `1` | 云层形状细节缩放 |
| `cloudMarchSteps` | `number` | - | 云层渲染步进数（数值越大，质量越高但性能越低） |
| `cloudSelfShadowSteps` | `number` | - | 云层自阴影步进数 |
| `enableAtmospherePass` | `boolean` | `true` | 是否启用大气后处理通道 |
| `enableCloudsPass` | `boolean` | `true` | 是否启用云层后处理通道 |
| `dynamicCloud` | `boolean` | `false` | 是否启用动态云层动画 |

### 预置天气属性

DynamicSky 内置了天气预设，可被 `DynamicWeather` 读取：

| 天气类型 | 太阳光强度 | 天光强度 | 云层覆盖 | 灰度因子 |
|---------|-----------|---------|---------|---------|
| `clear` | 2.5 | 0.1 | 0 | 0 |
| `partlyCloudy` | 2.2 | 0.2 | 0.55 | 0.2 |
| `cloudy` | 0.5 | 0.5 | 0.6 | 0.5 |
| `overcast` | 0 | 0.5 | 0.8 | 0.75 |
| `foggy` | 0 | 0.5 | 0 | 1 |
| `rainy` | 0 | 0.5 | 0 | 1 |
| `snowy` | 0 | 0.5 | 0 | 1 |
| `stormy` | 0 | 0.5 | 0.8 | 1 |
| `thunderstorm` | 0 | 0.3 | 0.3 | 0.5 |

### 示例

```javascript
// 创建动态天空
const sky = engine.add(new mapvthree.DynamicSky());

// 开启动画循环以实现天空动态效果
engine.rendering.enableAnimationLoop = true;
engine.rendering.animationLoopFrameTime = 40;

// 设置时间
sky.time = 3600 * 17.5;
```

---

## 常见场景

### 场景一：高品质户外场景

```javascript
import * as mapvthree from '@baidu/mapv-three';
import * as THREE from 'three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { sky: null },
});

engine.map.lookAt([116, 39], { pitch: 80, range: 1000 });
engine.rendering.enableAnimationLoop = true;
engine.rendering.animationLoopFrameTime = 40;

// 使用动态天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.cloudsCoverage = 0.3;
sky.affectWorld = true; // 天空反射影响场景中的金属/光滑材质

// 设置时间为下午3:30
sky.time = 3600 * 15.5;

// 添加金属球体观察环境反射效果
const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(2, 32, 32),
    new THREE.MeshStandardMaterial({
        color: 0xffffff,
        roughness: 0,
        metalness: 1,
    })
);
sphere.position.set(0, 0, 2);
engine.add(sphere);
```

### 场景二：自定义天空贴图

```javascript
const engine = new mapvthree.Engine(document.getElementById('map_container'));

engine.map.setCenter([0, 0]);
engine.map.setZoom(24);
engine.map.setPitch(80);

// 使用自定义天空贴图
const sky = engine.add(new mapvthree.CustomStaticSky());
sky.textureURL = 'https://example.com/sunset_panorama.jpg';
```

### 场景三：夜景渐变天空

```javascript
const sky = engine.add(new mapvthree.DefaultSky());
// 设置深蓝色夜空渐变
sky.color = new THREE.Color(0x0a0a2e);     // 地平线深蓝
sky.highColor = new THREE.Color(0x000011); // 天顶近黑

// 降低光照模拟夜间
sky.sunLightIntensity = 0;
sky.skyLightIntensity = 0.3;
```

## 注意事项

1. **时间控制**：通过 `sky.time` 设置时间，单位为秒（如 `3600 * 17` 表示下午5点）。
2. **性能考虑**：
   - `EmptySky` / `DefaultSky` / `HazeSky` 性能开销最低，适合移动端。
   - `StaticSky` / `CustomStaticSky` 加载纹理后性能开销低。
   - `DynamicSky` 包含大气散射和体积云，开销较高；可通过 `useVolumetricClouds = false` 关闭体积云。
