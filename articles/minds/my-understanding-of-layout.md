# 我理解的前端-布局篇

简单回顾一下上篇内容 [我理解的前端-动画篇](https://www.yuque.com/docs/share/60e5cf65-c147-4ac7-b9fc-1a1965be5bd8) ，动画由起始数值/结束数值/运行时长/动画节奏组成。后面的暂停重复反转等动画链，或者位移旋转加速等数值抽象概念，再或者过程思维和结果思维等开发习惯，再或者播放式动画和即时反馈式动画等表现形态，都算是延伸的产物了。

那么布局的核心是什么，又延伸了那些产物呢？请看下文。

<a name="H6dHH"></a>

## 前言

在栅栏布局时代我就在想，float/inline-block/table 这些布局都有着这样或那样的问题，<br />
虽然这些属性在设计之初也本来就不是拿来做布局的，<br />
那么为什么不发明一款更实用的横排布局呢，之后 flex 就出现了。<br />
_写于 06 年的博客：_[https://www.cnblogs.com/foreverZ/p/no-float-is-better.html](https://www.cnblogs.com/foreverZ/p/no-float-is-better.html)<br />
那么是否还有其他你觉得也应该存在的样式属性呢？

有位 [大佬说过](https://www.zhihu.com/question/39659757) ，不管 CSS 的形式如何改变，它都只是个 DSL 语言而已，那布局的语义和语法该是什么？<br />
比如我们会写很多 scss mixin，或者像 flutter 的样式组件那样，那是否就代表着其实样式可以缩写的，或者说指定我们自己的 DSL。（或者低码平台经常讲的 Schema）<br />
_广告位：_[https://www.npmjs.com/package/common-scss-mixin](https://www.npmjs.com/package/common-scss-mixin)

如今，虽然我们用了很多 UI 库或组件库，不用再写那么多 CSS，<br />
但却依旧需要我们夯实 css 的基础才能处理各式奇怪的布局 bug。<br />
比如键盘，比如文字编辑，比如横竖屏切换，比如吸顶，ios safe-area 等。<br />
具体再比如中文数字/符号不对齐、行高在精细布局上的问题、吸顶总要搞骚操作等。<br />
那么是否有容错性更高或更容易理解的样式开发习惯呢？让写个样式不用过多地去反复调试。<br />

另一方面，我个人非常看好 Surface Duo 这类新概念手机（折叠屏、伸缩屏）的出现，也一直感叹应用在 pad 端适配的拉跨。<br />
略加思索也会意识到，越来越多的设备和交互形式（比如智能手表、智能眼镜）会产生，如今的这套布局还适用吗。<br />
比如双面屏或分屏如果还是视窗为基准的 rem 会造成字号不一致等。<br />
更多工具型应用也在尝试颠覆现有的 block 流式布局，比如相机切换景深的圆盘，更多样的左滑长按等。<br />

再者，可视化开发依旧是很火热的一个市场，低码平台很有可能是未来趋势。<br />
我们经历了很多流派，从 table 布局，流式布局，定位布局，响应式布局，弹性布局一步步尝试和演进，<br />
可视化开发工具也在变迁，比如远古的 DW，或依旧通用的 wordpress，或微秀、凡科建站，或设计转代码。<br />
都非常出色，还有大厂们在推荐的物料库，但也有着诸多麻烦和问题。<br />
比如物料难以寻找、实现混乱、自定义难度极高等等，这让可视化开发在现阶段并不那么美妙。

带着以上的牢骚与思考，我们先梳理一下前端布局的本质。

## 结构与样式的关系

通常，我个人是更希望 html 只管语义和结构的，html 不应该为样式让步，但现实并不如此。

#### 结构与样式的语义杂糅

一方面，是很难处理好组件与样式的语义。约定了一个，另一个可能就要妥协。<br />
我们会比较在意单个元素的语义，而并不太在意层级结构的语义。<br />

比如能确定这是个 list，但内部 list-item 和 list-tools 是否该同级呢。<br />
再比如实现不了布局就再套一层 div 可实现，那新的一层 div 该给什么语义呢。

```html
.goods-list-item
  .goods
    .goods-image
    .goods-content
      .goods-name
      .goods-price
      .goods-sold-number
  .goods-list-item-tools
    .checkbox
    .delete-goods-button
```

更现实的，看了一眼设计稿，发现它是单行左右布局/有内边距/左右等高/左右有间隙，<br />
然后就写下了 `div>(div+div)` 的结构，语义什么的以后再加吧。<br />
不得不承认，这是超级快捷完成业务的好办法，甚至于将样式直接写进 dom 还会更快速一些。

```html
<div class="flex-row flex-row-stretch item-gap-right-20 padding-20">
  <div class="flex-shrink width-120"></div>
  <div class="flex-grow"></div>
</div>
```

看上面的 html 代码，我不禁觉得，这不挺像 Flutter 的吗。<br />
比如写个 FlexRow 直接实现单行布局的效果，结构和样式完美融合。<br />
*这可能也是 Flutter 选用 Dart 为开发语言的原因之一吧 [https://www.infoq.cn/article/why-flutter-uses-dart](https://www.infoq.cn/article/why-flutter-uses-dart)*

更多前端应该要熟悉 Tailwind CSS 多一些，但很显然，结构的语义在其中很难体现了。<br />
当然这种开发习惯的提升非常直观，不用先写 html 再写 css，也不存在写样式过程中需要多套一层 div 又去改 html。

#### 样式嵌套与结构嵌套的冲突

另一方面，为了更稳定的结构，组件常常需要三层包裹，这样可以满足绝大多数需求变动。<br />
比如给 Input 组件加前缀的 icon 或加删除按钮，给单行容器设置宽度比和间隙，给列表加栏数和间隙或边框 等。<br />
张鑫旭老师所提倡的 **“宽度分离”** 和三层包裹是相似的。

```html
.zyh-input
  .input-box
    input

.zyh-list
  .list-box
    .list-box-item
      .goods-item

.zyh-flex-row[row="100px|1fr"]
  .flex-row-item
    .some-other-box
  .flex-row-item
    .some-other-box
```

这也意味着，当开发者不愿去思考和实践三层包裹这种规范时，结构变动必然会带来原样式的修改。<br />
举个例子，父级为 flex + space-between 的容器，现在中间加了一层 div，那子级原来的效果就丢失了。

而三层结构则能适当避免上述窘况，外层做尺寸和装饰，中层做布局，内层做嵌套子级，<br />
通常只会进行子级的增删而已，不会出现嵌套关系的改变。

当然，为了样式的嵌套而增加很多结构的嵌套，来尽量寻求修改样式代码的稳定性，并不是多么美妙的方案，但确实也是个办法，看你的选择吧。

## 样式与行为的关系

CSS 这门语言在旧时代并不具备太多的交互响应功能，随着时代发展倒是有了些变化。<br />
比如像 W3C 中也有了 `:focus-within` `:target` 或 `<output>` 等这些存在；<br />
比如像 Tailwind CSS 有了些带交互的样式类，比如 `dark:bg-black` `group-hover:text-blue-500` 等；<br />
比如像下方代码这样，直接将信息输入给 CSS var 变量去进行处理的，这种做简单交互真的很香。

```javascript
const docStyle = document.documentElement.style;

document.addEventListener('mousemove', e => {
  docStyle.setProperty('--mouse-x', e.clientX);
  docStyle.setProperty('--mouse-y', e.clientY);
});
```

```css
:root {
  --mouse-x: 0;
  --mouse-y: 0;
}
.point {
  position: absolute;
  width: 10px;
  height: 10px;
  background: red;
  left: calc(var(--mouse-x) * 1px);
  top: calc(var(--mouse-y) * 1px);
}
```

而更复杂或更需要计算的场景，则只能走 css-in-js 这条路了。<br />
比如 先知道文本是否换行来觉得显不显示展开按钮，或先知道图片尺寸来进行布局计算 之类的。

那么，样式与行为是否有和谐统一的可能性呢，我们来分析一下。

首先我们要避免调试的麻烦，比如是 css 失效还是 js 赋值错误要能 debug 出来，显然 css 变量被排除在外。<br />
交互样式通常有两种，切换状态 或 变动数值，
另一方面，无论 js 还是 css 都是将信息转为了状态值，但 css 有希望支撑得起状态值转为布局的处理吗，比如 if。<br />再其次，有的输出是靠 class 改动来实现的，有的输出靠 css 变量的，这不美妙呀。

_也正是这样想拆又拆不开/当断不能断的现状，也是我个人不太喜欢 css-in-js 的原因_

## 样式的原子化

就像 css-in-js 有 20 多种方案一样，样式原子化的粒度或开发的习惯上，依旧没有一条比较明晰的道路。<br />
在知乎问题 [如何评价 CSS 框架 TailwindCSS？](https://www.zhihu.com/question/337939566/answer/1679266812) 我也是这样的态度，现在流行的方案并没有形成最佳实践。

但在个人开发中，scss 的 mixin 和 css 的 var 又相当好用，
