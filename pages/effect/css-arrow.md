# 尖角与箭头的实现

可能需要考虑的事情：
* 是否要加阴影
* 是否要圆角
* 是否方便改颜色
* 尖角是 60° 还是 45°

<iframe height="265" style="width: 100%;" scrolling="no" title="CSS 三角" src="//codepen.io/foreverZ133/embed/WNewBLd/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/WNewBLd/'>CSS 三角</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 一、border
```css
.triangle:before {
  content: "";
  width: 0;
  display: block;
  border-left: .5em solid transparent;
  border-right: .5em solid transparent;
  border-bottom: 1em solid;
}
```

### 二、rotate
```css
.triangle { overflow: hidden; }
.triangle:before {
  content: "";
  width: 1em;
  height: 1em;
  display: block;
  background: currentColor;
  transform: translateY(0.5em) scale(.707, 1.414) rotate(45deg);
}
```

### 三、base64
```css
.triangle:before {
  content: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M 50,0 L 100,100 L 0,100 Z" fill="" /></svg>');
  width: 1em;
  display: block;
}
```