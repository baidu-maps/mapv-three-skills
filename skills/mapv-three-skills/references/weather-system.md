# 天气系统

mapv-three 的天气系统通过 `DynamicWeather` 类实现，提供雨、雪、雾、雷暴等天气效果，并支持天气之间的平滑过渡。天气系统需要搭配天空系统（通常为 `DynamicSky`）使用。

## 快速开始

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { sky: null },
});

engine.map.lookAt([116, 39], { pitch: 80, range: 1000 });
engine.rendering.enableAnimationLoop = true;

// 第一步：创建动态天空
const sky = engine.add(new mapvthree.DynamicSky());

// 第二步：创建天气系统，传入天空实例
const weather = engine.add(new mapvthree.DynamicWeather(sky));

// 第三步：设置天气类型
weather.weather = 'rainy'; // 下雨
```

## DynamicWeather 构造函数

```javascript
const weather = new mapvthree.DynamicWeather(sky);
engine.add(weather);
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sky` | `DynamicSky` \| `EmptySky` | 是 | 关联的天空实例，天气系统将修改其光照和云层属性 |

## 支持的天气类型

| 天气类型 | 标识 | 粒子效果 | 说明 |
|---------|------|---------|------|
| 晴天 | `clear` | 无 | 强太阳光，几乎无云 |
| 局部多云 | `partlyCloudy` | 无 | 正常光照，少量云层 |
| 多云 | `cloudy` | 无 | 减弱太阳光，增加云层覆盖 |
| 阴天 | `overcast` | 无 | 无太阳光，厚云层覆盖 |
| 雾天 | `foggy` | 无 | 无太阳光，天空灰白 |
| 雨天 | `rainy` | 雨滴粒子 | 无太阳光，雨滴效果，密度 1.0 |
| 雪天 | `snowy` | 雪花粒子 | 无太阳光，飘雪效果 |
| 暴雨 | `stormy` | 大雨粒子 | 无太阳光，高密度雨滴（密度 4.0），厚云层 |
| 雷暴 | `thunderstorm` | 雨滴 + 闪电 | 雨滴 + 闪电效果，天空半灰 |

## 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `weather` | `string` | `'partlyCloudy'` | 当前天气类型，设置后自动触发天气切换 |
| `transitionDuration` | `number` | `1000` | 天气过渡动画时长（毫秒） |

## 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `addWeatherChangedListener(listener)` | `listener: (weather: string) => void` | 添加天气变化监听器 |
| `removeWeatherChangedListener(listener)` | `listener: Function` | 移除天气变化监听器 |

## 天气效果详解

### 雨天效果 (rainy)

雨天使用基于 GPU 粒子系统的雨滴效果，雨滴会跟随相机位置移动。

```javascript
const sky = engine.add(new mapvthree.DynamicSky());
const weather = engine.add(new mapvthree.DynamicWeather(sky));
weather.weather = 'rainy';
```

雨天效果包含：
- 雨滴粒子效果（速度朝向型粒子）
- 太阳光关闭，天空光增强
- 天空变灰（mixGrayFactor = 1）

### 雪天效果 (snowy)

雪天使用雪花粒子系统，雪花缓慢飘落。

```javascript
weather.weather = 'snowy';
```

雪天效果包含：
- 雪花粒子效果
- 太阳光关闭，天空光增强
- 天空变灰

### 暴雨效果 (stormy)

暴雨在雨天基础上增加雨滴密度，并增加厚云层覆盖。

```javascript
weather.weather = 'stormy';
```

暴雨效果包含：
- 高密度雨滴粒子（密度 4.0，普通雨天为 1.0）
- 厚云层覆盖（cloudsCoverage = 0.8）
- 天空完全变灰

### 雷暴效果 (thunderstorm)

雷暴在雨天基础上叠加闪电效果，闪电为 Billboard 平面，使用专用闪电材质渲染。

```javascript
weather.weather = 'thunderstorm';
```

雷暴效果包含：
- 雨滴粒子（密度 2.0）
- 随机间隔闪电效果
- 闪电闪烁和发光

## 常见场景

### 场景一：基础天气切换

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { sky: null },
});

engine.map.lookAt([116, 39], { pitch: 80, range: 1000 });
engine.rendering.enableAnimationLoop = true;

// 创建天空和天气系统
const sky = engine.add(new mapvthree.DynamicSky());
const weather = engine.add(new mapvthree.DynamicWeather(sky));

// 初始天气为局部多云
weather.weather = 'partlyCloudy';

// 设置过渡时间为 2 秒
weather.transitionDuration = 2000;

// 监听天气变化
weather.addWeatherChangedListener((weatherType) => {
    console.log('天气已切换为:', weatherType);
});

// 按钮切换天气
document.getElementById('btn-rain').addEventListener('click', () => {
    weather.weather = 'rainy';
});
document.getElementById('btn-snow').addEventListener('click', () => {
    weather.weather = 'snowy';
});
document.getElementById('btn-clear').addEventListener('click', () => {
    weather.weather = 'clear';
});
```

### 场景二：雷暴场景

```javascript
import * as mapvthree from '@baidu/mapv-three';

const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: {
        features: {
            bloom: {
                enabled: true,     // 开启泛光增强闪电效果
                strength: 2.0,
                radius: 0.8,
            },
        },
        enableAnimationLoop: true,
        sky: null,
    },
});

engine.map.lookAt([116, 39], { pitch: 80, range: 1000 });

// 创建动态天空
const sky = engine.add(new mapvthree.DynamicSky());
sky.time = 3600 * 16; // 下午4点

// 创建雷暴天气
const weather = engine.add(new mapvthree.DynamicWeather(sky));
weather.weather = 'thunderstorm';
```

### 场景三：天气循环演示

```javascript
const sky = engine.add(new mapvthree.DynamicSky());
const weather = engine.add(new mapvthree.DynamicWeather(sky));
weather.transitionDuration = 3000; // 3秒过渡

// 天气列表循环切换
const weatherList = ['clear', 'partlyCloudy', 'cloudy', 'rainy', 'snowy', 'thunderstorm'];
let index = 0;

setInterval(() => {
    index = (index + 1) % weatherList.length;
    weather.weather = weatherList[index];
    console.log('切换天气:', weatherList[index]);
}, 8000); // 每8秒切换一次
```

### 场景四：配合静态天空使用

```javascript
// DynamicWeather 也可以搭配 StaticSky 使用
// StaticSky 会根据天气状态自动切换纹理贴图
const engine = new mapvthree.Engine(document.getElementById('map_container'), {
    rendering: { skyType: 'static' },
});
engine.rendering.enableAnimationLoop = true;

const sky = engine.rendering.sky;
const weather = engine.add(new mapvthree.DynamicWeather(sky));
weather.weather = 'rainy';
```

### 场景五：移除天气系统

```javascript
const sky = engine.add(new mapvthree.DynamicSky());
const weather = engine.add(new mapvthree.DynamicWeather(sky));
weather.weather = 'snowy';

// 移除天气效果（光照和云层参数会恢复默认值）
engine.remove(weather);

// 移除天空
engine.remove(sky);
```

## 天气过渡机制

`DynamicWeather` 切换天气时会自动执行平滑过渡：

1. 读取旧天气和新天气的预设属性（`sunLightIntensity`、`skyLightIntensity`、`cloudsCoverage`、`mixGrayFactor` 等）。
2. 对数值类属性使用线性插值（lerp）进行平滑过渡。
3. 过渡期间每帧更新属性值，直到 `transitionDuration` 结束。
4. 粒子效果（雨/雪/雷暴）会在天气切换时立即创建或销毁。

## 注意事项

1. **必须搭配天空使用**：`DynamicWeather` 构造时必须传入天空实例，天气系统通过修改天空的属性（光照、云层等）来实现天气效果。
2. **动画循环**：天气效果（粒子、闪电）需要动画循环支持：
   ```javascript
   engine.rendering.enableAnimationLoop = true;
   ```
3. **性能影响**：雨雪粒子和闪电效果会增加 GPU 开销，暴雨（stormy）的粒子密度最高。
4. **天气重复设置**：设置相同的天气类型不会触发过渡，直接被忽略。
5. **移除清理**：调用 `engine.remove(weather)` 时会自动恢复天空光照参数为默认值，并清除雾效果。
6. **过渡时间**：默认 `transitionDuration` 为 1000ms（1秒），设置为 0 则立即切换无过渡动画。
7. **地球模式兼容**：雨雪粒子在地球模式下会自动根据相机位置旋转，确保粒子方向正确。
8. **雷暴闪电**：闪电效果当前不支持 3D 地球模式，仅在平面模式下效果最佳。
