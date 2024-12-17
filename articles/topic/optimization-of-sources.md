# 前端资源优化导览

所有的优化都应有指标，前端的资源优化、打包优化也是如此：

* 打包时长
* 打包体积
* 访问时长
* 响应时长

但它们并不是绝对的指标，比如：打包总体积变小核心包变大、访问时长会变大；分包后访问时长变小、但响应时长会变大等。<br />
所以，盲目追求优化数值并不能带来好的效果，需要根据实际情况进行取舍。

本文仅梳理优化手段，以 `vite` 项目为例，具体采用哪些手段需按你业务的实际情况来。

## 减少打包时长

普遍脚手架打包结束时都会打印打包时长，

#### externals 提取依赖

比如 `vue.min.js` 或 `axios.min.js` 这些需要全量引入又不需要打包的依赖，可以提取出来，以提高打包速率。

再比如 `vconsole` 等非业务型依赖，也不应打包。

#### 缓存未变更的模块

比如 `DllPlugin` 或 `cache-loader` 等。

#### 多线程打包

比如 `happy-pack` 或 `thread-loader` 等。

或者 `terser-webpack-plugin` 等插件自带 `parallel` 参数配置。

#### 编译器选型

比如 `swc` 或 `esbuild` 等。

#### 关闭 sourceMap

正式环境可以不导出 `sourceMap`。

#### 图片等资源不揉进代码中

没必要 `import Icon from '@/assets/icon.svg'`，直接使用 `src="/public/icon.svg"` 更佳。

但也分情况，比如你想做雪碧图或小图片 `base64` 化。

## 减少打包体积

首先要学会利用 `vite-plugin-visualizer` 等脚手架插件分析依赖体积。

#### 按需加载

比如 `lodash` 会全量引入，但并未用到所有 `api`，因此使用 `lodash-es` 来 `tree-shaking`。

再比如 `moment` 的语言包会占用大量体积，实际要用到的语言去单独引入更佳。

再比如 `element-plus` 等组件库的 [自动导入方案](https://element-plus.org/zh-CN/guide/quickstart.html#%E8%87%AA%E5%8A%A8%E5%AF%BC%E5%85%A5-%E6%8E%A8%E8%8D%90)

再比如分包都引入了包被重复打包多次，提取公共代码。*小程序主包体积比分包的珍贵，可另行考虑*

再比如 `import * as x from 'x'` 之类的 `echarts` 和 `three` 被错误书写而全量引入。

再比如 `import.meta.glob` 等的错误使用，而将未使用的模块或资源也打包进来。

#### 依赖选型

比如 `dayjs` 要比 `moment` 小很多

#### 压缩资源

压缩 `html`、`css`、`js`、`svg`、`json` 代码。

比如代码 `compression-webpack-plugin` 等插件。

比如代码 `terser-webpack-plugin` 等插件自带 `minify` 参数配置。

再比如图片 `image-webpack-loader` 等插件自带 `optimization` 参数配置。

## 减少访问时长

首先你需要熟悉从网址到显示这个过程都经历了哪些步骤。

#### 预加载

给 `<script>` 打上 `preload` `prefetch` 标签，让浏览器预加载。

#### 延迟加载

给 `<script>` 打上 `async` `defer` 标签，让浏览器异步加载。

#### 懒加载/异步加载

比如异步路由 `import(./page.vue)`，或者异步组件 `const Component = () => import('./page.vue')`。

或者，用到的时机才 `document.body.append(script)` 注入。

#### 资源缓存

给请求配上缓存相关的 `request headers`

* 比如 `Expires` 和 `Cache-Control` 可以去设定过期时间。*也即强缓存，后续两条为协商缓存*
* 比如 `Last-Modified` 和 `If-Modified-Since` 可以对比变更时间来判断缓存是否更新。
* 比如 `Etag` 和 `If-None-Match` 可以对比变更文件的 `md5` 值来判断缓存是否更新。

或者通过 `service-worker` 来缓存资源。

#### Gzip 压缩

给请求配上压缩相关的 `request headers`

#### 服务器端优化

增加宽带，增加 `CDN` 服务，减少 `cookie` 使用。

## 减少响应时长

牵扯到 `LCP`、`FCP` 等视觉方面的指标，以及 `FID`、`CLS` 等交互方面的指标，

#### 分包策略

因为主包不加载完成，后续脚本就无法执行，所以要控制主包体积。

比如全量引入的 `jquery` 核心依赖或 `element-plus` 组件库，都会卡住业务代码的执行。

或者通过 `optimization.splitChunks` 来避免主包体积过大。

再比如，抽离 `CSS` 代码到独立文件。

#### 减少请求次数

牵扯到资源数量，比如小体积包过多都浪费大量的时间在三次握手上。

且浏览器并行下载速率有限，请求太多会卡住后续请求。

再比如雪碧图，小图片 `base64` 化，等等。

#### 减少路由拦截

比如在 `router.beforeEach` 判断用户权限要等待接口返回就阻塞了页面访问，可以改为先跳页再说。

#### 组件缓存

一般是 `<keep-alive>` 或给元素设置 `key` 或 `ref`，

比如，给异步脚本加 `id` 避免重复请求。

#### 减少错误样式使用

比如大量的 `*` 或者 `!important` 样式，会阻塞页面渲染。

#### 减少 DOM 数量

当 `DOM` 数量过多会使页面卡顿。

#### 避免频繁重排重绘

比如避免使用 `offsetWidth` 触发重排，大量 `display: none` 等。

有频繁重排的元素，进行脱离文档流处理。

加上节流防抖机制。

优化动画，有动画的元素加 `transform: translate3d(0, 0, 0)`。

#### 弹窗内容延迟渲染

比如弹窗内容存在渲染、计算耗时大的组件，可以先 `v-if` 等待弹窗开启后再渲染。

#### 避免频繁组件更新

比如 `shouldComponentUpdate` `PureComponent` `React.memo` 等。

再比如 `shallowRef`。

再比如拖拽等交互，使用 `DOM` 操作临时取代数据驱动。
