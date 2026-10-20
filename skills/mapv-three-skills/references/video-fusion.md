# 视频融合、接流与定标

`VideoFusion` 负责场景表面融合；`VideoStream` 负责 WHEP／WebRTC 接流；`VideoCalibrationEditor` 负责可撤销的定标草稿。它们在 Web 运行，不依赖云渲染引擎。本文统一说明，按用户任务读取对应段落。

以下代码使用已初始化的 `engine`、引擎命名空间 `mapvthree`，以及项目已有的 Three.js 命名空间 `THREE`。初始化见 [引擎核心](./engine-bindmap.md)，地形／道路加载见 [瓦片与地形](./vector-tile.md) 和 [3DTiles](./3dtiles.md)。示例资源地址须换成自己可访问的资源。

## 点对定标后投影

1. 创建视频融合对象并等待媒体就绪。
2. 选择至少四个分布良好的地标，每对是同一地物的地图经纬高和原视频 UV。
3. 提交定标，播放并检查地标与画面的对应关系。

```javascript
const fusion = engine.add(new mapvthree.VideoFusion({
    video: '/videos/uav.mp4', opacity: 0.85,
}));
await fusion.ready;
fusion.video.muted = true;
// 仅演示数据形状；真实项目必须替换成该视频中同一地物的实测点对。
const points = [
    {id: 'a', point: [113.354, 23.126, 0], uv: [0, 0]},
    {id: 'b', point: [113.356, 23.126, 0], uv: [1, 0]},
    {id: 'c', point: [113.356, 23.124, 0], uv: [1, 1]},
    {id: 'd', point: [113.354, 23.124, 0], uv: [0, 1]},
];
await fusion.setKeyframes([{timeSeconds: 0, points}], {
    interpolation: 'footprint', heightMode: 'landmarks',
});
await fusion.play();
```

point 为 WGS84 `[经度,纬度,椭球高]`，uv 左上角为 `[0,0]`，右下角 `[1,1]`。不要把播放器黑边、缩放后的容器像素直接当作原图 UV；也不要把无人机位置当作地面地标位置。

有点对时会自动确定计算原点及投影范围，**无需再填写接收中心和 size**。不定标、直接铺到演示矩形时，用 `receiver:{origin:[lng,lat,height],size:[width,length],heightTolerance:0.1}`，尺寸单位米，size 每边不超过 10000。既没有定标也没有 receiver 不会投影。

定标时如果同时传了 receiver，它仍然生效：heightMode 默认 `'receiver'`，地标高度必须落在 receiver 高度容差内，投影面高度取 receiver 原点高度；geoPath 的 10 km 范围也以 receiver 原点为准。纯点对定标建议不传 receiver，或显式使用 `heightMode:'landmarks'`。

`fusion.ready` 要在 `engine.add(fusion)` 之后才会 resolve，先 await 再 add 会一直挂起。

| 构造参数 | 类型 | 默认值 | 含义 |
| --- | --- | --- | --- |
| video | string／HTMLVideoElement／对象 | 必填 | 视频地址、外部元素或 `{type:'webrtc',url,...}` |
| coordType | string | `'WGS84'` | 只能是 `'WGS84'`，其他值直接抛错；地图 coordType 显式设为非 WGS84 时 `engine.add` 也会抛错 |
| receiver | 对象／null | null | 演示矩形接收面，见上文；定标时仍影响高度约束 |
| opacity | number | 1 | 0～1；运行时直接赋值不校验，暂停状态下不会自动重绘 |
| crop | 对象／null | null | 原视频归一化裁剪范围 |
| mask | Texture／null | null | 灰度遮罩 |
| invertMask、flipX、flipY | boolean | false | 反转遮罩／左右翻转／上下翻转 |
| heightTolerance | number／null | null | 接收高度额外余量，米 |

## 关键帧、接收高度和静态校正

每帧 `{timeSeconds,points,crop?}`，4～256 对点，点 ID 唯一。最多 2048 帧，非负时间严格递增，相邻至少 1 微秒；地标限制在共同局部原点 10 km 内。不要传共线、重合点或跨投影地平线的画面范围。

| setKeyframes 选项 | 默认值 | 含义 |
| --- | --- | --- |
| interpolation | `'matrix'` | matrix 插值矩阵系数；footprint 插值地面四角后重新求矩阵 |
| heightMode | `'receiver'` | receiver 使用接收高度约束；无 receiver 自动取地标高度；landmarks 明确使用地标平均高度 |
| signal | 无 | AbortSignal，取消本次更新 |

推荐动态地面定标显式传 footprint 和 landmarks。后续调用省略选项保留已有设置。一个关键帧表示固定投影；多个关键帧按视频时间推进。下面仅用平移点对演示第二关键帧的结构，真实飞行视频必须逐时刻重新标记：

```javascript
const pointsAt5 = points.map(item => ({...item,
    point: [item.point[0] + 0.0001, item.point[1], item.point[2]],
}));
await fusion.setKeyframes([{timeSeconds: 0, points}, {timeSeconds: 5, points: pointsAt5}], {
    interpolation: 'footprint', heightMode: 'landmarks',
});
```

footprint 在中间投影折叠或退化时保持本轨道最后有效投影，随后有效时恢复；没有有效投影就不绘制，换轨道不复用旧结果。matrix 退化时不绘制，保持原 UV 残差拒绝阈值 0.02。动态推进优先取实际视频帧 mediaTime，不提前使用尚未显示的 seek 目标时间；不支持帧回调时使用 currentTime。

landmarks 不是三维地形拟合：先在局部平面求地面位置到 UV 的单应映射，再在接收高度范围内的可用场景深度表面显示。道路或地形高度超出时可放宽余量：

```javascript
fusion.heightTolerance = 500; // 在地标高度跨度之外，上下各增加 500 米。
fusion.heightTolerance = 300; // 运行时赋值生效，不需重新定标。
```

null 恢复 receiver.heightTolerance，未提供 receiver 则为 0.1 米。0 仅取消额外余量，仍保留地标高度跨度及深度精度补偿。放宽会让范围内其他表面也接收画面，不能解决错误坐标、UV、地标匹配或透视参数。它不是完整相机内外参的三维重建。

静态超过四点时可做残差校正：

```javascript
await fusion.setCalibration(points, {heightMode: 'landmarks',
    crop: {min: [0, 0.25], max: [1, 1]},
    warp: {mode: 'tps', smoothing: 0.05, resolution: 128},
});
fusion.clearKeyframes(); // 动态优先；已有关键帧时显式切回静态。
console.log(fusion.getCalibrationDiagnostics());
```

四点时只使用单应矩阵；warp 的作用需要更多点。mode 支持 none（默认）、tps、radial、localIDW。smoothing 默认 0.05，resolution 默认 128、范围 32～1024；localIDW 的 neighbors:6、power:2、radiusScale:3；outlierScale:0 默认不过滤残差点。纹理预算 `resolution² × 有效点数 <= 40000000`。校正仅用于静态，不能套用到动态关键帧上。

diagnostics 返回每帧 timeSeconds、rmsErrorPixels、maxErrorPixels、pointErrors 等；视频分辨率未知时返回 []。四点拟合误差接近 0 不保证点对真实正确或所有地形都对齐。

`clearKeyframes()` 清动态轨道并恢复静态定标（若有）；`clearCalibration()` 只清静态。都清除后恢复显式 receiver 矩形，没有 receiver 就停止投影。

## 裁剪、遮罩与翻转

```javascript
fusion.crop = {min: [0, 0.5], max: [1, 1]}; // 只保留原视频下半部分，不拉伸。
const maskTexture = await new THREE.TextureLoader().loadAsync('/textures/video-mask.png');
fusion.mask = maskTexture;
fusion.invertMask = false;
fusion.flipX = false;
fusion.flipY = false;
```

赋值即生效，不需重新点击定标或重建实例。crop 与当前关键帧 crop 取交集；设 null 恢复实例完整范围。遮罩读取纹理红色通道：白显示、黑隐藏、灰半透明；按原视频方向准备，同画面宽高比最便于理解。反转交换显示和隐藏，mask=null 关闭。

翻转作用于最终原图采样坐标，裁剪和遮罩使用同一原图坐标。外部纹理由调用者管理，结束时先解绑再 dispose，不由 VideoFusion 销毁。

## 视频接入与播放

只有 HTTP(S) 视频文件或浏览器可播放媒体，才可直接使用地址。RTSP 是持续视频流，但浏览器不直接播放 `rtsp://...`，需要已有服务转为浏览器可接收的 WHEP／WebRTC；此库不含 RTSP 转流服务器。

推荐由融合对象内部托管连接：

```javascript
const liveFusion = new mapvthree.VideoFusion({
    video: {type: 'webrtc', url: 'https://media.example.com/uav001/whep'},
});
// engine.add 会立即开始连接并同步派发 connecting，监听要在 add 之前绑定。
liveFusion.stream.addEventListener('statechange', event => console.log(event.state));
liveFusion.stream.addEventListener('error', event => console.error(event.error));
engine.add(liveFusion);
await liveFusion.ready;
// 接流就绪后，仍须使用该实时视频的实际点对定标，才会出现场景投影。
```

上面的 media.example.com 只是地址占位，不是可用演示服务。`video:{type:'webrtc'}` 内部创建 VideoStream；不需要业务再创建一份。必须是 WHEP 信令地址，不是任意厂商 WebRTC 播放地址、RTSP 地址或播放器页面。

仅在多个展示组件需要共享同一媒体连接时独立创建：

```javascript
const stream = new mapvthree.VideoStream({url: 'https://media.example.com/uav001/whep'});
await stream.open();
const sharedFusion = engine.add(new mapvthree.VideoFusion({video: stream.video}));
await sharedFusion.ready;
// 页面可将 stream.video 放进自己的预览容器；最终所有消费者结束后再释放 stream。
```

| VideoStream 参数 | 类型／默认值 | 说明 |
| --- | --- | --- |
| url | string，必填 | WHEP HTTP(S) 地址 |
| timeout | number，10000 | 连接到首帧播放的总超时，毫秒 |
| reconnect | boolean，true | 成功播放后的断线重连；首次失败直接拒绝 |
| token | string／function，无 | Bearer 或每次连接获取新令牌的同步／异步函数 |
| iceServers | RTCIceServer[]，默认从服务 Link 读取 | 显式 [] 禁用 STUN／TURN 配置 |

open 返回首帧可播的 Promise；close 断开但可再 open；dispose 永久释放。video 引用跨重连保持稳定。状态为 idle、connecting、playing、reconnecting、closed、failed、disposed；statechange 含 previousState 和 state，error 含 error.code／message。视频暂停与连接状态不同，读取 video.paused。成功播放后断线会自动重连：连接 disconnected 后宽限 3 秒，最多重试 5 次，间隔 1／2／4／8／10 秒，用尽进入 failed。

服务须支持 POST SDP（返回 201＋Location）和会话 DELETE；OPTIONS 可选（返回 404／405 也会继续）。另有三条硬约束：Location 必须与 WHEP 地址同源，否则报 INVALID_RESPONSE；请求不跟随重定向，302 跳转会失败；请求不带 Cookie，鉴权只能用 token（Authorization: Bearer）。跨域时暴露 Location 和可选 Link。HTTPS 页面使用 HTTPS 服务；http 地址也能带 token，但令牌会明文传输，生产环境应使用 HTTPS。浏览器须能解码流的编码格式；不支持 trickle ICE，会等候选收集完成再发 SDP。服务不采用 WHEP 时需自身适配器。浏览器接收端时间没有自动对应无人机采集时钟，直播定标／遥测联动需业务统一时间基准。

错误码：VideoStream 有 INVALID_OPTIONS、UNSUPPORTED、TIMEOUT、CONNECTION_LOST、CONNECTION_ERROR、PLAYBACK_ERROR、HTTP_ERROR、INVALID_RESPONSE、CANCELLED、DISPOSED；VideoFusion 有 INVALID_ARGUMENT、MEDIA_ERROR、MEDIA_TIMEOUT（片段准备超过 30 秒）、MEDIA_CHANGED、LIVE_UNSUPPORTED、SUPERSEDED、ABORTED、DISPOSED。

文件视频控制示例，在按钮等用户操作中调用：

```javascript
await fusion.loadVideo(); // 等待画面数据，不自动播放。
fusion.pause();
await fusion.play();
fusion.seek(3);
await fusion.playSegment(0, 5, {autoplay: true});
fusion.addEventListener('segmentend', event => console.log(event.startSeconds, event.endSeconds));
fusion.clearSegment();
```

playSegment 的 Promise 表示起点准备好并按选项启动，不是播完整段；自然到末端发 segmentend。clearSegment 保持当前位置和播放状态，退出片段约束。片段已结束时再调 play 会重播该片段。直播不支持 seek／playSegment：seek 同步抛 LIVE_UNSUPPORTED，playSegment 返回被拒绝的 Promise。ready 是一次性初始化 Promise，首次失败后的重试应等待 play 或 stream.open，不能反复等待旧 ready。

## 地理平移路径与导入导出

```javascript
fusion.setGeoPath([
    {timeSeconds: 0, point: [113.3552, 23.1252, 100]},
    {timeSeconds: 5, point: [113.3562, 23.1252, 110]},
    {timeSeconds: 20, point: [113.3572, 23.1262, 120]},
], {interpolation: 'smooth', compose: true});
const calibration = fusion.exportCalibration();
await fusion.loadCalibration(calibration);
```

路径为相对首点的地面 XY 平移，高度不是投影面高度。interpolation 默认 linear，可选 smooth；compose 默认 false，true 才平移已有定标结果。需要视频相机变化时优先提供实际定标关键帧，不把飞机轨迹自动当作视频定标。

exportCalibration 输出原生 Web v2 定标对象，loadCalibration 接受对应 v1／v2。导出内容包含当前视频的 `source.url`；导入时会校验它与当前视频地址一致，不一致直接抛错（CDN 换域名、签名参数变化都会导致失败）。带签名参数的地址会原样写进定标文件，分享前注意去掉令牌。导入 v1 会强制使用 matrix 插值和 receiver 高度模式，并清掉现有静态定标。不要把任意 GeoJSON 或 UE 世界厘米矩阵直接交给它；外部点对须先核对坐标系、UV 方向和时间，再转换为 setKeyframes 所需结构。

## 定标工作台集成

VideoCalibrationEditor 是无界面的草稿模型；不自动生成 DOM、地图拾取或播放器控件。UI 监听 change 刷新，把地图点及原图 UV 交给 updatePoint。示例复用上面的 points，演示一次完整静态预览和应用：

```javascript
const editor = new mapvthree.VideoCalibrationEditor(fusion);
editor.setMode('static');
editor.addFrame(0);
for (const pair of points) {
    const id = editor.addPoint(pair.id);
    editor.updatePoint(id, {point: pair.point, uv: pair.uv});
}
await editor.preview();
console.log(editor.diagnostics());
await editor.apply();
const draft = editor.exportDraft();
editor.importDraft(draft);
```

| API | 用途 |
| --- | --- |
| setMode('static'／'dynamic',warp?) | 选择静态当前帧或动态轨道；初始 dynamic |
| addFrame(timeSeconds?,{copyPrevious:false}?) | 默认当前视频时间；可复制此前帧点对 |
| selectFrame(index)／removeFrame() | 切换／移除当前帧 |
| addPoint(id?)／selectPoint(id)／removePoint(id?) | 点对管理；addPoint 返回 ID |
| updatePoint(id,{point,uv}) | 地图和视频拾取可分别提交，草稿允许暂未配齐 |
| setCrop(crop)／useMarkedRegion() | 当前帧裁剪／按已标 UV 的包围矩形裁剪 |
| undo()／redo() | 草稿撤销重做 |
| preview()／cancel()／apply() | 可撤销场景预览／恢复预览前状态／正式应用 |
| exportDraft()／importDraft(data) | 草稿交换，包含尚未配齐的点对 |
| rebindVideo() | 关联新视频并清除旧 UV，保留地面点，需重新确认图像点 |
| frames／currentFrame／selectedId／canUndo／canRedo／matchesVideo | UI 状态 |

使用地图点击获得 WGS84 point 后调用 updatePoint；视频取点需先减去 contain 黑边，再逆向抵消预览平移和缩放，最后除以原图尺寸。只接受原图内 0～1 的 UV。不要把 React 示例面板类当成引擎导出。

预览会校验完整点对、视频关联和投影范围；preview 已把草稿临时写入 fusion，apply 才正式提交。未 apply 时调用 `editor.dispose()` 会恢复预览前的定标。草稿不能当运行时定标对象导入 fusion，应用结果由 fusion.exportCalibration 获取；反过来，`importDraft` 可以接受 fusion 导出的 v1／v2 定标，用于在已有定标上继续编辑。视频分辨率未知时 `editor.diagnostics()` 会抛错（fusion 的 getCalibrationDiagnostics 则返回 []）。关闭工作台先 `await editor.dispose()`，再释放 fusion。

## 排查与释放

- 完全无画面：先检查媒体就绪、点对定标／receiver 是否存在、浏览器播放限制、视频跨域和 WebGL 纹理权限。
- 投影跨地平线：核对点对顺序、对应地物和 UV；裁掉天空／远景范围。提高 heightTolerance 不解决二维单应的地平线退化。
- 地形或 HDMap 部分缺失：检查表面是否进入深度接收链、实际高程与高度余量；再检查遮罩／crop。没有深度信息的表面不能凭扩大高度范围补出。
- 关键帧间变形：检查各帧同一地物、时间基准和地标分布，优先 footprint；不要以四点 RMS=0 认定全图正确。
- 粒子、拖尾、透明面等不写深度的半透明物体位于接收面上方时，会被视频画面覆盖。
- 每个 VideoFusion 实例都有独立的全屏后处理通道，多路视频的开销按路数线性增长。

临时移出融合对象保留媒体连接，可再次添加；永久释放用 dispose。外部 video／stream／mask 由创建方管理，暂停共享 video 会影响所有消费者。按前面示例创建资源时，结束代码为：

```javascript
await editor.dispose();
fusion.mask = null;
maskTexture.dispose();
fusion.dispose();
liveFusion.dispose(); // 内部托管连接一并释放。
sharedFusion.dispose();
stream.dispose(); // 所有共享消费者结束后统一释放。
```
