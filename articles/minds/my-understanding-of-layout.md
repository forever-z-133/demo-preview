# 我理解的前端-布局篇

简单回顾一下 [上篇内容](https://www.yuque.com/docs/share/60e5cf65-c147-4ac7-b9fc-1a1965be5bd8) ，动画由起始数值/结束数值/运行时长/动画节奏组成，后面的暂停重复反转等动画链，或者位移旋转加速等数值抽象概念，再或者过程思维和结果思维等开发习惯，再或者播放式动画和即使反馈式动画等表现形态，都算是延伸的产物了。

那么布局的核心是什么，又延伸了那些产物呢？请看下文。

<a name="H6dHH"></a>

## 前言

在栅栏布局时代我就在想，float/inline-block/table 这些布局都有着这样那样的问题，<br />虽然这些属性在设计之初也本来就不是拿来做布局的，<br />那么为什么不发明一款更实用的横排布局呢，之后就知晓了 flex 的存在。<br />_写于 06 年的博客：_[https://www.cnblogs.com/foreverZ/p/no-float-is-better.html](https://www.cnblogs.com/foreverZ/p/no-float-is-better.html)<br />那么是否还有其他你觉得更应该存在的属性呢？

有个 [大佬说过](https://www.zhihu.com/question/39659757) ，不管 css 的形式如何改变，它都只是个 DSL 语言而已，那布局的语义和语法该是什么？

如今，虽然我们用了很多 UI 库或组件库，不用写很多的 CSS，<br />但却依旧需要我们夯实 css 的基础才能处理各式奇怪的布局 bug。<br />比如键盘，比如文字编辑，比如横竖屏切换，比如吸顶，等。<br />具体再比如中文数字/符号不对齐、行高在精细布局上的问题、吸顶总要搞骚操作等。<br />那么是否有容错性更高或更容易理解的样式开发习惯吗？让写个样式不用过多地去尝试的那种。<br />_广告位：_[https://www.npmjs.com/package/common-scss-mixin](https://www.npmjs.com/package/common-scss-mixin)

另一方面，我个人非常看好 Surface Duo 这类新概念手机的出现。<br />略加思索也会意识到，越来越多的设配和交互形式会产生，如今的这套布局还适用吗。<br />比如双面屏或分屏如果还是视窗为基准的 rem 会造成字号不一致等。<br />更多工具型应用也在尝试颠覆现有砖块式的设计，比如相机切换景深的圆盘，更多样的左滑长按等。<br />_下文会具体讲到我所能注意到的新式设配与交互，它们对布局的要求大不同了。_

再者，可视化开发依旧是很火热的一个市场，很有可能是未来趋势。<br />我们经历了很多流派，从 table 布局，流式布局，堆布局，定位布局一步步尝试和演进，<br />可视化开发工具也有比如远古的 DW，或依旧通用的 wordpress，或微秀，或设计转代码的变迁，<br />都非常出色，还有大厂们在推荐的物料库，但也有着诸多麻烦和问题。<br />比如难找、实现混乱、会覆盖文件、不宜修改等等，这让可视化开发并不那么可控。

## 结构与样式的关系

通常，我个人是更希望 html 只管语义和结构的，html 不应该为样式让步，但现实并不美妙。

### 结构与样式的语义杂糅

一方面，是很难处理好组件的语义与样式的语义，约定了一个另一个就要妥协。<br />我们会比较在意单个元素的语义，而并不太在乎层级结构的语义，如果在乎，那写 css 就需要麻烦一点。<br />考虑样式时也很实际，上手就是一把梭，最多考虑下这是 flex 还是定位，不行就再嵌套 div。

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

而前端开发最最真实的一面则是，看了一眼设计稿，发现它是单行左右布局/有内边距/左右等高/左右有间隙，所以写下了 `div>(div+div)` 的结构，语义什么的以后再加吧。<br />不得不承认，这是超级快捷完成业务的好办法，甚至于将样式直接写进 dom 还会更快速一些。

```html
<div class="flex-row flex-row-stretch item-gap-right-20 padding-20">
  <div class="flex-shrink width-120"></div>
  <div class="flex-grow"></div>
</div>
```

再然后你就会发现：这不就是 Dart 吗，写个   FlexRow 传入拥有属性的对象；或者你写了个 `<flex-row>` 的组件，并加上了一些 props 而已。英吹思婷，样式与 html 开始合为一体，而 html 又以 jsx 或 dart 的形式融入 js，前端开发大一统了。

这可能也是 Flutter 选用 Dart 为开发语言的原因之一吧。<br />[https://www.infoq.cn/article/why-flutter-uses-dart](https://www.infoq.cn/article/why-flutter-uses-dart)

### 样式嵌套与结构嵌套的冲突

另一方面，为了更稳定的结构，组件常常需要三层包裹，这样可以满足绝大多数需求变动。<br />比如给 Input 组件加前缀的 icon 或加删除按钮，给单行容器设置宽度比和间隙，给列表加栏数和间隙或边框等。<br />张鑫旭老师鼓励宽度分离在结果上也和三层包裹是相似的。

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

这也意味着，当开发者不愿去思考和实践三层包裹这种规范时，结构变动必然会带来原样式的修改。<br />举个例子，父级为 flex +space 容器，现在中间加了一层，那子级原来的效果就丢失了。<br />而三层结构则能适当保证，通常只有子级的增删而已，不会出现嵌套关系的改变。

## 样式与行为的关系

CSS 这门语言很多时候并不具备信号输入功能，<br />比如先知道文本是否换行，或先知道图片尺寸，再来进行布局计算。<br />这时你就会感受到了，在现在这个时代 css-in-js *(其实是 css in template)*  也有了些可能性。

在原生 css3 中也有着 `content: attr()`  或 `:focus-within` `:target`  这些存在，<br />或者像下方代码这样，直接将信息输入给 css 去进行处理的，真的很香很香。

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

那么样式与行为是否也将合并统一呢，个人角度上我依旧不希望这样的事前发生。

一方面，现在的调试器让我们对这种场景并不能轻松的调试 bug，是 css 失效还是 js 给值错误无法立马反应。<br />另一方面，无论 js 还是 css 都是将信息转为了状态值，但 css 有希望支撑得起状态值转为布局的处理吗，比如 if。<br />再其次，有的输出是靠 class 改动来实现的，有的输出靠 css 变量的，这不美妙呀。

_也正是这样想拆又拆不开/当断不能断的现状，也是我个人不太喜欢 css-in-js 的原因_

## 进击的原子化

显然，一个元素只有一个 class 的时代逝去了，无论是在 css 里写 mixin 还是在 template 中写 Tailwind，让 css 也模块化是必然的趋势了。  

但就像 css-in-js 有 20 多种方案一样，原子化的粒度或样式开发的习惯上，依旧没有一条比较明晰的道路。  
在知乎问题 [如何评价 CSS 框架 TailwindCSS？](https://www.zhihu.com/question/337939566/answer/1679266812) 我也是这样的态度，现在流行的方案并没有形成大的变革。  
