# 低代码平台分析-产品篇

## 低代码必然有其边界

笔者陆陆续续做过 海报编辑器、H5 编辑器、Scratch 编程游戏、网站编辑器、SaaS 平台编辑器，<br />
但无论是哪种低代码，都会遇到一个非常磨人的问题，就是 **定制化需求防不胜防**。

比如 轮播里加视频、修改兄弟组件状态、默认不登陆点击后才登陆 等等，<br />
有些是伪需求，有些是产品过度设计，有些是系统或语言不支持，有些是偏离架构设计，所以才说是磨人。

为了团队和作品着想，低代码一定要设定边界，想好自己的产品特色。用户提什么就做什么最是大忌。<br />
假如你是做 GIS 地图服务的编辑器，你总不会想要把地图 SDK 所有 api 都开放给用户吧。<br />
假如你是做 SaaS 平台编辑器，自定义数据库、自定义组件，你总不会想要三个月就达到商用水平吧。<br />

再举些正面的例子：<br />
比如做管理后台编辑器的大佬分享时，总会说一句 “得益于 Ant Design 的广泛普及，交互与视觉越来越标准化”，那么就不要强求这个平台会提供丰富的样式定制化。<br />
比如做特色 H5 的编辑器 iH5，为了做滚动或长屏等非传统 H5，选择的是新开编辑器而非让旧编辑器去兼容，就是比较明智的选择，宁可零散而专业，也不臃肿而复杂。<br />

## 低代码的产品特性

低代码通常的作用是为了 **快速处理重复工作**，比如 快速写页面样式、

## 低代码的元素组成

以前，网站三剑客为 DreamWeaver，Fireworks，Flash；前端三剑客为 HTML，CSS，Javascript；建站三剑客为 PhpStudy，DW，PhotoShop。<br>
早期做 H5 或海报编辑器的则会按 **元件(设计时)** **属性** **组件(渲染时)** 来划分。<br />
更甚者，现在的低代码平台有的开始转向做 SaaS 平台，会按 **数据**、**设计**、**流程** 来诠释什么叫网站，不仅仅涵盖前端范畴。<br />
其实这种划分挺有意思的，一些技术大佬在分享架构设计的时候也爱介绍产品设计的这一面，所以我们也来思考一下，低代码平台的产品设计应该如何取舍。

从产品角度看，前端是 **样式/布局**、**行为/交互**、**资源和数据**、**跨端和兼容** 这四大组成部分。<br />

图标属于样式还是资源，

## 低代码的技术难点