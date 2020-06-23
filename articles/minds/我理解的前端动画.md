# 我理解的前端-动画篇

撇开代码去理解前端，前端实际为 **布局**、**行为/交互**、**资源和数据**、**跨端和兼容** 这四大类。<br />所以心血来潮之下想另开几个大坑，去整理和表述出自己现在所能构想到的前端蓝图。<br />本系列包含 **动画**、**布局**、**交互**、**媒介** 四个方面，分别梳理我所理解的前端。

---


<a name="s271P"></a>
## 前言

在 Behance 2020 设计趋势中有描述，2020 年度趋势为 Motion Design & Animations。<br />原文：[https://mp.weixin.qq.com/s/EvtTSCkN6IxcY1nw1oePRA](https://mp.weixin.qq.com/s/EvtTSCkN6IxcY1nw1oePRA)

伴随 5G 的来临，前端也需要更多思考交互响应与性能问题，<br />低延时下能解决大多数加载问题，就算本机性能差，也有云设备同步方案。<br />当不再有 loading 时，突然地数据显示变化是否会显得突兀呢。

在 imgCook 这类弱 AI 应用崛起时，网站开发者可以如何突围。

本文以动画为切入点，尝试探究整理其概念与意义。

<a name="Toi8J"></a>
## 前端动画的价值

首先，我想先理清交互的含义，它的概念很大，也可能存在些误解。<br />理清它有助于理解动画在交互中的作用。

交互，即人机交互，不同于机器反馈的单方面运行，<br />就好比输入输出中的输入不只有函数的变量传入，还有用户的操作行为带来的输入。

其实可见，交互设计并不是艺术设计，美感那是界面设计的事情（即下篇布局）。<br />交互更侧重于用户行为上的考量，通常也是分析大于设计，其实非常工程学。

而动画一定程度上是同时为美感和行为赋能，<br />你可以给按钮加上小动效，让画面更灵动，也可以为交互服务引导用户操作。<br />这是我理解的前端做动画的价值。

> 程序员通常是一个差的设计师和差的管理者。
> —— 交互设计之父：Alan cooper


之后将介绍动画类型分类、如何实现动画、以及动画设计原则，与君共勉。

<a name="fKA6K"></a>
## 单动画

![image.png](https://s2.ax1x.com/2020/02/08/1Rd5UP.png)

最基本的动画构成，只需要 **起始态**，**结束态**，**时长** 和 **节奏**

```javascript
function anim(start, target, duration, options = {}) {
  let time = +new Date();
  const every = options.every;
  const cubicBezier = options.cubicBezier;
  const finish = options.finish;

  (function loop() {
    const t = +new Date() - time;
    let per = cubicBezier ? cubicBezier(t, duration) : (t / duration);
    const res = start + per * (target - start);
    every && every(res, per);
    if (per >= 1) return finish && finish();
    requestAnimationFrame(loop);
  })();
}
```

示例：[https://codepen.io/foreverZ133/pen/QWwWRVY](https://codepen.io/foreverZ133/pen/QWwWRVY)

_思考题：假如结束态如果是跳跃式的怎么办，比如从屏幕右侧滑出后从屏幕左侧闪现出现，是否也适用于本方程。_

<a name="CbXjI"></a>
#### 目标思维和过程思维

从以上思考题中，这也能看出动画的两种思维方式，<br />a. 起止数值与运动元素有明确关系，数值即为元素实际表现；<br />b. 起止数值只与运动量有关，或叫运动进程的百分比，与元素状态无关。

示例：[https://codepen.io/foreverZ133/pen/eYmmjNm](https://codepen.io/foreverZ133/pen/eYmmjNm)

```javascript
// 基于目标值的运算
var every = (now) => ($el.style.left = 50 + (now - 50) % ((1050 - 50) / 5) + 'px');
anim(50, 1050, 3e3, { every });

// 基于运动过程的运算
var every = (now, per) => ($el.style.left = 50 + (per % 0.2) * 1000 + 'px');
anim(0, 1000, 3e3, { every });
```

像 jquery.transit 和 CSS 动画都是目标思维的，更为便捷<br />
[http://www.jq22.com/demo/jquery.transit-master20161220/](http://www.jq22.com/demo/jquery.transit-master20161220/)

而像 TweenJS 和上述 anim 方式则能兼容过程思维的，更为灵活。
TweenJS：[http://www.createjs.cc/tweenjs/docs/modules/TweenJS.html](http://www.createjs.cc/tweenjs/docs/modules/TweenJS.html)<br />


<a name="iW0RL"></a>
#### 单独讲讲 cubic-bezier

cubicBezier(t, start, changeBy, duration, a, b, c, d)<br />其中，t 为 nowDate - startDate，changeBy 为 target - start，abcd 为两个曲线句柄。<br />_曲线句柄即为类似 PS 钢笔工具中的两个曲率标记点，可见下方 CSS 式节奏 的效果。_<br />_而 js 式的就是其平面直角坐标系里的曲线函数。<br />_

CSS 式节奏：[http://yisibl.github.io/cubic-bezier/](http://yisibl.github.io/cubic-bezier/)<br />JS 式节奏：[https://cdn.bootcss.com/jquery-easing/1.4.1/jquery.easing.js](https://cdn.bootcss.com/jquery-easing/1.4.1/jquery.easing.js)<br />

有了节奏后，各类抛物线动画、非线性动画之类的概念就都很轻松实现了。

<a name="fffwe"></a>

## 动画链

![image.png](https://s2.ax1x.com/2020/02/08/1R0rlD.png)

动画链包含：**间隔/延时**、**暂停**、**反转**、**重复** 和 **同时动画/动画组**。<br />

动画链是十分常见且有效的组合方式，但多数语言和工具都未能将其全部实现。<br>
另外，动画组是多个动画元素（比如 x, y）同时运行的代称，如路径动画便是动画组的体现。

示例：[https://codepen.io/foreverZ133/pen/LYEYKEj](https://codepen.io/foreverZ133/pen/LYEYKEj)

<a name="N9EGT"></a>
## 图形动画

_此处的图形暂不包含线条哈，我个人感觉会比较难讲。_<br />
图形动画的本质其实就是多个单点动画，只是点的存储方式可以有些不同。<br />
这也是绘制引擎的性能差异的影响因素之一，也能在自己写动画时产生一些跳出常规的思考。

![image.png](https://s2.ax1x.com/2020/02/08/1RBBEn.png)

咱们继续思考，其实图形动画并不简单，<br />
比如从三角形变为四边形，从三个点出发其实会有三种不同的动画。<br />

以下便延伸两个我觉得较为重要的思考点：**对称中心问题** 和 **结果反推过程问题**。

<a name="sPBt8"></a>
#### 对称中心问题

[https://forever-z-133.github.io/demos/single/edit-size-box2.html](https://forever-z-133.github.io/demos/single/edit-size-box2.html)<br />
如上链案例，旋转到颠倒后再缩放，原增加宽度的逻辑就该改为减少了。<br />

对称中心在图形动画中会产生相当大程度的影响，像旋转中心和宽度计算中心等。

不过也能发觉一些经验性的规律，比如，图形每多一维就会多一个对称中心。<br />
像三维世界中，除了物体本身的对称中心，还会多一个世界中心，也就是摄像机视点。

<a name="ujJ0b"></a>
#### 结果反推过程

我们自己纯 js 手写动画时，多半是 `x += 1` 这样不断加到结果位置的，<br />
但其实我们也可以直接用结果反推过程来做，设定结果位置后，过程自动去补全。<br />

这时就不禁让人产生一些跳出现有习惯的思考。<br />
大家用 CSS 动画超方便的，然而当动画组成分增多后这个过程其实就复杂了。<br />
假设设计师给予了一个又旋转又斜切又位移还不知道对称中心的结果图，那我该如何重现呢？<br />

这时就得提到一种叫做 FLIP 动画的概念，它并不在乎你是先旋转再位移还是如何，只需告诉它结果状态即可。
比如某元素从 `width:100px` 放大到 `width:500px`，FLIP.js 会自动补上 `scaleX(5)` 的动画。

<a name="wnWiC"></a>
## 动画类型

![image.png](https://s2.ax1x.com/2020/02/08/1RydzV.png)<br />

终于回到了我们熟悉的地方来，我们经常操作的都是上图的动画类型，如位移旋转这样的概念。<br />
但上文所述的诸多规律和思考也都适用于处理这堆动画类型。你品，你细品。

比如 matrix 是位移旋转斜切概念的合并等，<br />
诸多数值都是变化量的某种抽象，世上本无旋转，但我发明了 rotate 去表现它。

摩擦(friction)，节制(dampening)，重力(gravity)，惯性(inertia) 也是如此，是表现抽象为数值的过程。<br />
晃动、闪烁、翻转、弹簧等等诸多动画，即是这些抽象数值组合的结果。

示例：[https://forever-z-133.github.io/demos/single/coolHover.html](https://forever-z-133.github.io/demos/single/coolHover.html)

<a name="bW5na"></a>
## 动画的触发

在淘宝大佬 winter 的 bindingX 中，有着 **手势、动画、滚动、陀螺仪** 这几种类型。<br />[https://alibaba.github.io/bindingx/guide/cn_guide_base](https://alibaba.github.io/bindingx/guide/cn_guide_base)

我总结一下，关于动画的触发应该分为两种，**即时反馈式动画** 和 **播放式动画**。<br />
_其实还有 **条件式动画**，为反馈过程中满足某条件时产生的，也算在即时性动画中吧。_

未来应该还会有更多的及时性动画出现，比如视频播放/VR/各种手势操作等 ，欢迎进行畅想。

<a name="QEjj7"></a>
## 多动画的联动

当多个抽象数值同时动画时，它们之间就会需要一些别样的规律和思考了。<br />
比如，拖动 A 则 B 也动，或者拖动 A 则 B 稍后再动 之类的。

笔者水平有限，可以大致总结为 **相似**、**相反**、**延时**、**减弱**、**其他** 这几类联动形式。

其他作者对此也有探究和思考，可见以下文章：<br />
[https://mp.weixin.qq.com/s/54acdnB6QQltm9vMcKpYpw](https://mp.weixin.qq.com/s/54acdnB6QQltm9vMcKpYpw)

<a name="IyApK"></a>
## 动画设计原则

什么样的动画才算是好动画呢，虽然 UX 设计并不属于前端范畴，但我个人一直很想探讨这方面。

![dgdsfgd2.gif](https://s2.ax1x.com/2020/02/08/1RghQ0.gif)<br />

很多时候都是别人想好了效果丢给前端去实现，但大多公司都不见得有 UX 设计这个岗位或概念，<br />
或者当在做个人项目的时候，干巴巴的跳转和 UI 库默认动画，都不够酷炫不够装逼。动画在前端领域真的很落后。

但当某天我想做类似 [https://www.figma.com/](https://www.figma.com/) 这样的 IDE 的时候才会发现自己的短板吧。

国内可借鉴的案例并不多，那么，动画思路从哪来，怎么做好动画，如何让动画更美妙，这便是设计原则。<br />
或者说，设计原则是已成文的规律性总结，能帮我们更好地去理解自己正在做的事情。

<a name="tzgZs"></a>
#### 流派壹

**迪斯尼工作室的电脑角色动画 12 原则**<br />_其实没啥关系，看着乐下吧_

1. squash & stretch（挤压和拉伸）
1. anticipation（预见）
1. staging（分镜、风格、关键位置）
1. straight ahead & pose to pose（连贯和关键动作）
1. follow through & overlapping（跟随和叠加）
1. slow in & slow out（渐快和渐慢）
1. arcs（弧形运动）
1. secondary action（附属动作）
1. timing（节奏）
1. exaggeration（夸张）
1. solid drawings（手绘）
1. appeal（吸引力）

<a name="74XX3"></a>
#### 流派贰

**Material Design**，谷歌对操作系统制定的设计语言<br />相关视频：[https://www.youtube.com/watch?v=rrT6v5sOwJg&list=PL8PWUWLnnIXPD3UjX931fFhn3_U5_2uZG](https://www.youtube.com/watch?v=rrT6v5sOwJg&list=PL8PWUWLnnIXPD3UjX931fFhn3_U5_2uZG)<br />主要文档：[https://www.material.io/design/motion/understanding-motion.html](https://www.material.io/design/motion/understanding-motion.html)

Informative（内容丰富） & Focused（有主体的） & Expressive（富有表现力的）<br />有反馈 & 有提示 & 表达有连续性的 & 有个性符号 & 运行是一致的

<a name="j5ImX"></a>
#### 流派叁

**iOS design**，美学完整性，一致性，直接操作，隐喻，用户控制<br />相关视频：[https://b23.tv/av66343171](https://b23.tv/av66343171)<br />主要文档：[https://developer.apple.com/design/human-interface-guidelines/ios/overview/themes/](https://developer.apple.com/design/human-interface-guidelines/ios/overview/themes/)

非线性动画，可交互的 & 可中断的 & 可重新定向的 & 可随时响应的

<a name="PJRZL"></a>
## 反思和实践

FLIP.js 虽然让前端有了一定的 FLIP 动画概念，但 CSS 语言本身特点使其应用场景是有限的，<br />
但在其他场景下依旧是魅力无限，我无需告诉你旋转了多少，你会帮我算出它是慢慢旋转成这样子的。

在以上总结的动画分类中，播放式动画其实就是如 animate.css 那样的动画模板，这类实践非常简单好用，<br />
就算使用 bindingX 那样的 js 动画，播放式动画也只是编写固定的动画模板而已。某天你再写个 canvas 或 flutter 的动画库也能火。<br />
这也是大多数网页制作工具偷懒的方式，以为这样就是给前端加上了动画。

即使反馈式动画和多动画联动要考虑的东西还很多，且很少有人去总结，也是可以继续深究的角度。

再者是抽象数值，其实还能有相当多的常见变量，比如滚动回弹效果中的甩力和反弹力等等。另外可以从 AE 软件中能学到相当多概念。

当在前端能熟练制作动画时，玩玩 AE 视频特效，玩玩抖音还不是小意思了，转职做 UX 设计或动作设计也阔以。

还有在 imgcook 基础上，如果我有两份设计稿，结合 FLIP 和 BindingX 前端动画也能靠 AI 做了，刺不刺激。

vue 的 transition-group 是真的很棒，但 react 中还没有。
https://cn.vuejs.org/v2/guide/transitions.html#%E5%88%97%E8%A1%A8%E7%9A%84%E8%BF%9B%E5%85%A5-%E7%A6%BB%E5%BC%80%E8%BF%87%E6%B8%A1

**在前端动画这个领域还有太多太多可以做的事情了。**