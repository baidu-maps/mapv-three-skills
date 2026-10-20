# 风场、流线与 CFD

`WindField` 展示速度场中的运动粒子，或沿给定流线运动。CFD／OpenFOAM 数据先在独立工具侧处理，再交给同一个 WindField，浏览器不运行流体求解器。雨雪雾见 [天气系统](./weather-system.md)。以下代码使用已有 `engine`、`mapvthree`。

## 最小风场

```javascript
const wind = engine.add(new mapvthree.WindField({
    particleCount: 2000, display: 'arrows', trailDuration: 2,
    colorRamp: ['#3b82f6', '#40ddcb', '#ffd24a', '#ff664f'], speedRange: [0, 30],
}));
const data = {
    coordType: 'WGS84', velocityFrame: 'ENU', sampling: 'linear',
    dimensions: [2, 2, 1], timesSeconds: [0],
    points: [
        113.354, 23.124, 100, 113.356, 23.124, 100,
        113.354, 23.126, 100, 113.356, 23.126, 100,
    ],
    velocities: [10, 0, 0, 10, 0, 0, 10, 0, 0, 10, 0, 0],
};
await wind.setData(data);
wind.play({loop: true});
console.log(wind.sample([113.355, 23.125, 100], 0));
```

这里四个节点都在 100 米高，每个速度 `[10,0,0]` 表示向东 10 米／秒。不是方向角、不是经纬度变化量，也不是每秒更新一帧的命令。示例是人工静态场，不代表实际气象。

速度向量表示风的**去向**。气象数据常给"风从哪里来"的风向角 θ（正北为 0、顺时针）和风速 S，换算为 `u = -S·sin θ`、`v = -S·cos θ`。

`setData`／`setStreamlines` 必须在 `engine.add(wind)` 之后调用，`play`／`seek`／`stop`／`sample` 还要求已设置数据，否则抛 `INVALID_STATE`；地图 `coordType` 显式设为非 WGS84 时 add 会抛错。

## 空间与时间格式

| 字段 | 类型 | 要求 |
| --- | --- | --- |
| coordType／velocityFrame | string | 必须声明 `'WGS84'`／`'ENU'` |
| sampling | string | 必须声明 `'linear'` 或 `'nearest'` |
| dimensions | number[3] | X、Y、Z 节点数，正整数，N 为三者乘积 |
| points | 数组／TypedArray | N×3 个数，扁平 `[经度,纬度,椭球高]` |
| velocities | 数组／TypedArray | T×N×3 个数，扁平 `[向东,向北,向上]` 速度，米／秒 |
| timesSeconds | 数组／TypedArray | 显式秒时间轴，严格递增 |
| timeStepSeconds | number | 固定正秒间隔；与 timesSeconds 必须且只能传一个 |
| validMask | 数组／TypedArray，可省略 | T×N 个 0／1，0 表示无效域，不是静风 |

空间顺序 X 最快，再 Y，再 Z；速度时间放最外层，即先放完整第 0 时次，再放第 1 时次。linear 要求每个轴（经度、纬度、高度）都严格递增且等间距，并按 X 最快排列，进行空间和时间插值；由北往南排列的栅格、不等间距的高度层都会被拒绝，这类数据选 nearest。nearest 不做距离阈值，包围盒内离节点很远的点也会取最近节点的速度。validMask 只接受数值 0／1，不接受 true／false。数据调用时复制，原数组变化不会自动修改场。

```javascript
// 同一组节点，三组风速分别对应 0、5、10 秒。
const {timesSeconds, ...fixedData} = data;
await wind.setData({
    ...fixedData, timeStepSeconds: 5,
    velocities: [...data.velocities, ...data.velocities.map(v => v * 2), ...data.velocities],
});
wind.play({loop: true, rate: 1});
```

不等间隔用 timesSeconds:[0,5,30]，配三组完整速度。固定间隔从 0 推导时间轴，仍逐帧插值，不是每 5 秒突变一次。两种时间参数都传／都不传、数组长度不匹配或无效时间会拒绝，不覆盖旧场。

`sample(point,timeSeconds)` 返回 ENU 速度数组或 null；时间超出数据范围钳制。场外或参与插值的节点被 validMask 标为无效时返回 null。单节点轴保持粒子该轴位置，不因该轴速度直接越界。数据最多 600 万速度分量，并不是建议满额使用。

## 显示与控制

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| particleCount | number | 5000 | 构造时决定，1～100000；之后不要修改 |
| lifetime | number[2] | [6,18] | 随机寿命秒范围，每项 >0 且 ≤3600；只能构造时设置 |
| display | string | `'points'` | 点或 `'arrows'` 箭头 |
| particleSize | number | 3 | 点模式 CSS 像素 |
| arrowLength | number[2] | [7,7] | CSS 像素长度；不同上下限按速度映射 |
| trailDuration | number | 0 | 拖尾保留秒数，0 关闭，最大 30 |
| trailSegments | number | 32 | 每粒子历史分段数，1～256 |
| lineWidth | number | 2 | 拖尾线宽，**物理像素**（不乘设备像素比，DPR=2 屏上看起来比 particleSize／arrowLength 细一半） |
| fadeIn／fadeOut | number | 0.5／1 | 渐入／渐出秒数，和不能大于最短寿命 |
| color／opacity | 颜色／number | '#54e7d5'／1 | 基础颜色及透明度 |
| colorRamp／speedRange | 颜色数组／number[2] | null／[0,30] | 2～16 色色带及对应风速范围 |

颜色只接受 3／6 位 hex、CSS 颜色名、整数或 THREE.Color，不支持 `rgb()`、`rgba()` 和 8 位 hex。构造参数和 setStyle 不会拒绝未知字段，拼错的参数会被静默忽略。

箭头头部与拖尾连接，默认 7 像素长，不随地图放大变成长箭杆。拖尾长度由速度和保留时间共同影响，不代表剩余路线点数。粒子越界或走完流线保留旧位置淡出，再重新生成；fadeOut:0 才立即消失。

```javascript
wind.setStyle({display: 'arrows', arrowLength: [7, 7], trailDuration: 2, lineWidth: 1.5});
wind.pause();
wind.seek(5);
wind.play({loop: true, rate: 1});
// 结束播放时调用：停止并重置时间。
wind.stop();
```

setStyle 部分更新，保留播放状态与粒子位置，拖尾需重新积累。play 默认 loop:false、rate:1（范围 (0,100]）；pause 冻结；stop 和 seek 都会重新随机播种全部粒子并清空拖尾，stop 还把时间重置到起点。每次 setData／setStreamlines 都会把 state 置回 `'idle'` 并重新播种，需要再调 play；它不适合高频实时推送整场数据。`state`（idle／running／paused／stopped／completed／disposed）、`mode`（field／streamlines）、`timeSeconds` 可读。setData／setStreamlines 支持第二参数 signal。

拖尾开启时 `particleCount * (trailSegments + 1) <= 500000`。性能不足先减少粒子和历史段数；不能把点数上限当作稳定帧率承诺。

## 给定流线与转换工具

```javascript
await wind.setStreamlines([{
    points: [[113.354, 23.124, 100], [113.355, 23.125, 110], [113.356, 23.125, 120]],
    colors: ['#3b82f6', '#40ddcb', '#ff664f'],
}], {speed: 15});
wind.play();
```

setStreamlines 的 speed 默认 10 米／秒（范围 (0,10000]），是展示沿线运动速度；colors 可省略，提供时与每个点一一对应，并优先于速度色带。流线数组不能为空，最多 1 万条、总计 10 万点，纬度 ±85°，相邻点不能重合。此模式循环沿线路径，不是完整空间速度场，不能用 sample 查询任意位置。粒子按序号轮流分配到各条线，粒子数少于线数时部分线上没有粒子。

有带秒时间戳的轨迹并希望近似转换为规则场时，才用 `windFieldFromStreamlines`：

```javascript
const field = mapvthree.windFieldFromStreamlines([{
    points: [[113.354, 23.124, 100], [113.355, 23.125, 110], [113.356, 23.125, 120]],
    timesSeconds: [0, 10, 20],
}], {dimensions: [8, 8, 3], timesSeconds: [0, 10, 20]});
await wind.setData(field);
wind.play({loop: true});
```

该转换对每条轨迹只取当前时刻所在的那一段估算速度，再给每个网格节点取最近的一条轨迹，不能恢复真实 CFD 原始体场；只有一条轨迹时，同一时刻整个网格速度相同。默认 dimensions [12,12,1]、timesSeconds [0]，输出为 TypedArray，需要存成文件时先转普通数组。另有高度梯度展示场：

```javascript
const heightField = mapvthree.windFieldFromHeightData({
    dimensions: [2, 2], bounds: [113.354, 23.124, 113.356, 23.126],
    heights: [100, 110, 100, 110], velocity: [5, 0],
});
await wind.setData(heightField);
wind.play({loop: true});
```

dimensions 是列／行数，bounds 是西／南／东／北，heights 为米制高度，按 X（列）最快排列，**第 0 行是南边**（常见图片栅格第 0 行是北边，需先上下翻转）；velocity 为东／北米每秒（默认 [5,0]）。输出 nearest 节点场，竖向速度由高度梯度估算，节点都在地表高度上，实质是 2.5D，亦不属于气象预测。`createWindFieldTestData` 不在开放版入口导出，不要生成依赖它的通用接入代码。

## OpenFOAM 后处理与接入

CFD 是流体数值计算；OpenFOAM 案例包含体网格、时间目录以及速度 U 等字段。这里处理的是已经算好的数据，不负责求解、飞行风险判定或航线规划。

1. 在配套引擎源码的 `scripts/cfd/` 工具中注册案例，确认模型原点、方向和单位。
2. 使用 Python＋VTK 提取规则风场或指定种子的流线，生成 JSON，或调用配套任务服务。
3. 网页按输出类型交给 setData 或 setStreamlines。

工具不包含在浏览器引擎导出中；使用 npm 包的用户需要另行获得／部署配套工具，不存在 `mapvthree.OpenFOAM` API。安装和完整服务配置以配套源码 `scripts/cfd/README.md` 为准。

注册格式示例，file 相对注册 JSON 所在目录：

```json
{"cases":{"district":{"file":"cases/district/district.foam","origin":[113.3552,23.1252,20],"metersPerUnit":1,"rotation":[[1,0,0],[0,1,0],[0,0,1]]}}}
```

origin 是模型零点的 WGS84 经纬高；rotation 将模型 XYZ 转为当地 ENU。metersPerUnit 仅缩放几何，速度字段须为物理米／秒，不再次乘几何比例。当前支持已重构的单区域内部体网格，分区数据先 reconstructPar，多区域分别导出。

独立处理请求示例：

```json
{"velocityField":"U","timesSeconds":[0,5,30],"dimensions":[20,20,6]}
```

dimensions 每轴 2～256，默认 [20,20,6]；timesSeconds 默认取案例全部时次，必须是案例中真实存在的时次；可选 `bounds:{west,south,east,north,bottomHeight,topHeight}` 指定采样范围，默认取网格范围。速度字段必须声明速度量纲 `[0 1 -1 0 0 0 0]`，否则拒绝。

流线请求：

```json
{"velocityField":"U","timeSeconds":5,"seeds":{"type":"line","start":[113.3538,23.1242,70],"end":[113.3538,23.1262,70],"count":30},"direction":"both","maxLength":1000,"colorBy":{"field":"U","component":"magnitude","range":[0,20],"colors":["#3b82f6","#40ddcb","#ff664f"]}}
```

时间必须为案例中真实存在的时次；种子要在有效流体域内。种子支持 `type:'line'`（count 2～128，默认 30）或 `type:'points'`（1～128 个点）。direction 支持 forward／backward／both，maxLength 为每方向米制长度（0.001～100000）；colorBy 可按向量 magnitude、向量分量整数索引，或标量 component:0 着色，colors 为 2～32 个 `#RRGGBB`。字段 p 可能不是 Pa，必须看量纲。

命令行入口（在引擎源码根目录执行，`python -m scripts.cfd --help` 查看）：

```bash
python -m scripts.cfd sample ./cfd-sample                 # 生成测试案例
python -m scripts.cfd inspect --registry reg.json --case district
python -m scripts.cfd wind-field --registry reg.json --case district --request wind.json --output wind-out.json
python -m scripts.cfd streamlines --registry reg.json --case district --request lines.json --output lines-out.json
python -m scripts.cfd serve --registry reg.json --port 8766 --origin http://localhost:5173
```

`--output` 不会覆盖已有文件。`serve` 默认端口 8766；浏览器页面跨域访问时必须用 `--origin` 声明页面来源（可重复），否则返回 403。

离线生成文件后，从自己的网站提供文件。流线结果在种子不在流体域内时可能是空数组，而 setStreamlines 不接受空数组，需要先判断：

```javascript
async function readJSON(url) {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
}
await wind.setData(await readJSON('/data/wind.json'));
wind.play({loop: true});
// 需要改为固定时刻流线展示时使用。
const result = await readJSON('/data/lines.json');
if (result.lines.length) {
    await wind.setStreamlines(result.lines, {speed: 15});
    wind.play();
}
else {
    console.warn('没有生成流线，请检查种子是否在流体域内');
}
```

| 配套服务请求 | 返回 |
| --- | --- |
| GET /api/cfd/cases | 案例 ID 列表 |
| GET /api/cfd/cases/:id | 时次、字段、范围、原点 |
| POST /api/cfd/cases/:id/wind-field 或 streamlines | 202，任务 id |
| GET /api/cfd/jobs/:id | running／completed／failed／cancelled；完成带 result |
| DELETE /api/cfd/jobs/:id | 取消运行任务或删除结束记录 |

POST 不直接返回风场，应轮询任务，完成后应用 result，最后删除记录；页面取消需取消后端任务。风场结果直接传 setData，流线结果传 result.lines（可能为空数组，见上）。配套服务固定只监听 127.0.0.1（不可配置），不自带认证、上传或多人资源隔离：不带 Origin 头的本机请求（如 curl）不受 `--origin` 白名单限制。服务同时最多 2 个运行中任务，超出返回 429，`GET /api/cfd/cases/:id` 读取元数据也占用一个任务槽；请求体上限 1 MiB，单任务超时 120 秒；元数据读取失败返回 422，来源或 Host 不允许返回 403。

风场工具上限为 10 万节点、32 帧、600 万速度分量；流线上限 128 个种子、每方向 350 步、总计 10 万点。有效域以 validMask 保留，0 值不能删掉冒充有效静风。不同积分器不保证逐点数值一致。

临时移出停止运行时监听，可再次加入；移出期间除 pause 和 setStyle 外的调用都会抛 `INVALID_STATE`。永久结束调用 `wind.dispose()`。

错误对象带 `code`：INVALID_ARGUMENT、INVALID_STATE、DISPOSED、CANCELLED、SUPERSEDED、OBJECT_IN_USE、BUDGET_EXCEEDED。
