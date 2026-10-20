# 3D Gaussian Splatting

`GaussianSplat` 用于把 3DGS 数据放入 MapV-Three 地理场景。它接受资源 URL 或内存中的 `ArrayBuffer`，二者必须且只能提供一个。

> `GaussianSplat` 当前只从完整包入口导出，`index.open.js` 不包含该类。使用前先确认安装版本存在 `mapvthree.GaussianSplat`。

## 快速开始

```javascript
const splat = engine.add(new mapvthree.GaussianSplat({
    url: '/data/site.ply',
    point: [116.404, 39.915, 0],
    scale: 1,
    opacity: 1,
}));

splat.addEventListener('loaded', event => {
    console.log('3DGS 点数：', splat.count, event.value);
});

splat.addEventListener('error', event => {
    console.error('3DGS 加载失败：', event.error);
});
```

该类没有 `ready` Promise。需要等待数据可用时监听 `loaded`，失败监听 `error`。

## 构造参数

| 参数 | 类型／默认值 | 说明 |
| --- | --- | --- |
| `url` | `string` | 3DGS 资源地址；与 `data` 二选一 |
| `data` | `ArrayBuffer` | 已读取的资源内容；与 `url` 二选一 |
| `format` | `string`／自动识别 | 可为 `splat`、`ply`、`gltf`、`glb` |
| `basePath` | `string` | glTF 外部资源的基准路径；URL 加载时默认使用资源所在目录 |
| `point` | `[lng,lat,height]`／`[0,0,0]` | 模型的地理锚点，高度单位为米 |
| `upAxis` | `'x'\|'y'\|'z'\|'-z'` | 显式指定数据向上轴；省略时读取解码结果 |
| `autoUpAxis` | `boolean`／`true` | 是否自动把数据向上轴转换到场景坐标 |
| `rotation` | `THREE.Euler\|THREE.Quaternion\|number[3]` | 锚点下的局部旋转，数组单位为弧度 |
| `scale` | `number\|THREE.Vector3\|number[3]` | 局部缩放 |
| `opacity` | `number`／`1` | 全局透明度，实际值限制在 `0～1` |

格式通常由文件头和 URL 后缀识别。当前不支持 SPZ；发现 `.spz` 数据时先转换为 `.ply`、`.splat`、`.gltf` 或 `.glb`。内存加载 glTF 且包含外部文件时必须提供 `basePath`。

## 动态控制

```javascript
splat.opacity = 0.65;
splat.setTransform({
    point: [116.405, 39.916, 12],
    rotation: [0, 0, Math.PI / 2],
    scale: 1.5,
});
```

| API | 说明 |
| --- | --- |
| `setTransform({point,rotation,scale})` | 更新地理锚点和局部变换，未传字段保持原值 |
| `opacity` | 动态调整整体透明度 |
| `count` | 已加载的 Gaussian 数量；加载前为 `0` |
| `splatMesh` | 底层渲染对象；加载前为 `null`，普通业务不应直接改写 |
| `loaded` | 解码并创建 GPU 资源后触发，`event.value` 为解码数据 |
| `error` | 下载、解码或构建失败时触发，错误位于 `event.error` |

从引擎移除会停止排序并释放当前 GPU 内容；重新添加会重新加载。大量 Gaussian 每帧都需要按相机深度排序，数量、屏幕覆盖面积和设备纹理上限都会影响性能。

## 常见错误

- 同时传 `url` 和 `data`，或二者都不传，会立即抛错。
- 把普通点云 PLY 当作 3DGS PLY；文件必须包含解码器需要的 Gaussian 属性。
- 使用 `.spz`；当前仅能识别并给出转换提示，不能渲染。
- 等待不存在的 `splat.ready`；应监听 `loaded` 和 `error`。
