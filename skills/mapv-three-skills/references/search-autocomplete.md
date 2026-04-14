# LocalSearch 本地搜索与 AutoComplete 自动完成

## 概述

`LocalSearch` 和 `AutoComplete` 是 mapv-three 提供的 LBS（基于位置的服务）组件，分别用于 POI 检索和输入提示自动补全。两者均支持百度地图 API 数据源，`LocalSearch` 还额外支持天地图数据源。

> **前置条件：** 使用百度数据源时，必须先设置 `mapvthree.BaiduMapConfig.ak = 'YOUR_BAIDU_AK'`，否则会报 `BaiduMapConfig.ak is not set` 错误。

---

## 一、LocalSearch 本地搜索

### 1.1 引入方式

```javascript
import * as mapvthree from '@baidu/mapv-three';

const localSearch = new mapvthree.services.LocalSearch(options);
```

### 1.2 构造函数

```javascript
new mapvthree.services.LocalSearch(options)
```

#### 参数表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.pageCapacity` | `number` | `10` | 每页结果数，范围 1-100 |
| `options.pageNum` | `number` | `0` | 起始页码，从 0 开始 |
| `options.apiSource` | `string` | `'baidu'` | API 数据源，可选 `services.API_SOURCE_BAIDU`（`'baidu'`）或 `services.API_SOURCE_TIANDITU`（`'tdt'`） |
| `options.renderOptions` | `Object` | `{}` | 渲染配置对象 |
| `options.renderOptions.engine` | `Engine` | - | 地图引擎实例，用于在地图上渲染搜索结果 |
| `options.renderOptions.autoViewport` | `boolean` | `true` | 是否自动调整地图视角以包含所有搜索结果 |
| `options.renderOptions.enableRender` | `boolean` | `true` | 是否启用自动渲染（Marker + Popup），设为 `false` 可禁用渲染 |

#### 静态常量

| 常量 | 值 | 说明 |
|------|----|------|
| `LocalSearch.DEFAULT_PAGE_CAPACITY` | `10` | 默认每页结果数 |
| `LocalSearch.MIN_PAGE_CAPACITY` | `1` | 最小页容量 |
| `LocalSearch.MAX_PAGE_CAPACITY` | `100` | 最大页容量 |
| `LocalSearch.DEFAULT_RADIUS` | `2000` | 默认检索半径（米） |
| `LocalSearch.MAX_RADIUS` | `100000` | 最大检索半径（米） |
| `LocalSearch.DEFAULT_PAGE_NUM` | `0` | 默认页码 |

### 1.3 方法列表

#### search(keyword, opts)

关键词搜索 POI。

| 参数 | 类型 | 说明 |
|------|------|------|
| `keyword` | `string` | 搜索关键词 |
| `opts.pageCapacity` | `number` | 可选，每页结果数 |
| `opts.pageNum` | `number` | 可选，页码 |

- **返回值**: `Promise<Object>` -- 搜索结果对象

```javascript
const result = await localSearch.search('餐厅', {
    pageCapacity: 20,
    pageNum: 0,
});
console.log('找到', result.pois.length, '个餐厅');
```

#### searchInBounds(keyword, bounds, opts)

在指定矩形范围内搜索 POI。

| 参数 | 类型 | 说明 |
|------|------|------|
| `keyword` | `string \| Array<string>` | 搜索关键词，支持多关键字 |
| `bounds` | `Object` | 搜索范围 |
| `bounds.sw` | `Array<number>` | 西南角坐标 `[lng, lat]` |
| `bounds.ne` | `Array<number>` | 东北角坐标 `[lng, lat]` |
| `opts.pageCapacity` | `number` | 可选，每页结果数 |
| `opts.pageNum` | `number` | 可选，页码 |

- **返回值**: `Promise<Object>`

```javascript
const result = await localSearch.searchInBounds('餐厅', {
    sw: [116.274625, 39.961627],
    ne: [116.367474, 39.988609],
});
```

#### searchNearby(keyword, center, radius, opts)

以指定中心点和半径进行周边检索。

| 参数 | 类型 | 说明 |
|------|------|------|
| `keyword` | `string \| Array<string>` | 搜索关键词 |
| `center` | `Array<number>` | 中心点坐标 `[lng, lat]` |
| `radius` | `number` | 检索半径（米），默认 2000，最大 100000 |
| `opts.pageCapacity` | `number` | 可选，每页结果数 |
| `opts.pageNum` | `number` | 可选，页码 |

- **返回值**: `Promise<Object>`

```javascript
const result = await localSearch.searchNearby('餐厅', [116.404, 39.915], 2000);
```

#### gotoPage(pageNum)

跳转到指定页码并重新搜索。需要先执行一次 `search` 才能使用。

| 参数 | 类型 | 说明 |
|------|------|------|
| `pageNum` | `number` | 目标页码 |

#### setPageCapacity(cap)

设置每页结果数量。

| 参数 | 类型 | 说明 |
|------|------|------|
| `cap` | `number` | 页容量，范围 1-100，超出范围则重置为默认值 10 |

#### getPageCapacity()

获取当前每页容量。返回 `number`。

#### setPageNum(num)

设置当前页码。

| 参数 | 类型 | 说明 |
|------|------|------|
| `num` | `number` | 页码，从 0 开始，负数会被重置为 0 |

#### getPageNum()

获取当前页码。返回 `number`。

#### getResult()

获取最近一次搜索结果对象。返回 `Object | null`。

#### setPopupByIndex(index)

在指定索引的标注点处显示弹窗（需要启用渲染功能）。

| 参数 | 类型 | 说明 |
|------|------|------|
| `index` | `number` | POI 在结果列表中的索引 |

#### clearResult()

清除当前搜索结果。

### 1.4 搜索结果对象结构

`search` / `searchInBounds` / `searchNearby` 返回的 Promise resolve 值：

```javascript
{
    keyword: '餐厅',          // 搜索关键词
    total: 100,               // 总结果数
    pois: [                   // 当前页 POI 列表
        {
            title: 'XX餐厅',   // POI 名称
            address: '北京市...', // POI 地址
            point: {x, y},     // 坐标（Vector3 格式）
            // ...其他字段
        }
    ]
}
```

### 1.5 使用示例

#### 基本搜索

```javascript
// 假设 engine 已初始化
const localSearch = new mapvthree.services.LocalSearch({
    renderOptions: {
        engine,
        autoViewport: true,
    },
    pageNum: 0,
});

localSearch.search('北京邮电大学').then(result => {
    console.log(result);
});
```

#### 分页搜索

```javascript
const localSearch = new mapvthree.services.LocalSearch({
    pageCapacity: 20,
    renderOptions: { engine, autoViewport: true },
});

// 第一页
await localSearch.search('酒店');

// 跳转到第二页
localSearch.gotoPage(1);
```

#### 使用天地图数据源

```javascript
const localSearch = new mapvthree.services.LocalSearch({
    apiSource: mapvthree.services.API_SOURCE_TIANDITU,
    pageCapacity: 10,
    renderOptions: {
        engine,
        autoViewport: false,
    },
});
const result = await localSearch.search('酒店');
```

---

## 二、AutoComplete 自动完成

### 2.1 引入方式

```javascript
import * as mapvthree from '@baidu/mapv-three';

const autoComplete = new mapvthree.services.AutoComplete(options);
```

### 2.2 构造函数

```javascript
new mapvthree.services.AutoComplete(options)
```

#### 参数表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.input` | `string \| HTMLElement` | `null` | 输入框 DOM 元素或其 ID |
| `options.types` | `Array<string>` | `[]` | 搜索类型过滤，如 `['ALL']`、`['CITY']` |
| `options.location` | `Array<number>` | `null` | 搜索区域限制坐标 `[lng, lat]` |
| `options.maxResults` | `number` | `10` | 最大结果数量 |
| `options.debounceTime` | `number` | `100` | 输入防抖时间（毫秒） |
| `options.apiSource` | `string` | `'baidu'` | API 数据源，目前仅支持 `services.API_SOURCE_BAIDU` |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |
| `options.onError` | `Function` | - | 错误回调 |
| `options.onNeedData` | `Function` | - | 需要数据回调 |

### 2.3 方法列表

#### search(keyword)

手动触发搜索。

| 参数 | 类型 | 说明 |
|------|------|------|
| `keyword` | `string` | 搜索关键词 |

- **返回值**: `AutoComplete` 实例（支持链式调用）

```javascript
autoComplete.search('北京');
```

#### setTypes(types)

动态设置搜索类型过滤。

| 参数 | 类型 | 说明 |
|------|------|------|
| `types` | `Array<string> \| string` | 搜索类型，如 `['CITY', 'POI']` 或 `'CITY'` |

- **返回值**: `AutoComplete` 实例

```javascript
autoComplete.setTypes(['CITY']);
```

#### setPickValue(value)

设置选中值，避免触发新的搜索请求。常用于键盘上下键选择时更新输入框值。

| 参数 | 类型 | 说明 |
|------|------|------|
| `value` | `string` | 选中的文本值 |

#### getResults()

获取当前搜索结果。

- **返回值**: `Object | null`

#### dispose()

销毁组件，释放资源（移除 DOM 元素、解除事件监听等）。

```javascript
autoComplete.dispose();
```

#### addEventListener(event, callback)

添加事件监听器。

#### removeEventListener(event, callback)

移除事件监听器。

### 2.4 事件列表

| 事件名 | 触发时机 | 回调参数 |
|--------|----------|----------|
| `searchComplete` | 搜索完成并返回结果时 | `{ results, keyword, target }` |
| `error` | 搜索出错时 | `{ message, target }` |
| `needData` | 需要加载数据时 | `{ keyword, target }` |

#### 事件回调数据结构

`searchComplete` 事件的 `results` 对象：

```javascript
{
    keyword: '北京',
    total: 5,
    pois: [
        {
            province: '北京市',
            city: '北京市',
            district: '海淀区',
            street: '中关村大街',
            business: 'XX大厦',
            address: '北京市海淀区...',
            name: 'XX地点',
            location: { lng, lat },
        },
        // ...
    ]
}
```

### 2.5 使用示例

#### 基本用法（配合 React）

```jsx
import React from 'react';
import * as mapvthree from '@baidu/mapv-three';

class SearchComponent extends React.Component {
    searchInputRef = React.createRef();

    componentDidMount() {
        this.autoComplete = new mapvthree.services.AutoComplete({
            apiSource: mapvthree.services.API_SOURCE_BAIDU,
            types: 'CITY',
            input: this.searchInputRef.current,
        });

        this.autoComplete.addEventListener('searchComplete', e => {
            const pois = e.results?.pois || [];
            console.log('搜索建议:', pois);
        });

        this.autoComplete.addEventListener('error', e => {
            console.error('搜索错误:', e);
        });
    }

    componentWillUnmount() {
        this.autoComplete?.dispose();
    }

    handleInput = (e) => {
        const value = e.target.value;
        if (value.trim().length >= 2) {
            this.autoComplete.search(value.trim());
        }
    };

    render() {
        return (
            <input
                ref={this.searchInputRef}
                onChange={this.handleInput}
                placeholder="搜索地点..."
            />
        );
    }
}
```

#### 键盘导航（上下键选择 + 回车确认）

```javascript
// 上下键切换时，使用 setPickValue 避免触发新搜索
autoComplete.setPickValue(selectedAddress);

// 回车确认后，可通过 results 获取 POI 详情
const results = autoComplete.getResults();
```

---

## 三、BMapGL 兼容适配器

mapv-three 还提供了与百度地图 JavaScript API（BMapGL）兼容的适配器层，位于 `src/adapters/bmap/services/` 下。如果你从 BMapGL 迁移过来，可以使用以下兼容接口。

### 3.1 BMapGL.LocalSearch 适配器

#### 构造函数

```javascript
const localSearch = new BMapGL.LocalSearch(map, {
    renderOptions: {
        map: map,
        panel: document.getElementById('results'),
        autoViewport: true,
        selectFirstResult: true,
        highlight: true,
    },
    pageCapacity: 20,
    onSearchComplete: function(results) { ... },
});
```

##### 适配器专属参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `location` | `Map \| Point \| string` | - | 第一个构造参数，地图实例、坐标点或城市名 |
| `options.renderOptions.map` | `Map` | `null` | 地图实例，传入后启用地图渲染 |
| `options.renderOptions.panel` | `HTMLElement` | `null` | 结果列表面板 DOM 元素 |
| `options.renderOptions.selectFirstResult` | `boolean` | `true` | 是否自动选中第一个结果 |
| `options.renderOptions.highlight` | `boolean` | `true` | 是否高亮显示结果 |
| `options.onSearchComplete` | `Function` | `null` | 搜索完成回调 |

#### 适配器额外方法

| 方法 | 说明 |
|------|------|
| `search(keyword, { callback })` | 回调风格搜索，兼容 BMapGL 用法 |
| `searchInBounds(keyword, bounds, opts)` | 范围检索，`bounds` 为 BMapGL Bounds 对象 |
| `searchNearby(keyword, center, radius, opts)` | 周边检索，`center` 支持 Point / string / LocalResultPoi |
| `getStatus()` | 获取状态码（`BMAP_STATUS_SUCCESS` 等） |
| `getCurrentNumPois()` | 当前页结果数量 |
| `getNumPois()` | 总结果数量 |
| `getResults()` | 获取结果对象 |
| `getPoi(index)` | 按索引获取 POI |
| `getPois()` | 获取所有 POI 数组 |
| `getPageIndex()` | 当前页码 |
| `gotoPage(pageNum)` | 跳转页码 |
| `clearResults()` | 清除结果并移除地图标记 |
| `enableFirstResultSelection()` | 开启自动选中第一个结果 |
| `disableFirstResultSelection()` | 关闭自动选中第一个结果 |

#### 适配器结果对象方法

`search` 回调中的 `results` 对象提供以下方法：

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `getCurrentNumPois()` | `number` | 当前页 POI 数量 |
| `getPoi(index)` | `Object` | 获取指定索引的 POI |
| `getNumPois()` | `number` | 总 POI 数量 |
| `getPageIndex()` | `number` | 当前页码 |
| `getKeyword()` | `string` | 当前搜索关键词 |

### 3.2 BMapGL.Autocomplete 适配器

#### 构造函数

```javascript
const ac = new BMapGL.Autocomplete({
    input: 'searchInput',
    location: map,
    types: [],
    showSuggestion: true,
    baseDom: null,
    onSearchComplete: function(result) { ... },
});
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `options.input` | `string \| HTMLElement` | `null` | 输入框 ID 或 DOM 元素 |
| `options.location` | `Map \| Point \| Array` | `null` | 位置限制，支持 Map 实例、Point 对象、坐标数组 |
| `options.types` | `Array` | `[]` | 搜索类型过滤 |
| `options.showSuggestion` | `boolean` | `true` | 是否显示内置建议下拉列表 |
| `options.baseDom` | `string \| HTMLElement` | `null` | 下拉列表定位的基准 DOM |
| `options.onSearchComplete` | `Function` | - | 搜索完成回调 |

#### 适配器方法

| 方法 | 说明 |
|------|------|
| `search(word)` | 手动触发搜索 |
| `show()` | 显示建议下拉列表 |
| `hide()` | 隐藏建议下拉列表 |
| `setTypes(types)` | 设置搜索类型 |
| `setLocation(location, callback)` | 设置位置限制 |
| `setInputValue(value)` | 设置输入框值 |
| `getResults()` | 获取结果，返回 `AutocompleteResult` 实例 |
| `setSearchCompleteCallback(callback)` | 设置搜索完成回调 |
| `addEventListener(event, callback)` | 添加事件监听 |
| `removeEventListener(event, callback)` | 移除事件监听 |
| `dispose()` | 销毁实例 |

#### 适配器事件

| 事件名 | 触发时机 | 回调参数 |
|--------|----------|----------|
| `onconfirm` | 用户点击选中某个建议项 | `{ item: { index, value } }` |
| `onhighlight` | 鼠标悬停建议项时 | `{ fromitem: { index, value }, toitem: { index, value } }` |
| `onhide` | 建议列表隐藏时 | `{}` |

#### AutocompleteResult 类

`getResults()` 返回的对象：

| 方法/属性 | 类型 | 说明 |
|-----------|------|------|
| `keyword` | `string` | 搜索关键词 |
| `getPoi(index)` | `Function` | 获取指定索引的 POI |
| `getNumPois()` | `Function` | 获取 POI 总数 |

---

## 四、常见场景

### 场景 1: 搜索 + 地图标注

```javascript
const localSearch = new mapvthree.services.LocalSearch({
    renderOptions: {
        engine,
        autoViewport: true,   // 自动调整视角
        enableRender: true,   // 自动渲染 Marker 和 Popup
    },
    pageCapacity: 10,
});

const result = await localSearch.search('咖啡厅');
// 点击第 3 个标注点显示弹窗
localSearch.setPopupByIndex(2);
```

### 场景 2: 仅获取数据不渲染

```javascript
const localSearch = new mapvthree.services.LocalSearch({
    renderOptions: {
        engine,
        enableRender: false,  // 不渲染，仅获取数据
    },
});

const result = await localSearch.search('银行');
result.pois.forEach(poi => {
    console.log(poi.title, poi.address, poi.point);
});
```

### 场景 3: 搜索框自动补全 + 选中后搜索

```javascript
const autoComplete = new mapvthree.services.AutoComplete({
    input: document.getElementById('searchInput'),
    types: ['CITY'],
    debounceTime: 300,
});

autoComplete.addEventListener('searchComplete', e => {
    // 渲染建议列表
    renderSuggestions(e.results.pois);
});

// 用户选中后，用 LocalSearch 做详细搜索
function onSelect(poi) {
    localSearch.search(poi.name);
}
```

### 场景 4: 周边搜索

```javascript
const result = await localSearch.searchNearby(
    '加油站',
    [116.404, 39.915],  // 天安门坐标
    5000                 // 5公里半径
);
```

---

## 五、注意事项

1. **数据源差异**: `LocalSearch` 支持百度地图（`API_SOURCE_BAIDU`）和天地图（`API_SOURCE_TIANDITU`）两种数据源；`AutoComplete` 目前仅支持百度地图数据源。

2. **异步 API**: mapvthree 核心层的 `LocalSearch` 方法（`search`、`searchInBounds`、`searchNearby`）均返回 `Promise`；BMapGL 适配器层则通过回调函数传递结果。

3. **页容量限制**: `pageCapacity` 取值范围为 1-100，超出范围会被重置为默认值 10。

4. **渲染控制**: 设置 `renderOptions.enableRender = false` 可以禁用自动渲染，适用于需要自定义 UI 展示的场景。即使禁用渲染，仍需传入 `engine` 以获取城市信息等功能。

5. **防抖处理**: `AutoComplete` 内置防抖机制（默认 100ms），但在实际应用中建议根据需要设置更大的 `debounceTime`（如 200-300ms），或在业务层自行实现防抖。

6. **资源释放**: `AutoComplete` 使用完毕后务必调用 `dispose()` 方法释放 DOM 资源和事件监听器，避免内存泄漏。

7. **坐标系**: mapvthree 核心层返回的 POI 坐标为 `{x, y}` 格式（Vector3）；BMapGL 适配器层会自动将其转换为 `{lng, lat}` 格式的 `Point` 对象。

8. **关键词输入**: `AutoComplete` 建议在关键词长度 >= 2 个字符时才触发搜索，避免无效请求。
