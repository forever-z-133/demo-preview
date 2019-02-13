# 六种蜂巢形样式

```css
body { background: #efefef; }
a { color: inherit; text-decoration: none; display: block; }
.comb { position: relative; width: 300px; }
.comb.bgImage { background-image: url(https://foreverz133.github.io/demos/img/lake.png); }
```

还需考虑的事情：
* 能否加背景图
* 能否加边框
* 是否为正六边形
* 有效点击范围是否为六边形
* 是否有内容

## 方法 1：CSS clip-path

```css
.comb {
  -webkit-clip-path: polygon(25% 0, 75% 0, 100% 50%, 75% 100%, 25% 100%, 0 50%);
  clip-path: polygon(25% 0, 75% 0, 100% 50%, 75% 100%, 25% 100%, 0 50%);
}
.comb:before { /* 用来撑起高度 */
  content: "";
  display: block;
  padding-top: 86.602%;
}
.comb:after { /* 用来画背景，边框实为原背景色 */
  content: "";
  position: absolute;
  top: 0; bottom: 0;
  left: 0; right: 0;
  -webkit-clip-path: inherit;
  clip-path: inherit;
  background-color: #fff;
}
```
```css
.comb.bgImage:after {
  @extend .image;
  background-size: cover;
}
.comb.border {
  background: #aaa;
}
.comb.border:after {
  $borderWidth: 10px;
  top: $borderWidth;
  left: $borderWidth;
  right: $borderWidth;
  bottom: $borderWidth;
}
```

<iframe height="430" style="width: 100%;" scrolling="no" title="蜂巢样式 css clip-path" src="//codepen.io/foreverZ133/embed/KaBBwX/?height=430&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/KaBBwX/'>蜂巢样式 css clip-path</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法2：SVG clip-path
```html
<svg width="100%" viewBox="0 0 100 86.602" style="position:absolute;left:-999em;">
  <defs> <!-- 全局公用，且需脱离文档流并显示 -->
    <clipPath id="combClip">
      <path id="combLine" d="M25,0  L75,0 L100,43.301 L75,86.602 L25,86.602 L0,43.301 Z" />
    </clipPath>
  </defs>
 </svg>

<div class="comb">
  <svg width="100%" viewBox="0 0 100 86.602">
    <a xlink:href="#">
      <image width="100" height="100" clip-path="url(#combClip)" 
        xlink:href="https://foreverz133.github.io/demos/img/lake.png"
        preserveAspectRatio="none"
      />
      <use xlink:href="#combLine" fill="transparent" stroke="black" 
        stroke-width="5" stroke-linejoin="round" stroke-linecap="round"
      />
    </a>
  </svg>
</div>
```

<iframe height="450" style="width: 100%;" scrolling="no" title="蜂巢样式 svg clip-path" src="//codepen.io/foreverZ133/embed/NdBLae/?height=450&theme-id=dark&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/NdBLae/'>蜂巢样式 svg clip-path</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法 3：三个元素旋转
```html
<div class="comb border bgImage">
  <a href="javascript:;" class="corner1"></a>
  <a href="javascript:;" class="corner2"></a>
  <a href="javascript:;" class="center">我是内容</a>
</div>
```

```css
.comb {
  background-size: 0 0;
  background-position: center;
}
.comb:before {
  content: "";
  display: block;
  padding-top: 86.602%;
}
.comb > * {
  position: absolute;
  top: 0; bottom: 0;
  left: 25%; right: 25%;
  background: inherit;
  overflow: hidden;
}
.comb > .center {
  background-size: auto 100%;
}
.comb > .corner1,
.comb > .corner2:after {
  transform: rotate(60deg);
}
.comb > .corner2,
.comb > .corner1:after {
  transform: rotate(-60deg);
}
.comb > .corner1:after,
.comb > .corner2:after {
  content: "";
  position: absolute;
  top: 0; bottom: 0;
  left: -50%; right: -50%;
  background: inherit;
  background-size: auto 100%;
}
```
```scss
.comb.bgImage {
  @extend .image;
}
.comb.border > * {
  border: 0 solid red;
  border-width: 2px 0;
}
```

<iframe height="490" style="width: 100%;" scrolling="no" title="蜂巢样式1" src="//codepen.io/foreverZ133/embed/mRjLdM/?height=490&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/mRjLdM/'>蜂巢样式1</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法 4：背景元素
```html
<a href="javascript:;" class="comb">
  <div class="comb-bg"></div>
</a>
```
```css

```
<iframe height="450" style="width: 100%;" scrolling="no" title="蜂巢样式2" src="//codepen.io/foreverZ133/embed/jypKjw/?height=450&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jypKjw/'>蜂巢样式2</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法 5：特殊背景图

原图长这个样子：
![蜂巢状特殊背景原图](http://mall.kdcer.com/Content/img/comb.png)

<iframe height="265" style="width: 100%;" scrolling="no" title="蜂巢样式 background" src="//codepen.io/foreverZ133/embed/yvzqvX/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/yvzqvX/'>蜂巢样式 background</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法 6：左右加个小尖角