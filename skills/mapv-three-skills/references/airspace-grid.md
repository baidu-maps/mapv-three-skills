# 空域网格

`UAVGrid` 与 `BeiDouGrid` 共用区域绘制、样式、选择和跟随能力，但编码规则不同。当前 UAVGrid 编码增加 `U` 前缀，与北斗码互斥；解码器拒绝对方的编码。业务应保存网格类型，外部无前缀的民航编码不能直接当作当前 UAVGrid 码使用。以下示例使用已有 `engine` 和 `mapvthree`，初始化见 [引擎核心](./engine-bindmap.md)。

| 能力 | UAVGrid | BeiDouGrid |
| --- | --- | --- |
| 规则 | 民航空域网格规则 | 北斗二维／三维网格规则 |
| 水平精度 | planeLevel，1～16，默认 11 | level，1～10，默认 7 |
| 高度精度 | heightLevel，1～15，默认 10 | 由 level 决定 |
| 显示维度 | 三维；另有独立二维编码工具 | dimension:2 或 3，默认 3 |
| 高度域 | [-512,15872) 米，层厚 16384／2^heightLevel 米（h=10 为 16 米） | 固定高度域约 [-6302 km, 528680 km)，与层级无关；层厚随层级变细，近地面 L1 约 457 km、L7 约 7.7 m |
| 航线／多边形覆盖 | 无专用入口 | setPath／setPolygon |

各级水平格尺寸（经差×纬差，数值来自源码刻度表）：

| 级别 | UAVGrid | BeiDouGrid（GB/T 39409） |
| --- | --- | --- |
| 1 | 6°×4° | 6°×4° |
| 2 | 3°×2° | 30′×30′ |
| 3 | 30′×30′ | 15′×10′ |
| 4 | 15′×10′ | 1′×1′ |
| 5 | 5′×5′ | 4″×4″ |
| 6 | 1′×1′ | 2″×2″ |
| 7 | 12″×12″ | 1/4″ |
| 8 | 4″×4″ | 1/32″ |
| 9～16 | 2″、1″、1/2″、1/4″、1/8″、1/16″、1/32″、1/64″ | L9 1/256″、L10 1/2048″（北斗只到 10 级） |

码长：北斗二维码 L1～L10 依次为 4、6、7、9、11、12、14、16、18、20 位，三维码在此基础上交叉加入高度码（L10 共 32 位）。UAV 平面码为 `U` 加 4～22 位，三维码再接 `-` 和 heightLevel 位二进制高度码。

## 坐标与编码

编码输入是 CGCS2000 经纬度及对应高度。CGCS2000 不是 GCJ-02；Web Mercator 是投影，也不是另一套经纬度偏移算法。

`allowApproximateDatum` 为 boolean，默认 **true**：显示时把 CGCS2000 经纬高近似当作 WGS84 输入引擎投影，不进行精密大地基准转换。设为 false 会拒绝需要近似的场景，不会自动启用高精度转换。它不允许直接把 GCJ-02、BD-09 或海拔值当成网格输入。

```javascript
const plane = mapvthree.UAVGrid.encode2D([113.3552, 23.1252], 11);
const volume = mapvthree.UAVGrid.encode3D([113.3552, 23.1252, 100], {
    planeLevel: 11, heightLevel: 10,
});
console.log(plane.code, mapvthree.UAVGrid.decode2D(plane.code));
console.log(volume.code, mapvthree.UAVGrid.decode3D(volume.code));

const beidou2D = mapvthree.BeiDouGrid.encode2D([113.3552, 23.1252], 6);
const beidou3D = mapvthree.BeiDouGrid.encode3D([113.3552, 23.1252, 100], 6);
console.log(mapvthree.BeiDouGrid.decode2D(beidou2D.code));
console.log(mapvthree.BeiDouGrid.decode3D(beidou3D.code));
console.log(mapvthree.BeiDouGrid.neighbor(beidou3D.code, {east: 1, up: 1}));
```

encode 返回 CellInfo 对象，字符串取 `.code`，decode 接收该字符串。二维编码不含高度；三维即使输入高度 0，也不等于二维编码。CellInfo 提供水平／高度边界（west、south、east、north、bottomHeight、topHeight）、中心和实际级别；`dimension` 字段只有北斗二维／三维结果和 UAV 二维结果才有，UAV 三维结果没有该字段，不要用 `cell.dimension === 3` 判断。二维结果中的零高度字段不代表三维第零层。旧 encodeCell／decodeCell 已删除。

两个类的静态方法参数形态不同：`UAVGrid.encode3D(point, {planeLevel, heightLevel})` 传对象，`BeiDouGrid.encode3D(point, level)` 传数字；`neighbor` 只有 BeiDouGrid 提供（offset 须为安全整数，east 跨日界线环绕，north 不环绕，二维码的 up 必须为 0）。

北斗三维采用交叉组合格式，不兼容旧的“二维编码＋高度码”直接拼接方式。外部平台编码应先确认规范、维度和级别；不要只凭字符串能被解析判断互通。PBF 只是容器格式，这两个类不自动读取任意 PBF 网格服务。

## 创建区域并按风险着色

```javascript
const grid = engine.add(new mapvthree.UAVGrid({
    style: {lineColor: '#40ddcb', lineOpacity: 1, gridFillOpacity: 0.08},
}));
grid.addEventListener('cellselected', event => console.log(event.cell));
const result = await grid.createGrid({
    west: 113.354, south: 23.124, east: 113.356, north: 23.126,
    bottomHeight: 0, topHeight: 128, planeLevel: 9, heightLevel: 10,
});
const cell = mapvthree.UAVGrid.encode3D([113.355, 23.125, 70], {
    planeLevel: result.planeLevel, heightLevel: result.heightLevel,
});
grid.setCellStyles([{ids: [cell.code], fillColor: '#ff664f', fillOpacity: 0.5}]);
grid.selectCellByCode(cell.code);
```

BeiDouGrid 使用相同区域边界，把 planeLevel／heightLevel 改为 level。生成完整编码格，实际边界可能超出请求边界，应读取构建结果。网格并非地形贴面，也不随地形起伏改变编码格形状。

| API | 用途 |
| --- | --- |
| `createGrid(options,{signal}?)` | 异步创建，成功后替换原显示；可用 AbortSignal 取消 |
| `setGlobalStyle(style)` | 修改整体线框、外表面和选择样式 |
| `setCellStyles(groups,{mode:'merge'}?)` | groups 每项为 ids 数组及单格样式 |
| `clearCellStyles(ids?)` | 清指定编码样式；省略清全部 |
| `selectCellByCode(code)`／`selectCellAtCoordinate(point)` | 选择当前显示范围中的格子 |
| `clearSelection()` | 取消高亮，不清业务样式 |
| `clearGrid()` | 停止跟随、取消构建并清显示 |
| `cellCount`、`revision`、`selectedCell`、`isFollowing`、`isGlobal`、`pickHeight` | 当前状态；BeiDouGrid 另有只读 `dimension` |

全局样式：lineColor 默认 '#4199e1'，lineOpacity:1，gridFillColor:'#4199e1'，gridFillOpacity:0，highlightColor:'#ffcf4d'，highlightOpacity:0.25，highlightLineColor:'#ffcf4d'，drawOnTop:false，lineEmissive:0。单格样式支持 fillColor／fillOpacity／lineColor／lineOpacity。

`merge` 只更新所列格子／字段，保留其他样式；`replace` 清空原单格样式表后应用本次分组。全局样式不被 replace 删除。风险颜色由业务把等级映射为编码列表，不由引擎自动计算风险。

`gridFillOpacity` 控制区域外表面，内部仍是多个格子。需要内部格子单独着色时用 setCellStyles。普通重建清旧单格样式和选择；跟随重建保留仍有效的编码样式和选择。

任何构建进行中（含跟随模式的后台重建），`setCellStyles`、`clearCellStyles`、`selectCellByCode`、`selectCellAtCoordinate`、`clearSelection` 都会抛 `BUSY`，鼠标点击选格会被静默忽略。跟随期间着色应等 buildcomplete 后再调用，或捕获 BUSY 后重试。

鼠标默认用三维射线选最近进入的格子；网格作为拾取兜底，让射线命中的其他可交互对象优先。要指定某高度切片，设置 `grid.pickHeight = 100`；恢复立体选格设为 null。二维北斗选零高平面，此时 pickHeight 只能是 null 或 0，设置其他值会让选格静默失效。透明外表面不代表关闭选格。

## 高度分层

两种方式选一种，不可混用：

- `bottomHeight/topHeight`：米制高度窗口，向完整编码格边界外扩。
- `startLayer/layerCount`：以包含高度 0 的格子为第 0 层，往上选择指定层数。不是“相机上方几层”，更不是网格编码级别数。

同一区域不同高度段不同颜色，可建立多个实例：

```javascript
const bands = [
    {bottom: 0, top: 50, color: '#40dd7b'},
    {bottom: 50, top: 100, color: '#ffd24a'},
    {bottom: 100, top: 150, color: '#ff664f'},
];
const bandGrids = [];
for (const band of bands) {
    const item = engine.add(new mapvthree.UAVGrid({style: {
        lineColor: band.color, gridFillColor: band.color, gridFillOpacity: 0.15,
    }}));
    await item.createGrid({west: 113.354, south: 23.124, east: 113.356, north: 23.126,
        bottomHeight: band.bottom, topHeight: band.top, planeLevel: 9, heightLevel: 13});
    bandGrids.push(item);
}
bandGrids[2].visible = false;
```

业务高度分界若不落在编码格边界，外扩后可能重叠。需要严格连续边界，应按实际格高规划分层；不要宣称任意米数都精确切开编码格。

## 相机自动选级与目标跟随

```javascript
const cameraGrid = engine.add(new mapvthree.BeiDouGrid({dimension: 3}));
await cameraGrid.followCamera({
    updateInterval: 50, targetCellPixels: 32, startLayer: 0, layerCount: 5,
});
```

三维网格相机跟随时，若高度窗口完全超出该网格的合法高度域（例如用 below／above 取相机上下范围，而相机已高于 UAVGrid 的 15872 米上限），会清空显示并派发 cellCount:0、layerCount:0 的 buildcomplete，同时继续监听，回到合法高度后自动恢复；部分越界时只显示合法部分。followTarget 不适用此规则。

未传层级／lodBands 时自动选择精度，基于相机可见地理范围、格子屏幕大小和构建预算。targetCellPixels 默认 64，较小值通常更早细分；updateInterval 默认 150 毫秒，上例为 50，连续缩放期间也检查。它不是构建完成时限或帧率。没有高度配置时，相机自动模式默认地面起一层。

固定高度分档模式：

```javascript
await cameraGrid.followCamera({
    updateInterval: 50, movementThreshold: 30, startLayer: 0, layerCount: 5,
    lodBands: [
        {minHeight: -512, maxHeight: 500, level: 6, radius: 900},
        {minHeight: 500, maxHeight: 30000, level: 4, radius: 6000},
        {minHeight: 30000, maxHeight: 1000000000, level: 1},
    ],
});
```

高度分档下限包含、上限不包含；相机高度不落在任何分档内时取最近的一档。radius 是相机周围水平窗口的半宽，单位米，不是圆形格子范围；自动模式不使用 radius／radiusScale。显式 level 或 lodBands 选择手动模式，勿再传 autoLevel:true。UAVGrid 的分档用 planeLevel／heightLevel，且每一档都要显式写全这两个值（缺省值目前不会在运行时补齐，配合层数窗口会在相机移动时报错）。

围绕飞机移动窗口，可直接传同引擎的 UAV，也可传返回位置对象的函数：

```javascript
let targetPoint = [113.3552, 23.1252, 100];
await cameraGrid.followTarget(() => ({point: targetPoint, coordType: 'WGS84'}), {
    level: 6, radius: 300, below: 30, above: 60, updateInterval: 150, // BeiDouGrid 默认 radius 为 150，这里显式放大
});
targetPoint = [113.357, 23.126, 120]; // 业务更新位置，网格后续采样刷新。
// 不再跟随时调用；保留当前显示。
await cameraGrid.followTarget(null);
```

followTarget 默认 below:60、above:120，单位米；radius 默认值两个类不同：UAVGrid 为 300，BeiDouGrid 为 150（followCamera 手动模式同样如此）。below／above 是目标上下范围。不要和固定高度窗口／层数混用。`movementThreshold`（默认 30 米）只对 followCamera 生效，followTarget 按 radius 的约 1/3 判断是否移窗。`followCamera(null)` 同样停止更新但保留显示；相机跟随、目标跟随、手动重建会互相接管。

全球粗网格受投影合法纬度约束：球体北斗至 ±88°、UAV 至 ±90°；平面 1 级完整格至 ±84°。局部跨日界线窗口不自动拆分。当前不是 MapView 瓦片树混合 LOD，也不是逐格精确视锥裁剪。

## 北斗航线及多边形覆盖

```javascript
const pathGrid = engine.add(new mapvthree.BeiDouGrid({dimension: 3, level: 6}));
const blocked = mapvthree.BeiDouGrid.encode3D([113.3552, 23.1252, 100], 6).code;
const coverage = await pathGrid.setPath([
    [113.3552, 23.1252, 100], [113.357, 23.1252, 100],
], {level: 6, coverage: 'continuous', restrictedCellIds: [blocked]});
console.log(coverage.cellIds, coverage.conflictCellIds);
if (coverage.conflictCellIds.length) {
    pathGrid.setCellStyles([{ids: coverage.conflictCellIds, fillColor: '#ff664f', fillOpacity: 0.5}]);
}
```

setPath 同时生成显示和返回穿越编码，不是只查询。默认 continuous 连续穿格；sampled 按 sampleSpacing（默认 10 米）取点，可能漏格，不能与 restrictedCellIds 一起使用。还可传 `curve`（'linear'／'centripetal'）和 `signal`。航点必须是三维 `[经度,纬度,高度]`，纬度不超过 ±85°，最多 2048 个；限制格不超过 10 万个。限制格维度、级别必须一致；结果只检查本次零宽中心线，不是飞机体积碰撞检测或持续侵入告警。返回值包含 cellIds、conflictCellIds、coverage、sampleSpacing 等，其中 layerCount 为 undefined。

```javascript
await pathGrid.setPolygon([[
    [113.354, 23.124], [113.358, 23.124], [113.358, 23.127],
    [113.354, 23.127], [113.354, 23.124],
]], {level: 6, bottomHeight: 0, topHeight: 150});
```

输入是 Polygon.coordinates：闭合外环及可选洞，不是完整 Feature。保留有正面积交集的完整格子，不裁剪格形。支持凹形和合法洞，拒绝自交、非法洞、MultiPolygon 和跨日界线区域；顶点最多 4096 个，二维网格忽略高度参数。成功后替换显示并停止跟随，返回值含 `cellIds`。

## 预算、事件和清理

构造 budget 默认 maxVertices:500000、maxChunks:256、maxStyledCells:10000；手动模式 autoDegrade 默认 false。自动模式可尝试更粗级别，结果以实际返回值为准；不要为了密度无限放大预算。

```javascript
cameraGrid.addEventListener('buildcomplete', event => {
    console.log(event.requestId, event.level, event.cellCount, event.bottomHeight, event.topHeight);
});
cameraGrid.addEventListener('buildcancelled', event => console.log(event.requestId, event.reason));
cameraGrid.addEventListener('error', event => console.error(event.error));
```

| 事件 | 时机 |
| --- | --- |
| `buildcomplete` | 新布局构建成功并提交到显示状态；UAVGrid 读取 planeLevel／heightLevel |
| `buildcancelled` | 已开始的构建被替换或取消；销毁阶段不保证再派发 |
| `error` | 构建／跟随失败；首次显式调用还需捕获 Promise 拒绝。跟随过程和挂载阶段派发的 error 不带 requestId |
| `cellselected` | 选择被设置或取消时；取消选择时 cell 为 null。`clearGrid()` 即使原本没有选中也会派发 null；普通重建清掉选择时不派发 |

错误对象带 `code`：INVALID_ARGUMENT、BUDGET_EXCEEDED、BUSY、CELL_NOT_FOUND、INVALID_STATE、DATUM_TRANSFORM_REQUIRED（allowApproximateDatum:false 时）、TARGET_LOST、DISPOSED、CANCELLED、SUPERSEDED。

没有 targetlost 事件。跟随源失效通过 error 与 isFollowing 判断；后台相机窗口构建失败保留最后成功显示并继续尝试，不能把一次 error 等同于网格已消失。

临时 `engine.remove(grid)` 保留布局、样式和用户监听，再添加重建；跟随需重新开启。永久结束用 `grid.dispose()`。上例创建的多个 bandGrids 也应逐个 dispose。
