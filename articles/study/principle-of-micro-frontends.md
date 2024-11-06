# 微前端原理与实现整理

自 **微服务** 这个概念提出以来，将业务拆小独立然后由统一网关调用就成了项目优化的大趋势，微前端也是如此。<br />
另一个作用是整合历史系统，新旧两套系统同时运行但只提供单一入口。

目前的微前端框架一般都具有以下三个特点：

- **技术栈无关**：主框架不限制接入应用的技术栈，子应用具备完全自主权。
- **独立性强**：独立开发、独立部署，子应用仓库独立。
- **状态隔离**：运行时每个子应用之间状态隔离。

撇开服务端反向代理这种丢失 SPA 体验的方案，微前端可以简单划分为三个流派：

- 古早的 iframe 方案
- 2018 年在 github 上开源的 `single-spa`
- 2019 年问世的 `qiankun`
- 2020 年 webpack5 推出的 `Module Federation`

## iframe 方案

iframe 优点:

- iframe 自带的样式、环境隔离机制使得它具备天然的沙盒机制。
- 嵌入子应用比较简单。

iframe 缺点：

- iframe 之间的跳转是无效的，刷新页面无法保存状态。
- history 路由状态丢失，刷新会返回首页。
- 模态窗的背景是无法覆盖到整个应用。
- 主应用劫持快捷键操作，事件冒泡不穿透到主文档树上。
- 通信非常困难，只能通过 postMessage 传递序列化消息。
- iframe 应用加载失败，内容发生错误主应用无法感知。
- 白屏时间长，重复资源不好复用。

## single-spa 方案

作为第一个微前端框架，在当年是概念较多的存在，但如今来看已然不够先进，此处仅整理其实现思路不做深究。

```js
import { registerApplication, start } from 'single-spa';

registerApplication({
  name: 'app1',
  app: () => import('src/app1/main.js'),
  activeWhen: ['/app1', (location) => location.pathname.startsWith('/some/other/path')],
  customProps: { some: 'value' },
});

start();
```

## qiankun 方案

https://km.woa.com/group/23453/articles/show/493974
qiankun 如今只是一种代称了，或者说市面上很多方案都是类似的原理，

- shadow DOM 作沙盒
- proxy 代理 window
- 独立于应用的全局状态和事件通信

## webpack5 方案

https://km.woa.com/group/34294/articles/show/502378
