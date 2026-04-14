# 设备状态控制

通过 `AssetsScene.handleActiveDevice()` 控制资产场景中各类设备的状态，如情报板显示文字、信号灯切换颜色、路灯开关、风机旋转、卷帘门升降等。

## 调用时机

`handleActiveDevice` **必须在设备模型加载完成后调用**。AssetsScene 按区域动态加载模型，涉及两个事件：

- **`assetsLibModelLoaded`** — 初始化时加载的模型（只触发一次）
- **`sceneViewAdd`** — 相机移动后新加载的模型（每次区域加载触发）

两个事件的回调参数都是 `{ models }`，需要**同时监听**才能覆盖所有模型。

**正确写法：**

```javascript
// 创建资产场景
const assetsScene = engine.add(
    new trafficAddon.AssetsScene({
        projectId: 'xxx',
        host: 'http://10.24.22.223',
        renderOption: {
            anchor: 'cameraPosition',
            radius: 300,
            ceilRender: true,
            useDefault3DTileLayer: false,
            useDefaultLodTileLayer: false,
        },
        auth: { ak: 'your_ak' },
    })
);

// 统一的设备状态设置函数
// model.traverse 遍历子节点，通过 child.modelType 匹配设备类型，用 child.uuid 调用
function setupDevices(models) {
    models.forEach(model => {
        model.traverse(child => {
            // 匹配情报板（通过 modelType 或 child.uuid）
            if (child.uuid === '5876401309329959082') {
                assetsScene.handleActiveDevice(child.uuid, 'guidanceScreen', {
                    text: '前方施工，注意减速慢行',
                    fontColor: '#0f0',
                    fontSize: 16,
                    typeFace: 'Songti SC',
                    layout: 'center',
                    screenLength: 300,
                    screenHeight: 48,
                    emissiveIntensity: 0.3,
                });
            }
        });
    });
}

// 初始加载的模型
assetsScene.addEventListener('assetsLibModelLoaded', e => {
    setupDevices(e.models);
});

// 相机移动后新加载的模型
assetsScene.addEventListener('sceneViewAdd', e => {
    setupDevices(e.models);
});
```

**错误写法：**

```javascript
// ❌ 错误1：创建后立即调用，模型还没加载
assetsScene.handleActiveDevice('xxx', 'guidanceScreen', { text: '...' });

// ❌ 错误2：只监听 assetsLibModelLoaded，相机移动后新加载的模型不会被设置
assetsScene.addEventListener('assetsLibModelLoaded', e => { ... });
```

## API

```javascript
assetsScene.handleActiveDevice(modelInfo, type, options)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `modelInfo` | `string \| Object3D` | 子设备 UUID（`child.uuid`）或模型对象。传字符串时内部通过 `engine.scene.getObjectByName()` 查找。需要先通过 `model.traverse()` 遍历找到目标子设备 |
| `type` | `string` | 设备类型 |
| `options` | `object` | 设备控制参数（各类型不同） |

## 设备类型总览

| type | 说明 | 控制方式 |
|------|------|---------|
| `guidanceScreen` | 情报板/诱导屏 | 文字贴图渲染 |
| `signalLamp` | 信号灯 | 灯组位状态切换 |
| `streetlight` | 路灯 | 发光材质开关 |
| `fan` | 风机 | 正转/反转动画 |
| `rollingShutterDoor` | 卷帘门 | 帧动画开关 |
| `liftRod` | 升降杆 | 帧动画升降 |
| `tunnelIndicatorLight` | 隧道指示灯 | 灯组位状态切换 |
| `fireDoor` | 防火门 | 左右门帧动画 |

---

## 1. 情报板 guidanceScreen

在情报板上渲染文字内容，支持多行、滚动、自定义字体颜色等。

### options 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `text` | `string` | `''` | 显示文字，用 `\\` 分隔多行 |
| `fontColor` | `string` | `'#0f0'` | 字体颜色 |
| `fontSize` | `number` | `16` | 字体大小 |
| `typeFace` | `string` | `'Songti SC'` | 字体 |
| `layout` | `string` | `'center'` | 对齐方式 |
| `overflow` | `string` | `'scroll'` | 溢出处理：`'scroll'` 滚动 |
| `scrollSpeed` | `number` | `0.001` | 滚动速度 |
| `scrollTime` | `number` | `2000` | 滚动间隔（ms） |
| `screenLength` | `number` | - | 屏幕宽度（可选） |
| `screenHeight` | `number` | - | 屏幕高度（可选） |
| `status` | `number` | - | 0=关闭（清空文字），其他=显示 |

### 注意事项

- **必须传 `screenLength` 和 `screenHeight`**：这两个参数指定情报板画布的像素尺寸，不传会导致文字不显示或挤在一起。横向情报板推荐 `screenLength: 300, screenHeight: 48`（宽高比约 6:1）。
- **换行符**：`\\` 是换行符，如需多行显示可在文字中插入 `\\`。但不要自动给用户的文字加换行，按用户原文传入即可。
- **`emissiveIntensity`**：控制发光强度，推荐 `0.3`。

### 示例

```javascript
// 单行文字
assetsScene.handleActiveDevice('8406824597308799088', 'guidanceScreen', {
    text: '前方施工，注意减速慢行',
    fontColor: '#0f0',
    fontSize: 16,
    typeFace: 'Songti SC',
    layout: 'center',
    screenLength: 300,
    screenHeight: 48,
    emissiveIntensity: 0.3,
});

// 多行文字（用 \\ 换行）
assetsScene.handleActiveDevice('8406824597308799088', 'guidanceScreen', {
    text: '事故多发路段\\请保持车距',
    fontColor: '#f00',
    fontSize: 16,
    layout: 'center',
    screenLength: 300,
    screenHeight: 48,
    emissiveIntensity: 0.3,
});

// 滚动文字（文字较长时用 overflow: 'scroll'）
assetsScene.handleActiveDevice('8406824597308799088', 'guidanceScreen', {
    text: '遇事故，车靠边，人撤离，即报警。',
    fontColor: '#0f0',
    fontSize: 16,
    overflow: 'scroll',
    scrollSpeed: 0.001,
    scrollTime: 2000,
});

// 清空情报板
assetsScene.handleActiveDevice('8406824597308799088', 'guidanceScreen', {
    text: '',
    status: 0,
});
```

---

## 2. 信号灯 signalLamp

控制交通信号灯的灯组状态，通过 `signalLampBitStatus` 数组设置每个灯组的颜色、方向和倒计时。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `signalLampBitStatus` | `number[]` | 灯组状态数组，每个元素对应一个灯头的状态编码，数组长度应与模型灯头数一致 |

### 编码规则

每个数组元素是一个数字，根据数值范围分三种类型：

**纯色圆灯（值 < 100）— 格式 `CC`**

| 编码 | 含义 |
|------|------|
| `10` | 红灯 |
| `20` | 黄灯 |
| `30` | 绿灯 |
| `40` | 灰灯（熄灭） |

**数字倒计时（值 1000 ~ 99999）— 格式 `CCYYY`**

CC = 颜色码，YYY = 倒计时数字（显示两位）

| 编码 | 含义 |
|------|------|
| `1030` | 红色 30 |
| `2015` | 黄色 15 |
| `3045` | 绿色 45 |
| `4000` | 灰色 00 |

**箭头灯（值 > 99999）— 格式 `CC00ZZ`**

CC = 颜色码，00 = 固定占位，ZZ = 方向码

| ZZ 方向码 | 含义 |
|----------|------|
| `01` | 掉头 |
| `02` | 左转 |
| `03` | 左转待行 |
| `04` | 直行 |
| `05` | 直行待行 |
| `06` | 右转 |
| `11` | 禁行 |
| `12` | 可通行 |

**颜色 + 方向组合速查：**

| 编码 | 含义 | 编码 | 含义 |
|------|------|------|------|
| `100002` | 红色左转 | `300002` | 绿色左转 |
| `100004` | 红色直行 | `300004` | 绿色直行 |
| `100006` | 红色右转 | `300006` | 绿色右转 |
| `100011` | 红色禁行 | `300012` | 绿色通行 |
| `200002` | 黄色左转 | `400011` | 灰色禁行（熄灭） |

### 模型灯组配置

模型的 `deviceType`（格式 `3-subtype-subtype2`）决定灯头数量和类型：

| deviceType | lightList | 说明 |
|------------|-----------|------|
| `3-201-0` | `[10, 20, 30]` | 三灯纯色竖排 |
| `3-201-1` | `[10, 20, 30]` | 三灯纯色横排 |
| `3-201-2` | `[100000, 200000, 300000]` | 三灯箭头竖排 |
| `3-201-3` | `[100000, 200000, 300000]` | 三灯箭头横排 |
| `3-201-4` | `[10, 1000, 300000]` | 纯色 + 数字 + 箭头 |
| `3-201-10` | `[10, 10, 10, 10]` | 四灯纯色横排 |
| `3-202-0` | `[999999]` | 单灯 |
| `3-205-0` | `[999999, 1000]` | 通用 + 数字 |
| `3-205-1` | `[100000, 1000, 300000]` | 箭头 + 数字 + 箭头 |

`signalLampBitStatus` 数组长度应与 `lightList` 长度一致，不一致时控制台会打印警告。

### 示例

```javascript
// 三灯信号灯：绿色直行 + 灰 + 灰
assetsScene.handleActiveDevice('信号灯UUID', 'signalLamp', {
    signalLampBitStatus: [300004, 40, 40],
});

// 红黄绿三灯
assetsScene.handleActiveDevice('信号灯UUID', 'signalLamp', {
    signalLampBitStatus: [10, 20, 30],
});

// 四灯横排：红色左转亮，其余灰
assetsScene.handleActiveDevice('信号灯UUID', 'signalLamp', {
    signalLampBitStatus: [100002, 40, 40, 40],
});

// 带数字倒计时（模型需支持，如 3-205-0）
assetsScene.handleActiveDevice('信号灯UUID', 'signalLamp', {
    signalLampBitStatus: [300006, 3025],  // 绿色右转 + 绿色倒计时25秒
});

// 全部熄灭
assetsScene.handleActiveDevice('信号灯UUID', 'signalLamp', {
    signalLampBitStatus: [40, 40, 40],
});
```

---

## 3. 路灯 streetlight

控制路灯的开关状态，通过发光材质（emissive）实现。

### options 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `status` | `boolean` | - | `true`=开灯，`false`=关灯 |
| `emissiveColor` | `string` | `'#fff'` | 发光颜色 |
| `emissiveIntensity` | `number` | `1` | 发光强度 |

### 示例

```javascript
// 开灯（白色）
assetsScene.handleActiveDevice('路灯UUID', 'streetlight', {
    status: true,
});

// 开灯（暖黄色，高强度）
assetsScene.handleActiveDevice('路灯UUID', 'streetlight', {
    status: true,
    emissiveColor: '#ffcc00',
    emissiveIntensity: 2,
});

// 关灯
assetsScene.handleActiveDevice('路灯UUID', 'streetlight', {
    status: false,
});
```

---

## 4. 风机 fan

控制风机的旋转动画，支持正转和反转。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `string \| boolean` | `'right'`=正转，`'left'`=反转，`false`=停止 |

### 示例

```javascript
// 正转
assetsScene.handleActiveDevice('风机UUID', 'fan', {
    status: 'right',
});

// 反转
assetsScene.handleActiveDevice('风机UUID', 'fan', {
    status: 'left',
});

// 停止
assetsScene.handleActiveDevice('风机UUID', 'fan', {
    status: false,
});
```

---

## 5. 卷帘门 rollingShutterDoor

控制卷帘门的开关，通过帧动画实现。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `string` | `'open'`=打开，`'close'`=关闭 |

### 示例

```javascript
// 打开卷帘门
assetsScene.handleActiveDevice('卷帘门UUID', 'rollingShutterDoor', {
    status: 'open',
});

// 关闭卷帘门
assetsScene.handleActiveDevice('卷帘门UUID', 'rollingShutterDoor', {
    status: 'close',
});
```

---

## 6. 升降杆 liftRod

控制升降杆的升降，通过帧动画实现。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `string` | `'open'`=升起，`'close'`=降下 |

### 示例

```javascript
// 升起
assetsScene.handleActiveDevice('升降杆UUID', 'liftRod', {
    status: 'open',
});

// 降下
assetsScene.handleActiveDevice('升降杆UUID', 'liftRod', {
    status: 'close',
});
```

---

## 7. 隧道指示灯 tunnelIndicatorLight

控制隧道内指示灯的状态。与 signalLamp 的区别：数组中每个元素分别对应独立的 `img1`、`img2`... 子网格（每个灯面板独立），而 signalLamp 是所有灯组渲染在同一个子网格上。

编码规则与 signalLamp 完全一致（见上方编码规则章节）。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `signalLampBitStatus` | `number[]` | 各灯面板的状态编码，每个元素对应一个独立面板（img1, img2...） |

### 模型配置

| deviceType | lightList | 说明 |
|------------|-----------|------|
| `3-203-0` | `[999999]` | 单面板 |
| `3-203-1` | `[999999, 999999, 999999, 999999]` | 四面板 |

### 示例

```javascript
// 正面绿色通行，反面红色禁行
assetsScene.handleActiveDevice('隧道灯UUID', 'tunnelIndicatorLight', {
    signalLampBitStatus: [300012, 100011],
});

// 四面板：全部绿色通行
assetsScene.handleActiveDevice('隧道灯UUID', 'tunnelIndicatorLight', {
    signalLampBitStatus: [300012, 300012, 300012, 300012],
});

// 全部熄灭
assetsScene.handleActiveDevice('隧道灯UUID', 'tunnelIndicatorLight', {
    signalLampBitStatus: [400011, 400011],
});
```

---

## 8. 防火门 fireDoor

控制防火门的开关，支持左右门独立控制。

### options 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `string` | `'open'`=全开，`'close'`=全关，`'rightOpen'`=右门开，`'rightClose'`=右门关，`'leftOpen'`=左门开，`'leftClose'`=左门关 |

### 示例

```javascript
// 全部打开
assetsScene.handleActiveDevice('防火门UUID', 'fireDoor', {
    status: 'open',
});

// 全部关闭
assetsScene.handleActiveDevice('防火门UUID', 'fireDoor', {
    status: 'close',
});

// 只开右门
assetsScene.handleActiveDevice('防火门UUID', 'fireDoor', {
    status: 'rightOpen',
});

// 只关左门
assetsScene.handleActiveDevice('防火门UUID', 'fireDoor', {
    status: 'leftClose',
});
```

---

## 批量控制

复用上方"正确写法"中的 `setupDevices` 函数，在两个事件中批量设置设备状态：

```javascript
const screenConfigs = [
    { uuid: '692302028113879040', text: '遇事故，车靠边，人撤离，即报警。' },
    { uuid: '692302028113879041', text: '限速80km/h' },
];

function setupDevices(models) {
    models.forEach(model => {
        model.traverse(child => {
            // 匹配情报板
            const screenConfig = screenConfigs.find(s => s.uuid === child.uuid);
            if (screenConfig) {
                assetsScene.handleActiveDevice(child.uuid, 'guidanceScreen', {
                    text: screenConfig.text,
                    fontColor: '#0f0',
                    fontSize: 16,
                    typeFace: 'Songti SC',
                    layout: 'center',
                    screenLength: 300,
                    screenHeight: 48,
                    emissiveIntensity: 0.3,
                });
            }

            // 匹配路灯并开灯
            if (child.modelType === '路灯classType') {
                assetsScene.handleActiveDevice(child.uuid, 'streetlight', { status: true });
            }
        });
    });
}

// 必须同时监听两个事件
assetsScene.addEventListener('assetsLibModelLoaded', e => {
    setupDevices(e.models);
});
assetsScene.addEventListener('sceneViewAdd', e => {
    setupDevices(e.models);
});
```
