# 蜂巢形样式的实现

```css
body { background: #efefef; }
a { color: inherit; text-decoration: none; display: block; }
.comb { position: relative; width: 300px; }
.comb.bgImage { background-image: url(https://foreverz133.github.io/demos/img/lake.png); }
```

还需考虑的事情：
* 有效点击范围
* 可加背景图
* 可加边框
* 是否为正六边形

### 方法 1：三个元素旋转
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
.comb.border > * {
  border: 0 solid red;
  border-width: 2px 0;
}
```

<iframe height="460" style="width: 100%;" scrolling="no" title="蜂巢样式1" src="//codepen.io/foreverZ133/embed/mRjLdM/?height=460&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/mRjLdM/'>蜂巢样式1</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法2：clip-path