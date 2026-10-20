# 无人机与机队

用于沿预设航线飞行、动态改线、第一／第三人称跟随、航线托管、标签及机载视域。`UAV` 管一架，`UAVFleet` 按 ID 管多架，内部复用 UAV。

真实设备逐时刻推送位置时使用 [Twin 实时无人机接入](./twin.md#实时无人机接入)，不要把 `updatePath()` 或 `setDrones()` 当作遥测接口。独立相机巡航使用 [PathTracker](./tracker.md)，不新建导航控制器。

以下代码在已初始化的 `engine` 中执行，`mapvthree` 为引擎导出命名空间；初始化见 [引擎核心](./engine-bindmap.md)。将 `/models/uav.glb` 换成可访问的模型地址。使用包含这些导出的引擎版本，不假定所有已发布旧版本都有这些 API。

## 单架飞行

```javascript
const path = [
    [113.3552, 23.1252, 80],
    [113.3570, 23.1252, 100],
    [113.3580, 23.1260, 120],
];
const uav = engine.add(new mapvthree.UAV({
    url: '/models/uav.glb', path, speed: 20, loop: 'pingpong',
}));
await uav.ready;
uav.start();
```

| 参数 | 类型 | 默认值 | 含义 |
| --- | --- | --- | --- |
| `url`／`object` | string／Object3D | 必须二选一 | glTF 地址，或借用并克隆现有模型 |
| `path` | number[][] | 无 | WGS84 `[经度,纬度,椭球高]`，度、度、米 |
| `point` | number[] | 航线首点，否则 `[0,0,0]` | 初始位置 |
| `curve` | string | `'linear'` | `'centripetal'` 为平滑路径 |
| `speed` | number | `60 / 3.6` | 米／秒，必须大于 0 |
| `loop` | string | `'pingpong'` | 往返；`once` 停在终点；`repeat` 跳回起点；`destroy` 完成后销毁 |
| `turnDuration` | number | `2000` | 往返端点掉头时长，毫秒 |
| `scale` | number／number[] | `1` | 正数缩放 |
| `rotation`／`modelRotation` | number[] | `[0,0,0]` | 地理姿态／模型轴修正，弧度，分量顺序 `[roll, pitch, heading]`；飞行中姿态由航线计算，会覆盖 rotation |

机身坐标为 +X 前、+Y 左、+Z 上。URL 加载的 glTF 只做 Y-up 到 Z-up 的转换（绕 X 轴转 90°），不会把 glTF 的 +Z 前向转成机头，所以模型机头须朝 +X，否则用 `modelRotation` 修正；`object` 传入的模型完全不转换。路径至少两个不同有效点，最多 2048 点（按去重前计数），纬度限 ±85°，单段最长 100 km（同时要求单段经度差 ≤10°、纬度差 ≤1°）；`curve:'centripetal'` 还要求所有航点距首点 100 km 内。仅支持 WGS84 椭球高，不自动识别海拔、GCJ-02 或 BD-09；地图 `coordType` 显式设为非 WGS84 时 `engine.add` 会抛 `UNSUPPORTED_MAP`。

```javascript
uav.pause();
uav.resume();
uav.setSpeed(25);
await uav.updatePath([...path, [113.3590, 23.1265, 130]], {continueTolerance: 2});
```

| 方法／属性 | 行为 |
| --- | --- |
| `start(options?)` | 从起点开始一轮；可覆盖 speed、loop、turnDuration，覆盖值会写回配置，之后各轮沿用 |
| `pause()`／`resume()` | 冻结／继续位移和掉头计时 |
| `stop({reset:false})` | 默认保留位置并解除跟随；reset:true 回到起点 |
| `setPath(path,{curve,signal}?)` | 替换路线，结束旧飞行及跟随，需重新 start；curve 不传时重置为 `'linear'`；不派发 pathchange／statechange |
| `updatePath(path,options?)` | 提交完整新路线，尝试续飞；默认容差 2 米；options 还可同时改 speed、loop、turnDuration、curve、signal，curve 不传时沿用原曲线 |
| `getPath()` | 实际采样路径副本，不是渲染对象 |
| `setTransform({point,rotation,scale})` | 静态变换；飞行或暂停时改位置／姿态需先停止 |
| `ready` | Promise，模型加载完成后 resolve；加载失败或对象销毁时 reject。`start`、`follow`、`enableViewShed` 都要求已 `engine.add` 且 ready |
| `state`、`point`、`speed`、`runId` | 当前状态（idle／running／turning／paused／stopped／completed）、坐标、速度、飞行轮次 |

`updatePath()` 能衔接时保留进度、方向、暂停状态及跟随；不能衔接时从新路径起点处理。未启动／主动停止／暂停的飞机不会因为更新路径自动起飞，但 `loop:'once'` 已完成（completed）的飞机在更新后会自动起飞并开启新的 runId。平滑曲线新增节点可能改变原线形，不能只按数组前缀判断能否续飞。

## 多机管理和自动航线

优先用 `route` 托管随飞机变化的航线，无需再独立初始化 Polyline 或 TubeLine。

```javascript
const fleet = engine.add(new mapvthree.UAVFleet({
    url: '/models/uav.glb', speed: 20, loop: 'pingpong',
    route: {type: 'pipeline', radius: 8, color: '#40ddcb', opacity: 0.25},
}));
await fleet.setDrones([
    {id: '001', path: [[113.3552, 23.1252, 80], [113.358, 23.126, 120]],
        properties: {name: '巡检一号'}},
    {id: '002', path: [[113.3552, 23.1262, 100], [113.358, 23.127, 120]],
        route: {radius: 4, color: '#ff664f', opacity: 0.5}},
]);
fleet.startAll();
await fleet.addOrUpdate({id: '001', path: [
    [113.3552, 23.1252, 80], [113.358, 23.126, 120], [113.359, 23.127, 120],
]});
```

`001` 继承全局管道，`002` 部分覆盖外观（未写 type 的样式对象沿用上级类型）。`route` 默认 false；true 或不带 type 的样式对象在没有上级类型时启用 polyline。支持的类型只有 `'polyline'`、`'pipeline'`，箭头和虚线是 polyline 的样式，不是另两个类型。Fleet 必须先 `engine.add`，才能调用 `addOrUpdate`、`setDrones`、`showLabels`。

| 配置 | 类型／默认值 | 含义 |
| --- | --- | --- |
| `continueTolerance` | number，2 | 更新航线的续飞容差，米 |
| `route.visible` | boolean，true | 隐藏但保留托管对象 |
| `route.dropMeters` | number，0 | 仅 polyline：显示线整体下移的米数（非负），不改变飞机路径；pipeline 忽略此值，管道始终包围飞机实际路径 |
| polyline 样式 | color:'#40ddcb'、lineWidth:2、opacity:1 | 还可设 dashed、dashArray、dashOffset、dashRatio、arrow、gradient |
| pipeline 样式 | radius:5、radialSegments:8、color:'#3edbd0'、opacity:0.25 | 不接受 arrow、lineWidth 等线样式 |

在两种类型之间切换时，`color`、`opacity`、`visible`、`dropMeters` 会从原类型继承，其余样式取新类型默认值。从 pipeline 切到 polyline 若不写 opacity，会沿用 0.25。

```javascript
// 从管道切换为流动箭头航线；显式写 opacity，否则继承管道的 0.25。颜色需要与底线有对比。
await fleet.addOrUpdate({id: '001', route: {
    type: 'polyline', color: '#40ddcb', lineWidth: 10, opacity: 1,
    arrow: {color: '#102b3b', length: 12, spacing: 35, speed: 15},
}});
const routeObject = fleet.getRoute('001');
console.log(routeObject, fleet.get('001').getPath());
```

`arrow`／`gradient` 按整个字段替换，null 关闭。箭头默认速度 0；到达端点开始掉头时，往返方向自动翻转。暂停飞机不会暂停独立材质箭头流动。样式单位见 [航线与 TubeLine](./polyline-wall.md)。

| API | 行为 |
| --- | --- |
| `addOrUpdate(data,{signal}?)` | 按 ID 新增或部分更新，返回 Promise<UAV>；完成路径去重时返回 undefined |
| `setDrones(list,{signal}?)` | 全量名单同步，成功后移除缺席成员；空数组清空，新成员需显式 start |
| `get(id)`、`getIds()`、`size` | 查询成员、ID 和数量 |
| `start/pause/resume/stop(id,...)` | 单机控制 |
| `startAll/pauseAll/resumeAll/stopAll(...)` | 批量控制 |
| `follow(id,options)`／`stopFollow()` | 跟随成员／停止机队跟随 |
| `getRoute(id)` | 返回托管 Polyline 或 TubeLine；未启用／尚未就绪为 undefined |
| `remove(id)`／`clear()` | 删除一架／全部，同时清理托管航线和对应资源 |
| `clearCompleted(id?)` | 清指定／全部已完成路径记录 |

ID 支持非空字符串和非负安全整数，数字 `1` 与字符串 `'1'` 区分。已有成员更换模型需要移除再添加。新增和动态改线都传完整航点数组。

`setDrones` 不适合只含变化目标的增量消息，否则会删除未列出的成员。失败时不执行名单外删除，但已成功更新的成员保留，不承诺事务回滚。相同 ID 的并发有效更新以最新请求为准；连续全量同步取消前一轮。

`loop:'destroy'` 完成后移除飞机及托管航线，机队记录该 ID 最近完成的路径；重复下发同一路径不会重建。不同路径可重建，业务明确需要重播时先 `clearCompleted(id)`。仅有单个 UAV 时不提供机队级去重。

## 第一／第三人称和机载视域

```javascript
uav.follow({mode: 'thirdPerson', range: 100, pitch: 60, heading: 0, transitionDuration: 800});
uav.follow({mode: 'firstPerson', offset: [0.4, 0, 0.2], transitionDuration: 800});
uav.follow({mode: 'thirdPerson'}); // 恢复此前记住的第三人称配置。
uav.unfollow();
fleet.follow('001', {mode: 'firstPerson'});
fleet.stopFollow();
```

follow 默认 mode:'thirdPerson'、range:100、pitch:60、heading:0、offset:[0.4,0,0.2]、transitionDuration:0。range 为正米数，pitch 范围 0～89°，0 俯视；heading 是相对机头逆时针偏角，注意与下方机载视域 heading 的正方向不同。transitionDuration 为毫秒；offset 是机身前／左／上的米制偏移。相机跟随不要求先启动飞行，但要求已 `engine.add` 且 `ready`；`firstPerson` 或 `transitionDuration > 0` 只支持球面地图或 BlankMap3D，否则抛 `UNSUPPORTED_MAP`。单机 `isFollowing`、`followMode` 可读取状态。

一个引擎同一时刻只有一个跟随目标，跟随新目标会解除旧目标。跟随期间禁用地图默认交互；第三人称下右键拖动调整 heading／pitch、滚轮调整 range，并按模式记住。第一人称时隐藏机身。页面隐藏时飞行冻结，恢复后不追帧。

```javascript
uav.enableViewShed({heading: 0, pitch: 55, fov: 55, distance: 550,
    opacity: 0.65, offset: [0, 0, -1]});
uav.setViewShedOptions({heading: 30, pitch: 45});
// 结束分析时调用。
uav.disableViewShed();
```

上述值也是视域默认值。heading 相对机头向右为正；pitch 范围 0～180°，0 向下、90 平视、180 向上。fov 在 0～180° 之间，distance 为分析距离，offset 随机身姿态变化且不随模型缩放。旧参数 depression 已删除，传入会直接抛错（换算为 `pitch = 90 - depression`）。方向和起点自动随飞机更新。重复调用 `enableViewShed` 等同于部分更新。

这是 [ViewShedAnalysis](./gis-analysis.md) 的机载封装，提供遮挡可视化，不输出任务区盲区面积、覆盖率或多机重叠统计；没有相机画面宽高比和画面滚转支持。投影地图下距离语义沿用底层分析器，不能当作高纬度精密测量。

## 共享数据和限时标签

```javascript
fleet.dataSource.defineAttributes({
    text: attributes => attributes.name || String(attributes.id),
    color: attributes => attributes.risk === 'warning' ? '#ff664f' : '#40ddcb',
});
const labels = engine.add(new mapvthree.Label({type: 'text', textSize: 16, textOffset: [0, 24]}));
const points = engine.add(new mapvthree.SimplePoint({size: 8, vertexColors: true}));
labels.dataSource = points.dataSource = fleet.dataSource;
await fleet.addOrUpdate({id: '001', properties: {name: '巡检一号', risk: 'warning'}});
```

数据源包含当前点位和 properties，以及 id、state、speed、height、isFollowing、followMode 等机队状态。它不是航线数组；业务属性浅合并，不能覆盖机队保留字段。业务可定义属性映射，但不要增删、清空或 dispose 机队数据源。拾取或查询时读 `attributes.id` 得到业务 ID；DataItem 自身的 id 是 `"string:001"`、`"number:1"` 这种带类型的内部值。

```javascript
fleet.setLabelOptions({duration: 5, fade: 0.2, style: {textSize: 16}});
fleet.showLabels([{id: '001', type: 'warning', title: '低电量', content: '剩余电量 18%'}]);
fleet.showLabels([{ids: 'all', title: '准备返航', duration: 10}]);
fleet.addEventListener('labelexpired', event => console.log(event.id, event.title));
fleet.hideLabels('001');
```

构造时可配置 `labels:{duration,fade,maxDistance,collision,offset,style}`。duration 默认 0 常驻，须非负；fade 默认 0.2 秒，淡入、淡出都计入总时长，各不超过总时长一半；maxDistance 是相机到飞机的最大显示距离（正米数），默认 null 不限；collision 默认 false，true 时机队标签之间互相避让（不是飞机碰撞检测）；offset 是标签沿竖直方向的高度偏移米数，可为负，默认 0。style 只接受 Label 的文本／图标样式字段，未知字段会报错；`style.icon` 设默认图标地址，`iconWidth`／`iconHeight` 默认 40，按有无图标和文字自动选择文字、图标或图文类型。

showLabels 每项可含 id 或 ids（数组／'all'，二者只能选一）、type、title、content、color、duration、icon。类型为 info／warning／error，可用 color 覆盖（CSS 颜色或 RGB(A) 数组）。icon 省略继承默认图标，null 取消图标；既无图标又无非空文字时，该机的标签被清除，不计时也不触发到期事件。每机一条，重复 show 替换并重新计时；ids:'all' 只作用于当前成员。`hideLabels()` 不传参隐藏全部，传入不存在的 id 抛 `NOT_FOUND`。

`setLabelOptions` 中 duration／fade 只作用于之后的显示；style、maxDistance、collision、offset 立即生效，不重新计时。自然到期触发 labelexpired，手动隐藏／删除不触发；事件不代表风险解除。`labelDataSource` 是临时标签专用源，独立于常驻 `dataSource`。

## 事件与释放

| 事件 | 时机与 payload |
| --- | --- |
| `loaded` | 模型加载完成 |
| `flightstart`／`flightfinish` | 飞行开始／一轮结束，带 runId、point；flightfinish.reason 为 completed／stopped／superseded（重新 start）／pathchanged（setPath）／disposed |
| `statechange` | 飞行状态变化，`{previousState, state, runId}`；setPath 不派发 |
| `endpoint` | 抵达端点，point、direction、count 表示本帧端点结果 |
| `pathchange` | 仅 `updatePath` 派发，continued、unchanged 表示续飞／是否未变化 |
| `followchange` | 跟随状态变化，`{isFollowing, mode, reason}`；reason 为 following／superseded／stopped／removed／disposed。同一飞机切换模式时会先收到一次 isFollowing:false（superseded）再收到 true |
| `error` | 异步运行失败，初始化和显式调用也应捕获 Promise |
| `disposed` | 永久释放，带 reason |
| Fleet `added/updated/removed` | 对应成员增删改；转发单机事件时附带 id、uav；`loop:'destroy'` 完成移除时也触发 removed |

错误对象带 `code`：INVALID_ARGUMENT、INVALID_STATE（如未 add 或未 ready）、DISPOSED、OBJECT_IN_USE、UNSUPPORTED_MAP、NOT_FOUND、CANCELLED、SUPERSEDED、RESOURCE_LOAD_FAILED、BUDGET_EXCEEDED。

移出引擎与永久销毁不同：临时移出停止运行时注册，可以再添加；`dispose()` 才永久释放。移出时会关闭机载视域、解除跟随，重新加入后不会自动恢复；Fleet 移出时还会清空全部限时标签，并取消进行中的 `addOrUpdate`／`setDrones`。机队自有航线通过机队更新，不要手动销毁 getRoute 返回值，也不要缓存它（更新时可能被替换）。外部展示组件解除 dataSource 绑定后再移除，最后由机队释放共享源：

```javascript
labels.dataSource = null;
points.dataSource = null;
engine.remove(labels);
engine.remove(points);
fleet.dispose();
uav.dispose();
```

模型数量、航线复杂度、视域数目和浏览器硬件共同决定性能；没有经过目标环境压测，不承诺一万架精细模型和视域同时运行。
