# 前端卡顿问题优化导览

在 [前端资源优化导览](/articles/topic/optimization-of-sources.md) 中，减少响应时长板块就涉及了一点些性能优化的方向，本文对此进行更多补充。

归咎卡顿原因，我是如此划分的：

* 网络原因
* 浏览器机制原因
* 代码原因

## 网络原因的卡顿

举些例子，有些是前端问题、有些是后端问题，你要说加带宽解决大多问题倒也没错，但方案还是多多益善。

* 打开页面后长时间白屏
* 页面呈现了，但图片视频长时间未加载完
* 跳页时，新页面还未加载完
* 跳页或点击后，接口还未返回

使用 `LCP`、`FCP`、`TTFB`、`TBT`  等指标，https://web.dev/articles/fcp?hl=zh-cn

在 [前端资源优化导览](/articles/topic/optimization-of-sources.md) 中都提过了，主要方向：

* 资源方面：压缩、合并资源、资源选型
* 打包方面：减主包、异步化
* 网络方面：加带宽、加缓存、加压缩、加预解析、加CDN

## 浏览器原因的卡顿

#### DOM 数量太多

比如 select 的 option 过多。

#### offsetTop 等触发重排

https://www.51cto.com/article/780992.html

#### CSS 卡顿

比如动画开销：https://forever-z-133.github.io/demos/others/funny/css-stuck.html

## 代码原因的卡顿

#### 前端框架性能优化

比如 react 的：https://juejin.cn/post/6935584878071119885#heading-16
比如 vue 的：https://juejin.cn/post/6922641008106668045

#### 异步优化

主线程占用过多，导致页面卡顿
