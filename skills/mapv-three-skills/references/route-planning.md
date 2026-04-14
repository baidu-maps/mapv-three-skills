# 路线规划服务

mapv-three 提供四种路线规划服务，用于在三维地图上进行驾车、步行、骑行和公交路线的搜索与可视化渲染。所有路线服务均继承自 `BaseRoute` 基类，共享统一的搜索与渲染接口。

> **前置条件：** 使用百度数据源时，必须先设置 `mapvthree.BaiduMapConfig.ak = 'YOUR_BAIDU_AK'`，否则会报 `BaiduMapConfig.ak is not set` 错误。

## 目录

- [通用说明](#通用说明)
- [DrivingRoute 驾车路线规划](#drivingroute-驾车路线规划)
- [WalkingRoute 步行路线规划](#walkingroute-步行路线规划)
- [RidingRoute 骑行路线规划](#ridingroute-骑行路线规划)
- [TransitRoute 公交路线规划](#transitroute-公交路线规划)
- [搜索结果对象](#搜索结果对象)
- [状态码常量](#状态码常量)
- [常见场景](#常见场景)
- [注意事项](#注意事项)

---

## 通用说明

### 引入方式

```javascript
import * as mapvthree from '@baidu/mapv-three';

// 通过 services 命名空间访问
const route = new mapvthree.services.DrivingRoute(options);
```

### 数据源切换

驾车路线（`DrivingRoute`）和公交路线（`TransitRoute`）支持百度地图和天地图两种数据源。步行和骑行仅支持百度地图数据源。

```javascript
// 全局切换数据源
mapvthree.services.setApiSource(mapvthree.services.API_SOURCE_TIANDITU);

// 或在实例化时指定
const route = new mapvthree.services.DrivingRoute({
    apiSource: mapvthree.services.API_SOURCE_BAIDU,
    // ...
});
```

| 常量 | 值 | 说明 |
|---|---|---|
| `API_SOURCE_BAIDU` | `'baidu'` | 百度地图 LBS 服务，坐标系 bd09 |
| `API_SOURCE_TIANDITU` | `'tdt'` | 天地图 LBS 服务，坐标系 CGCS2000 |

### renderOptions 公共配置

所有路线服务的构造函数均支持 `renderOptions` 对象，用于控制路线在地图上的渲染行为。

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `engine` | `Engine` | - | 渲染引擎实例，提供后路线会自动绘制到地图上 |
| `autoViewport` | `boolean` | `false` | 搜索完成后是否自动调整地图视野以适配路线范围 |
| `panel` | `string \| HTMLElement` | - | 路线详情面板容器，可传 DOM 元素或元素 ID 字符串 |

### 公共回调函数

所有路线服务的构造函数均支持以下回调：

| 参数 | 类型 | 说明 |
|---|---|---|
| `onSearchComplete` | `Function` | 路线搜索完成时触发，参数为搜索结果对象 |
| `onMarkersSet` | `Function` | 起终点标记点渲染完成时触发 |
| `onPolylinesSet` | `Function` | 路线折线渲染完成时触发 |
| `onResultsHtmlSet` | `Function` | Panel 面板 HTML 渲染完成时触发 |

### 公共方法

以下方法由 `BaseRoute` 基类提供，所有路线服务均可调用：

| 方法 | 返回值 | 说明 |
|---|---|---|
| `search(start, end, opts?)` | `Promise<Object>` | 搜索路线，返回结果 Promise |
| `getResult()` | `Object \| null` | 获取最近一次搜索结果 |
| `clearResults()` | `void` | 清除搜索结果并移除地图上的所有覆盖物（折线、标记、面板） |
| `toString()` | `string` | 返回服务类名 |

#### search 方法参数

```javascript
route.search(start, end, options);
```

| 参数 | 类型 | 说明 |
|---|---|---|
| `start` | `Array<number> \| string` | 起点坐标 `[lng, lat]` 或地名字符串 |
| `end` | `Array<number> \| string` | 终点坐标 `[lng, lat]` 或地名字符串 |
| `options.waypoints` | `Array<Array<number>>` | 途经点坐标数组（仅 DrivingRoute 和 TransitRoute 支持） |

> 当 `start` 或 `end` 传入字符串时，服务会通过 `LocalSearch` 搜索 POI 候选点，需用户交互选择后再发起路线搜索。

---

## DrivingRoute 驾车路线规划

提供驾车路线规划功能，支持多种驾车策略和途经点设置。

### 构造函数

```javascript
const route = new mapvthree.services.DrivingRoute(options);
```

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `options.renderOptions` | `Object` | `{}` | 渲染配置，详见 [renderOptions 公共配置](#renderoptions-公共配置) |
| `options.policy` | `number` | `BMAP_DRIVING_POLICY_DEFAULT (0)` | 驾车策略，见下方策略常量表 |
| `options.alternatives` | `number` | `0` | 备选路线数量 |
| `options.enableTraffic` | `boolean` | `false` | 是否启用实时路况 |
| `options.apiSource` | `string` | 全局配置值 | API 数据源，`'baidu'` 或 `'tdt'` |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |
| `options.onMarkersSet` | `Function` | - | 标记点渲染完成回调 |
| `options.onPolylinesSet` | `Function` | - | 路线渲染完成回调 |
| `options.onResultsHtmlSet` | `Function` | - | 面板渲染完成回调 |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|---|---|---|---|
| `search(start, end, opts?)` | 起点、终点、选项 | `Promise<Object>` | 规划驾车路线 |
| `setPolicy(policy)` | `number` | `void` | 动态设置驾车策略 |
| `getStatus()` | - | `number` | 获取路线规划状态码 |
| `getResults()` | - | `Object \| null` | 获取包装后的规划结果 |
| `getPlan(index)` | `number` | `Object \| null` | 获取指定索引的路线方案 |
| `getNumPlans()` | - | `number` | 获取路线方案数量 |
| `clearResults()` | - | `void` | 清除结果及地图覆盖物 |

### 驾车策略常量

可通过 `DrivingRoute.BMAP_DRIVING_POLICY_*` 或全局 `window.BMAP_DRIVING_POLICY_*` 访问。

| 常量 | 值 | 说明 |
|---|---|---|
| `BMAP_DRIVING_POLICY_DEFAULT` | `0` | 默认策略（推荐路线） |
| `BMAP_DRIVING_POLICY_DISTANCE` | `2` | 距离最短 |
| `BMAP_DRIVING_POLICY_AVOID_HIGHWAYS` | `3` | 避开高速 |
| `BMAP_DRIVING_POLICY_FIRST_HIGHWAYS` | `4` | 优先高速 |
| `BMAP_DRIVING_POLICY_AVOID_CONGESTION` | `5` | 避开拥堵 |
| `BMAP_DRIVING_POLICY_AVOID_PAY` | `6` | 少收费 |
| `BMAP_DRIVING_POLICY_HIGHWAYS_AVOID_CONGESTION` | `7` | 高速优先且避开拥堵 |
| `BMAP_DRIVING_POLICY_AVOID_HIGHWAYS_CONGESTION` | `8` | 避开高速且避开拥堵 |
| `BMAP_DRIVING_POLICY_AVOID_CONGESTION_PAY` | `9` | 避开拥堵且少收费 |
| `BMAP_DRIVING_POLICY_AVOID_HIGHWAYS_CONGESTION_PAY` | `10` | 避开高速、避开拥堵且少收费 |
| `BMAP_DRIVING_POLICY_AVOID_HIGHWAYS_PAY` | `11` | 避开高速且少收费 |

### 示例

```javascript
const route = new mapvthree.services.DrivingRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
        panel: 'route-panel', // DOM 元素 ID
    },
    policy: BMAP_DRIVING_POLICY_AVOID_CONGESTION,
    onSearchComplete(result) {
        console.log('搜索完成', result);
    },
});

// 坐标方式
route.search([116.33297, 39.99932], [116.36181, 39.86875]);

// 带途经点
route.search(
    [116.33297, 39.99932],
    [116.36181, 39.86875],
    { waypoints: [[116.37, 39.91]] }
);

// 动态切换策略
route.setPolicy(BMAP_DRIVING_POLICY_DISTANCE);
```

---

## WalkingRoute 步行路线规划

提供步行路线规划功能，计算最优步行路径。仅支持百度地图数据源。

### 构造函数

```javascript
const route = new mapvthree.services.WalkingRoute(options);
```

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `options.renderOptions` | `Object` | `{}` | 渲染配置 |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |
| `options.onMarkersSet` | `Function` | - | 标记点渲染完成回调 |
| `options.onPolylinesSet` | `Function` | - | 路线渲染完成回调 |
| `options.onResultsHtmlSet` | `Function` | - | 面板渲染完成回调 |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|---|---|---|---|
| `search(start, end, opts?)` | 起点、终点、选项 | `Promise<Object>` | 规划步行路线 |
| `getStatus()` | - | `number` | 获取路线规划状态码 |
| `getResults()` | - | `Object \| null` | 获取包装后的规划结果 |
| `clearResults()` | - | `void` | 清除结果及地图覆盖物 |

### 示例

```javascript
const route = new mapvthree.services.WalkingRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
    },
});

const result = await route.search([116.404, 39.915], [116.414, 39.925]);
console.log('步行距离:', result.distance);
console.log('预计时间:', result.duration);
```

---

## RidingRoute 骑行路线规划

提供骑行路线规划功能，计算最优骑行路径。仅支持百度地图数据源。

### 构造函数

```javascript
const route = new mapvthree.services.RidingRoute(options);
```

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `options.renderOptions` | `Object` | `{}` | 渲染配置 |
| `options.policy` | `number` | `0` | 骑行策略，默认 `BMAP_ROUTING_TYPE_DEFAULT (0)` |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |
| `options.onMarkersSet` | `Function` | - | 标记点渲染完成回调 |
| `options.onPolylinesSet` | `Function` | - | 路线渲染完成回调 |
| `options.onResultsHtmlSet` | `Function` | - | 面板渲染完成回调 |

### 静态常量

| 常量 | 值 | 说明 |
|---|---|---|
| `RidingRoute.BMAP_ROUTING_TYPE_DEFAULT` | `0` | 默认骑行策略 |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|---|---|---|---|
| `search(start, end, opts?)` | 起点、终点、选项 | `Promise<Object>` | 规划骑行路线 |
| `getStatus()` | - | `number` | 获取路线规划状态码 |
| `getResults()` | - | `Object \| null` | 获取包装后的规划结果 |
| `clearResults()` | - | `void` | 清除结果及地图覆盖物 |

### 示例

```javascript
const route = new mapvthree.services.RidingRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
    },
});

const result = await route.search([116.404, 39.915], [116.414, 39.925]);
```

---

## TransitRoute 公交路线规划

提供公交路线规划功能，支持百度地图和天地图两种数据源。支持显示公交站点和换乘信息。

### 构造函数

```javascript
const route = new mapvthree.services.TransitRoute(options);
```

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `options.renderOptions` | `Object` | `{}` | 渲染配置 |
| `options.policy` | `number` | `0` | 公交策略，见下方策略常量表 |
| `options.apiSource` | `string` | 全局配置值 | API 数据源，`'baidu'` 或 `'tdt'` |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |
| `options.onMarkersSet` | `Function` | - | 标记点渲染完成回调 |
| `options.onPolylinesSet` | `Function` | - | 路线渲染完成回调 |
| `options.onResultsHtmlSet` | `Function` | - | 面板渲染完成回调 |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|---|---|---|---|
| `search(start, end, opts?)` | 起点、终点、选项 | `Promise<Object>` | 规划公交路线 |
| `setPolicy(policy)` | `number` | `void` | 动态设置公交策略 |
| `getStatus()` | - | `number` | 获取路线规划状态码 |
| `getResults()` | - | `Object \| null` | 获取包装后的规划结果 |
| `clearResults()` | - | `void` | 清除结果及地图覆盖物 |

### 公交策略常量

| 常量 | 值 | 说明 |
|---|---|---|
| `BMAP_TRANSIT_POLICY_LEAST_TIME` | `0` | 最少时间 |
| `BMAP_TRANSIT_POLICY_LEAST_TRANSFER` | `1` | 最少换乘 |
| `BMAP_TRANSIT_POLICY_LEAST_WALKING` | `2` | 最少步行 |
| `BMAP_TRANSIT_POLICY_AVOID_SUBWAYS` | `3` | 不乘地铁 |

### 示例

```javascript
const route = new mapvthree.services.TransitRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
    },
    policy: BMAP_TRANSIT_POLICY_LEAST_TRANSFER,
});

// 支持途经点
const result = await route.search(
    [116.33297, 39.99932],
    [116.36181, 39.86875],
    { waypoints: [[116.37, 39.91]] }
);
```

---

## 搜索结果对象

`search()` 方法返回的结果对象（以及 `getResults()` 返回的包装对象）包含以下结构：

### 原始结果字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `distance` | `number` | 路线总距离（米） |
| `duration` | `number` | 路线总耗时（秒） |
| `start` | `Object` | 起点信息，含 `point`（坐标）和 `title`（名称） |
| `end` | `Object` | 终点信息，含 `point`（坐标）和 `title`（名称） |
| `steps` | `Array` | 路线分段步骤数组 |
| `paths` | `Array<Array>` | 路线路径坐标数组（用于绘制折线） |
| `walkingPaths` | `Array<Array>` | 步行段路径坐标数组（公交路线中的步行段） |
| `waypoints` | `Array` | 途经点信息数组 |

### 包装结果方法

通过 `getResults()` 获取的包装对象额外提供以下方法：

| 方法 | 返回值 | 说明 |
|---|---|---|
| `getPlan(index)` | `Object \| null` | 获取指定索引的路线方案 |
| `getNumPlans()` | `number` | 获取方案数量（当前固定返回 1） |

### Plan 方案对象方法

通过 `getPlan(0)` 获取的方案对象提供以下方法：

| 方法 | 参数 | 返回值 | 说明 |
|---|---|---|---|
| `getDistance(format?)` | `boolean` | `string \| number` | 获取距离。`format=false` 返回米数，否则返回格式化字符串如 `"10.5公里"` |
| `getDuration(format?)` | `boolean` | `string \| number` | 获取耗时。`format=false` 返回秒数，否则返回格式化字符串如 `"25分钟"` |
| `getNumRoutes()` | - | `number` | 获取路线段数量 |
| `getRoute(index)` | `number` | `Object` | 获取指定索引的路线段 |
| `getStart()` | - | `Object` | 获取起点信息 |
| `getEnd()` | - | `Object` | 获取终点信息 |

#### 时间格式化规则

- **驾车/步行/骑行**（`nav` 模式）：天数超过 1 天时不显示分钟，如 `"1天2小时"`
- **公交**（`bustime` 模式）：四舍五入到最近的 5 分钟，最小显示 `"5分钟"`

---

## 状态码常量

通过 `getStatus()` 返回，也可通过全局 `window.BMAP_STATUS_*` 访问。

| 常量 | 值 | 说明 |
|---|---|---|
| `BMAP_STATUS_SUCCESS` | `0` | 检索成功 |
| `BMAP_STATUS_CITY_LIST` | `1` | 城市列表结果 |
| `BMAP_STATUS_UNKNOWN_LOCATION` | `2` | 位置结果未知 |
| `BMAP_STATUS_UNKNOWN_ROUTE` | `3` | 导航结果未知 |
| `BMAP_STATUS_INVALID_KEY` | `4` | 非法密钥 |
| `BMAP_STATUS_INVALID_REQUEST` | `5` | 非法请求 |
| `BMAP_STATUS_PERMISSION_DENIED` | `6` | 没有权限 |
| `BMAP_STATUS_SERVICE_UNAVAILABLE` | `7` | 服务不可用 |
| `BMAP_STATUS_TIMEOUT` | `8` | 超时 |

---

## 常见场景

### 1. 基础驾车路线规划并渲染到地图

```javascript
// 假设 engine 已初始化（需开启 enableAnimationLoop）
const route = new mapvthree.services.DrivingRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
    },
});

route.search([116.33297, 39.99932], [116.36181, 39.86875]);
```

### 2. 带面板的公交路线查询

```javascript
const route = new mapvthree.services.TransitRoute({
    renderOptions: {
        engine: engine,
        autoViewport: true,
        panel: document.getElementById('route-detail'),
    },
    policy: BMAP_TRANSIT_POLICY_LEAST_TRANSFER,
    onSearchComplete(result) {
        if (result) {
            const plan = result.getPlan(0);
            console.log('距离:', plan.getDistance());
            console.log('耗时:', plan.getDuration());
        }
    },
});

route.search([116.33297, 39.99932], [116.36181, 39.86875]);
```

### 3. 使用地名搜索

```javascript
const route = new mapvthree.services.WalkingRoute({
    renderOptions: { engine: engine, autoViewport: true },
});

// 传入字符串会触发 POI 搜索，用户交互选择后再发起路线规划
route.search('北京大学', '清华大学');
```

### 4. 动态切换策略后重新搜索

```javascript
const route = new mapvthree.services.DrivingRoute({
    renderOptions: { engine: engine, autoViewport: true },
});

// 初始搜索
await route.search([116.33, 39.99], [116.36, 39.86]);

// 切换为距离最短策略后重新搜索
route.setPolicy(BMAP_DRIVING_POLICY_DISTANCE);
await route.search([116.33, 39.99], [116.36, 39.86]);
```

---

## 注意事项

1. **Engine 实例必须开启动画循环**：使用路线服务前，需确保 Engine 创建时设置 `rendering.enableAnimationLoop: true`，否则路线渲染可能不会更新。

2. **坐标系**：使用百度数据源时坐标系为 bd09；使用天地图数据源时坐标系为 CGCS2000。请确保传入的坐标与当前数据源的坐标系一致。

3. **途经点支持**：只有 `DrivingRoute`（驾车）和 `TransitRoute`（公交）支持通过 `options.waypoints` 设置途经点，`WalkingRoute` 和 `RidingRoute` 不支持。

4. **数据源限制**：`WalkingRoute`（步行）和 `RidingRoute`（骑行）仅支持百度地图数据源，不支持天地图。

5. **方案数量**：当前版本搜索结果固定返回 1 个方案（`getNumPlans()` 返回 1），`getPlan(index)` 仅 `index=0` 有效。

6. **Panel 滚动冲突**：设置 `panel` 后，面板容器会自动阻止鼠标滚轮事件冒泡到地图，避免滚动面板时误触发地图缩放。

7. **clearResults 联动**：调用 `clearResults()` 会同时清除搜索结果、地图上的折线和标记点，以及 Panel 面板内容。

8. **字符串输入**：当起点或终点传入字符串（地名）时，需要 Engine 实例支持，服务会通过 `LocalSearch` 搜索 POI 候选点并展示交互选择界面。
