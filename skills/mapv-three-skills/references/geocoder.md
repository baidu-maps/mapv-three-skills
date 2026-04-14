# Geocoder 地理编码

`Geocoder` 是 mapv-three 提供的地理编码服务，支持**地址转坐标**（地理编码）和**坐标转地址**（逆地理编码）两种核心能力。底层支持百度地图和天地图两种数据源，可通过配置切换。

> **前置条件：** 使用百度数据源时，必须先设置 `mapvthree.BaiduMapConfig.ak = 'YOUR_BAIDU_AK'`，否则会报 `BaiduMapConfig.ak is not set` 错误。天地图数据源需要天地图 token。

## 快速开始

```javascript
// 前置：设置百度地图 AK
mapvthree.BaiduMapConfig.ak = 'YOUR_BAIDU_AK';

// 创建地理编码服务实例（不依赖 engine）
const geocoder = new mapvthree.services.Geocoder();

// 地址转坐标
geocoder.getPoint('北京市海淀区上地街道上地十街10号', (point, detailInfo) => {
    console.log('坐标:', point); // { x: 116.xxx, y: 40.xxx, z: 0 }
    console.log('详情:', detailInfo);
});

// 坐标转地址
geocoder.getLocation([116.404, 39.915], (result) => {
    console.log('地址:', result.address);
    console.log('地址组成:', result.addressComponents);
});
```

## 构造函数

```javascript
new mapvthree.services.Geocoder(options?)
```

### 参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `options` | `Object` | 否 | `{}` | 配置参数 |
| `options.apiSource` | `string` | 否 | `services.API_SOURCE_BAIDU` | API 数据源，可选值见下方说明 |

### API 数据源常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `mapvthree.services.API_SOURCE_BAIDU` | `'baidu'` | 百度地图 LBS 服务，坐标系为 BD09 |
| `mapvthree.services.API_SOURCE_TIANDITU` | `'tdt'` | 天地图 LBS 服务，坐标系为 CGCS2000（与 WGS84 误差极小） |

也可以通过全局方法切换默认数据源：

```javascript
// 全局切换为天地图
mapvthree.services.setApiSource(mapvthree.services.API_SOURCE_TIANDITU);
```

## 方法列表

### getPoint(address, callback, city?)

地址转坐标（地理编码）。将文本地址解析为经纬度坐标。

**参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `address` | `string` | 是 | 地址字符串，不能为空或纯空白 |
| `callback` | `Function` | 是 | 回调函数 |
| `city` | `string` | 否 | 城市名称，用于限定搜索范围。不传则全国检索 |

**返回值**

`Promise<Object|undefined>` -- 返回一个 Promise，resolve 值为坐标点对象。

**回调参数**

`callback(point, detailInfo)` 回调接收两个参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `point` | `Object \| null` | 坐标点对象，包含 `x`（经度）、`y`（纬度）属性。解析失败时为 `null` |
| `detailInfo` | `Object \| null` | 详细信息对象，解析失败时为 `null` |

**detailInfo 结构（百度数据源）**

| 字段 | 类型 | 说明 |
|------|------|------|
| `city` | `string` | 所在城市 |
| `citycode` | `string` | 城市编码 |
| `address` | `string` | 地址 |
| `precise` | `number` | 是否精确匹配 |
| `confidence` | `number` | 置信度分数 |
| `level` | `string` | 地址类型级别 |

**detailInfo 结构（天地图数据源）**

| 字段 | 类型 | 说明 |
|------|------|------|
| `address` | `string` | 地址 |
| `confidence` | `number` | 置信度分数 |
| `level` | `string` | 地址类型级别 |

**示例**

```javascript
const geocoder = new mapvthree.services.Geocoder();

geocoder.getPoint('北京市朝阳区', (point, detailInfo) => {
    if (point) {
        console.log('经度:', point.x);
        console.log('纬度:', point.y);
        console.log('置信度:', detailInfo.confidence);
    } else {
        console.log('地址解析失败');
    }
}, '北京市');
```

也可以使用 Promise 风格调用：

```javascript
const point = await geocoder.getPoint('北京市海淀区上地十街10号', () => {});
console.log('坐标:', point);
```

---

### getLocation(point, callback, opts?)

坐标转地址（逆地理编码）。将经纬度坐标解析为结构化地址信息。

**参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `point` | `Array<number> \| Vector2 \| Vector3` | 是 | 坐标点。数组格式为 `[lng, lat]` 或 `[lng, lat, alt]` |
| `callback` | `Function` | 是 | 回调函数 |
| `opts` | `Object` | 否 | 额外选项（仅百度数据源支持） |
| `opts.poiRadius` | `number` | 否 | 周边 POI 检索半径，单位米，默认 `100` |
| `opts.numPois` | `number` | 否 | 返回周边 POI 数量上限，默认 `10` |

**返回值**

`Promise<Object|null>` -- 返回一个 Promise，resolve 值为逆地理编码结果对象。

**回调参数**

`callback(result)` 回调接收一个参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `result` | `Object \| null` | 逆地理编码结果对象。解析失败时为 `null` |

**result 结构**

| 字段 | 类型 | 说明 |
|------|------|------|
| `address` | `string` | 完整地址字符串 |
| `point` | `Object` | 坐标点对象，包含 `x`、`y` 属性 |
| `addressComponents` | `Object` | 地址组成部分 |
| `surroundingPois` | `Array<Object>` | 周边 POI 列表（仅百度数据源） |
| `business` | `string` | 所在商圈（仅百度数据源） |

**addressComponents 结构**

| 字段 | 类型 | 说明 |
|------|------|------|
| `province` | `string` | 省份 |
| `city` | `string` | 城市 |
| `district` | `string` | 区/县 |
| `street` | `string` | 街道 |
| `streetNumber` | `string` | 门牌号 |

**surroundingPois 数组元素结构（百度数据源）**

| 字段 | 类型 | 说明 |
|------|------|------|
| `title` | `string` | POI 名称 |
| `uid` | `string` | POI 唯一标识 |
| `point` | `Object` | 坐标点对象 |
| `city` | `string` | 所在城市 |
| `address` | `string` | 地址 |
| `postcode` | `string \| null` | 邮编 |
| `phoneNumber` | `string \| null` | 电话 |
| `tags` | `Array<string>` | POI 类型标签 |
| `type` | `number` | POI 类型（普通 POI） |

**示例**

```javascript
const geocoder = new mapvthree.services.Geocoder();

geocoder.getLocation([116.404, 39.915], (result) => {
    if (result) {
        console.log('地址:', result.address);
        console.log('省份:', result.addressComponents.province);
        console.log('城市:', result.addressComponents.city);
        console.log('区县:', result.addressComponents.district);
        console.log('街道:', result.addressComponents.street);
        console.log('商圈:', result.business);
        console.log('周边POI:', result.surroundingPois);
    }
});
```

自定义周边 POI 检索范围：

```javascript
geocoder.getLocation([116.404, 39.915], (result) => {
    console.log(result);
}, {
    poiRadius: 500,  // 500米范围
    numPois: 20,     // 返回最多20个POI
});
```

## 常见场景

### 场景一：地址转坐标并在地图上标注

```javascript
// 假设 engine 已初始化
const geocoder = new mapvthree.services.Geocoder();

geocoder.getPoint('北京市海淀区上地十街10号', (point, detailInfo) => {
    if (point) {
        // 将地图视角移动到解析出的坐标
        engine.map.lookAt([point.x, point.y], { range: 2000 });
        console.log('解析成功:', point.x, point.y);
        console.log('置信度:', detailInfo.confidence);
    }
}, '北京市');
```

### 场景二：坐标转地址

```javascript
const geocoder = new mapvthree.services.Geocoder();

geocoder.getLocation([116.404, 39.915], (result) => {
    if (result) {
        const addr = result.addressComponents;
        const fullAddr = `${addr.province}${addr.city}${addr.district}${addr.street}${addr.streetNumber}`;
        console.log('完整地址:', fullAddr);
    }
});
```

### 场景三：先地址编码再逆编码（链式调用）

```javascript
const geocoder = new mapvthree.services.Geocoder();

geocoder.getPoint('北京市海淀区上地街道上地十街10号', (point, detailInfo) => {
    console.log('地理编码结果:', point);

    // 用返回的坐标进行逆编码
    geocoder.getLocation(point, (result) => {
        console.log('逆地理编码结果:', result.address);
    });
});
```

### 场景四：使用天地图数据源

```javascript
const geocoder = new mapvthree.services.Geocoder({
    apiSource: mapvthree.services.API_SOURCE_TIANDITU,
});

geocoder.getPoint('北京市朝阳区', (point, detailInfo) => {
    if (point) {
        // 天地图返回的坐标为 CGCS2000 坐标系
        console.log('经度:', point.x, '纬度:', point.y);
    }
});
```

### 场景五：使用 Promise 风格

`getPoint` 和 `getLocation` 均返回 Promise，可以配合 async/await 使用：

```javascript
const geocoder = new mapvthree.services.Geocoder();

async function geocodeAddress() {
    const point = await geocoder.getPoint('北京市海淀区', () => {});
    if (point) {
        const result = await geocoder.getLocation(point, () => {});
        console.log('地址:', result?.address);
    }
}

geocodeAddress();
```

## 注意事项

1. **地址不能为空** -- `getPoint` 传入空字符串或纯空白字符串时将不会发起请求，也不会触发回调。

2. **坐标系差异** -- 百度数据源使用 BD09 坐标系，天地图数据源使用 CGCS2000 坐标系（与 WGS84 误差极小），混用时需注意坐标转换。

3. **回调与 Promise 双模式** -- 两个方法同时支持回调和 Promise。回调会先于 Promise resolve 执行。即使使用 Promise 风格，仍需传入回调函数参数（可传空函数 `() => {}`）。

4. **周边 POI 参数** -- `getLocation` 的 `opts.poiRadius` 和 `opts.numPois` 选项仅在百度数据源下生效，天地图数据源不支持该配置。

5. **周边 POI 数据** -- `surroundingPois` 和 `business` 字段仅百度数据源返回，天地图数据源的结果中不包含这两个字段。

6. **全局数据源切换** -- 调用 `mapvthree.services.setApiSource()` 会影响所有后续创建的服务实例。已创建的实例不受影响。如需单个实例使用不同数据源，请通过构造函数的 `options.apiSource` 参数指定。

7. **导入方式** -- Geocoder 通过 `mapvthree.services.Geocoder` 访问，无需单独导入。
