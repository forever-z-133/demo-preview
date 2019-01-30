# 蜂巢形样式的实现

各套方案并不完美，有以下几个难点：
* 有效点击范围
* 可加背景图
* 可加边框

### 三个元素旋转而成
```html
<div class="wrap">
  <div class="comb border comb1" style="background-image: url(https://foreverz133.github.io/demos/img/lake.png)">
  <a href="#" class="corner1"></a>
  <a href="#" class="corner2"></a>
  <a href="#" class="center">我是内容</a>
  </div>
</div>
```

```css
.comb {
  width: 300px;
}
.comb1 {
  position: relative;
  background-size: 0 0;
  background-position: center;
}
.comb1:before {
  content: "";
  display: block;
  padding-top: 86.602%;
}
.comb1 > * {
  position: absolute;
  top: 0; bottom: 0;
  left: 25%; right: 25%;
  background: inherit;
  background-color: #fff;
  overflow: hidden;
}
.comb1 > .corner1 {
  transform: rotate(60deg);
}
.comb1 > .corner2 {
  transform: rotate(-60deg);
}
.comb1 > .corner1:after,
.comb1 > .corner2:after {
  content: "";
  position: absolute;
  top: 0; bottom: 0;
  left: -50%; right: -50%;
  background: inherit;
  background-size: auto 100%;
}
.comb1 > .corner1:after {
  transform: rotate(-60deg);
}
.comb1 > .corner2:after {
  transform: rotate(60deg);
}
.comb1 > .center {
  background-size:  auto 100%;
}
.comb1.border > * {
  border: 2px solid #000;
  border-left: none;
  border-right: none;
}
```

<iframe height="365" style="width: 100%;" scrolling="no" title="蜂巢样式1" src="//codepen.io/foreverZ133/embed/mRjLdM/?height=365&theme-id=dark&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/mRjLdM/'>蜂巢样式1</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>