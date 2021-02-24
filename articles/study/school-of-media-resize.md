# 响应式或自适应布局的流派

![image.png](https://s1.ax1x.com/2020/10/23/BAjQjf.png)<br />_（此图有可能名称反了，但不重要，我个人更偏向于 bootstrap 被叫作响应式的）_

本文旨在罗列实现响应式或自适应布局的几种方案。

---

<a name="h4eMe"></a>

## 前言

四种设备：移动端、PC 端、超大屏、高清屏<br />两种环境：没 device-width、有 device-width<br />其他注意事项：毛细线、多端兼容、设计稿还原精度

在 `@media`  出现之前，大家是怎么过的呢？<br />PC 页面大多是给容器定宽的，手机上屏宽等于定宽，想看清内容就得靠缩放拖拽。<br />弊端在哪呢，每进一页就要放大一次，PC 端与移动端设计必然多套。

<a name="xQhXA"></a>

## 栅栏布局 方案

随后 `@media`  和  viewport device-width 的组合拳之下，偷懒的方案栅栏布局横空出世。

如果将元素或内容看作是一个个的区块，那么搬运一下位置岂不是挺方便的嘛，<br />将宽度分为 12 栏，左边占 3 栏，中间占 7 栏，右边占 2 栏；<br />当宽度变小时，左边占 12 栏跑到上面去，中间占 9 栏，右边占 3 栏，相应变化。

且  device-width 自动会让文字跟随屏幕放大，<br />原本 PC 端 3 栏的内容，到移动端看到 12 栏的内容，一眼所能看到的信息量是相近的。

很明显，栅栏布局能非常方便且粗浅的处理 PC 端与移动端的样式调整，<br />字体大小会变大，适合小屏设备阅读，多端简单地适配操作非常简单。<br />但依旧有着问题，比如专注于小屏设备的话误差很大，专注于超大屏的话还缺些火候。

<a name="qaTMh"></a>

## em 方案

本方案算是优化的一类，可以稍稍弥补上述所说的大小屏问题。

虽然有着  device-width 文字得到了放大，<br />但 320px 小屏中的 12px 与 414px 小屏中的 12px 视觉上还是有较大差异的。<br />那么，用 em 去跟随这些细小的适配粒度，再放大一次呢。

```css
html { font-size: 10px; }
@media screen and (min-width: 321px) and (max-width: 375px) {
  html { font-size: 11px; }
}
@media screen and (min-width: 376px) and (max-width: 414px) {
  html { font-size: 12px; }
}
@media screen and (min-width: 415px) and (max-width: 639px) {
  html { font-size: 15px; }
}
@media screen and (min-width: 640px) and (max-width: 719px) {
  html { font-size: 20px; }
}
@media screen and (min-width: 720px) and (max-width: 749px) {
  html { font-size: 22.5px; }
}
@media screen and (min-width: 750px) and (max-width: 799px) {
  html { font-size: 23.5px; }
}
```

说实话，设定字体大小、边距间隙、定宽定高、动画位移，需要用到 px 的场景并不太多，<br />前两者甚至可以统一公共类来完成，所以随着 em 缩放一些，能够比较粗浅轻易的适配更小的粒度。

还带来了一个蛮有意思的效果，在只有 px 的时候，宽度变化想要高度也变化是困难的，<br />而宽度随 em 变化时，如果高度也写成 em 那就很妙了。

但这也有点小问题，就是 em 中的 em 其实是翻倍，这计算量可就哦豁了。

<a name="1xdxZ"></a>

## rem 方案

如果 em 会面临嵌套后多倍计算的话，直接用 rem 不就好了吗。

再进一步，em 方案中根节点的字体大小能否直接不用 `@media`  而用 js 来实现，<br />这也便造就了 lib-flexible 等插件的出现。

而 px2rem 或 postcss-pxtorem 等工具也让开发不用侧重于写 rem 而是写 px 了。

<a name="eRwgx"></a>

### 主要代码

```javascript
function flexable(remRatio = 75) {
  function setRem() {
    var winW = docEl.getBoundingClientRect().width;
    $style.innerText = 'html{font-size:' + (docEl.style.fontSize = winW / remRatio + 'px') + ' !important;}';
  }
  var win = window,
    doc = document,
    docEl = doc.documentElement,
    $style = doc.createElement('style');
  doc.head.appendChild($style);
  setRem();
  win.addEventListener('resize', setRem, !1);
}

flexable(7.5); // 7.5rem = 100vw
```

<a name="AUIgS"></a>

### 改动 viewport

[https://foreverz133.github.io/demos/works/lib-flexible/js/lib-flexible.js](https://foreverz133.github.io/demos/works/lib-flexible/js/lib-flexible.js)

众所周知，小数和单数的尺寸数值其实并不精准，甚至 1px 就能造成 float 布局的错位，<br />而 rem 方案中转化为实际 px 其实相当多情况都是小数，所以会更注重毛细线效果的结果。

而  initial-scale 一定程度上可以稍稍减小误差的概率。<br />也能让高清屏得到  initial-scale 放大，显得更清晰细致（不包括图片）。

```javascript
var isAndroid = win.navigator.appVersion.match(/android/gi);
var isIPhone = win.navigator.appVersion.match(/iphone/gi);
var devicePixelRatio = win.devicePixelRatio;
if (isIPhone) {
  // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
  if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {
    dpr = 3;
  } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)) {
    dpr = 2;
  } else {
    dpr = 1;
  }
} else {
  // 其他设备下，仍旧使用1倍的方案
  dpr = 1;
}
scale = 1 / dpr;
metaEl.setAttribute(
  'content',
  'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no'
);
```

<a name="dkJgO"></a>

### body 的 font-size

在很多插件中，body 的 font-size 会被设置为 12 \* dpr 的大小。

而如果将其设为 12rem 的话将出现一种很优秀的效果。<br />就是 Ctrl + 加号 去缩放屏幕时，并不会改变视图。

![dgdsfgd.gif](https://s1.ax1x.com/2020/10/23/BAjJEQ.gif)

<a name="xkyiS"></a>

## vw 方案

然而当开始使用 rem 后你会发现一个问题， `@media`  不再那么有效了，<br />那么我想用  `@media(min-height: 414px)`  或 `@media(min-height: 414rem)`  去调整些什么都是不可行的。

vw 方案则比较好地能**部分解决**这个问题，毕竟它并没有改变什么比例基准。<br />而 initial-scale 也可以靠  `@media(min-device-pixel-ratio)`  来搞搞。<br />可以说 vw 方案其实要比 rem 方案更应该被推崇，至少大漠也这么认为。

但是吧，rem 方案 和 vw 方案，在非全屏宽布局中其实都不太 OK。

```css
.container {
  width: 1280px;
  margin-left: auto;
  margin-right: auto;
}
```

<a name="XNuVC"></a>

## pt 方案

以上方案，都用到了 viewport 的  device-width 来进行屏幕宽度的响应，而 pt 方案则不然。

```html
<meta name="viewport" content="user-scalable=no" />
```

```scss
@function px($px, $designWidth: 750) {
  @return (735 / $designWidth) * $px * 1pt;
}
```

它有着 rem 方案类似的效果，但不需要修改根节点字体大小。<br />比如：此处的 px(375, 750) 相当于 50vw。

不过，此方案在屏宽大于 980px 后就没用了，因此只适用于手机端。<br />以前有试用过三个月，没有出现过纰漏，感觉也是个非常有效的方案。

具体原理不详，原文来自于 [移动端 HTML 响应式布局之神奇的 pt](https://juejin.im/post/5a9a3bc6f265da237a4c7722)。

其实这和流行 viewport 前的原始形态很相似，也是字体大小会随屏幕缩放，<br />但有个比例尺后，就和设计稿尺寸对应上了，妙哉妙哉。

<a name="cIqwv"></a>

## 固定视图 方案

```html
<meta name="viewport" content="width=750,user-scalable=no" />
```

此方案就很骚了，直接改 viewport 其他都按 px 来。

不搞什么自动间隙，什么左右靠齐，就是固定宽，开发起来贼舒服。<br />以前试用了半年多，用于移动端也完全没问题，PC 端有极少设备不能用。

<a name="RK019"></a>

## 百分比定位

其实这是最常见的响应式方案了，只是并不处理文字而已。<br />所以仅有图片等元素的很多活动 H5 就直接用百分比绝对定位来实现自适应了。

<a name="GWyti"></a>

## scale 缩放居中

以上方案都是根据屏宽来产生响应的，那么有没有办法以容器宽度来响应的呢。<br />很遗憾，要么 iframe 要么 transform 的 scale 来实现了。

比如一个游戏界面，只要固定的宽高不超出屏幕，居中就完事了。

```javascript
function initWindowSize(designWidth = 750, designHeight = 1334) {
  var winW = window.innerWidth;
  var winH = window.innerHeight;
  var $body = $('.container');
  var ww, hh;
  if (winW / winH < designWidth / designHeight) {
    ww = winW;
    hh = (designWidth / designHeight) * ww;
  } else {
    hh = winH;
    ww = (designWidth / designHeight) * hh;
  }

  var scale = ww / designWidth;
  $body.css('transform', 'scale(' + scale + ')');
}
```

而且此方案特别适合横屏应用，是改造量最少的方案。
