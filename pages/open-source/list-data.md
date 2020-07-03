# 分页数据请求

可以用于 jquery vue react 等各类项目中，可简化较多重复代码，使分页数据请求更便于管理和使用。

## 如何安装

```js
npm i -S list-data
```
https://www.npmjs.com/package/list-data

## 基础使用

以 vue 开发实践举例，只需将各属性方法绑定即可。

```js
new Vue({
  template: `
    <div class="some-list">
      <div class="list-box">
        <block v-for="(item, index) in someList.data" :key="index">
          <div class="item>{{ index }}</div>
        </block>
      </div>
      <div class="list-status">
        <span v-if="someList.status === 'LOADING'">加载中</span>
        <span v-if="someList.status === 'NODATA'">没有数据</span>
        <span v-if="someList.status === 'EMPTY'">没有更多数据了</span>
      </div>
    </div>
  `,
  data() {
    const ListManager = new ListData();
    return {
      ListManager,
      someList: ListManager.list,
    };
  },
  created() {
    this.initSomeList();
    this.ListManager.refresh();
  },
  methods: {
    initSomeList() {
      this.ListManager.ajax = this.someAjax.bind(this);
    },
    someAjax(params, calllback) {
      this.$axios.get("https://some-url", params, callback);
    },
    onReachBottom() {
      // 触底加载 或 换页加载
      // 且当数据加载中或无数据或全加载完时自动不会继续请求
      this.ListManager.update();
    },
    onPullDownRefresh() {
      // 下拉刷新 或 更换tab时
      this.ListManager.refresh();
    },
    changePage(page = 1) {
      // 表格类的应用，跳页功能
      this.ListManager.list.page = page;
      this.ListManager.refresh();
    },
  },
});
```

react 或 flutter 需使用 setState，所以只需在 initSomeList 再加上绑定即可。

```js
this.ListManager.stateChange = (state) => this.setState({ someList: state });
```

## 扩展使用

```js
// ListData 构造器
class ListData {
  list: {
    data: [],
    size: 10,
    page: 1,
    status: 'WAIT' // WAIT LOADING NODATA EMPTY
  },
  // 用于：1. 统一修改 size 初始值；2. 统一添加更多联动项比如 total
  beforeInit(config, list) {},
  // 用于：1. 统一 size 改 pageSize；2. 统一添加 loading
  beforeAjax(params) {},
  // 每个实例应该都要改此方法，且需注意 bind(this) 哟
  ajax(params, callback) {},
  // 用于：1. 统一 res.result 结构；2. 统一取消加载图；3. 统一报错弹窗
  beforePostData(res, params) {},
  // 用于个性化结果调整，比如返回 total
  postData(res, params) {},
  // 用于个性化列表数据调整，比如每项加上已选择
  convert(newData) {},
  // 本次过程全部完成
  finish(data, list) {},
  // --- 三个常调用的方法
  refresh(params) {},
  update(params) {},
  stateChange(list, key, value) {}
}
```

可进行类似以下操作的拓展：

```js
ListData.prototype.beforeInit = function(config, list) {
  config.size = 15;
  list.total = 0;
};
ListManager.beforeAjax = params => {
  Toast.loading('加载中');
  const { page, size } = params;
  return { pageNo: page, pageSize: size, type: '1' };
};
ListManager.beforePostData = res => {
  Toast.hide();
  if (res.status !==) {
    Toast.error(res.errMsg || '请求错误');
    return true; // 返回 true 则不继续运行
  }
};
ListManager.postData = res => {
  const data = res.result.list || [];
  this.total = res.result.total; // 获取数据后用在别处
  res.result = data;  // 注意：列表数据会从 res.result 中取
}
ListManager.convert = data => {
  data.forEach(item => (item.flag = true));
}
```

其中，你也可以利用 ListData.prototype 做二次封装，  
这样就不用每次 new ListData 时都修改实例方法了。

## 使用 TypeScript

```js
import ListData from "~list-data/index.ts";
```
