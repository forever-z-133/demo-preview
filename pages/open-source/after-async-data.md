# after-async-data

在获得某结果前进行排队，等获取结果后再运行。

## 安装

```
npm i -S after-async-data
```
https://www.npmjs.com/package/after-async-data

## 如何使用

> #### 比如在进入应用后直接调用了多个接口，但后来需在之前加上 getToken 方法

```js
import AfterAsyncData from "~after-async-data";

// 伪·业务代码
const getToken = (params, next) => setTimeout(() => next("token"), 500);
let ajax = (params, success, fail) => setTimeout(() => success(params), 500);
const getUserInfo = (callback) => ajax(null, callback);
const getIndexData = (callback) => ajax(null, callback);

// 先注册前置函数
AfterAsyncData().add("get token before ajax", getToken);
// 包装原方法，把原逻辑置于获取结果之后
const _ajax = ajax;
function ajax(params, success, fail) {
  AfterAsyncData().get("get token before ajax", (token) => {
    localStorage.setItem("token", token);
    _ajax(params, success, fail);
  });
}
// 再调用业务代码时，会等到获取 token 后才运行
getUserInfo();
getIndexData();
```

> #### 用于缓存一点异步数据

```js
let getLargeData = (next) => setTimeout(() => next({}), 500);
AfterAsyncData().add("cache large data", getLargeData);
const _getLargeData = getLargeData;
function getLargeData(callback) {
  AfterAsyncData().get("cache large data", (data) => {
    _getLargeData(callback);
  });
}
// 业务代码运行后，data 会存入插件缓存，下次再调用会先取缓存
// 使用 delete AfterAsyncData().dataMap['cache large data'] 可清掉缓存
getLargeData();
```

> #### 结合上述两者，可处理很多等待数据的场景

比如在 app.js 获取权限，在 index.js 获取完数据后就要用，  
但这两个异步谁先谁后其实并不确定，又因为跨页面而无法 Promise.all,  
则可通过本 `after-async-data` 插件来进行等待。

再比如，在某个节点前只接收操作但不反馈，等满足条件后再筛选操作再运行。

当然，本案例实现得可能并不甚妙，欢迎 issuse 和 pull request 交流。
