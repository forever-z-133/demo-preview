# 饼图的绘制

## 一、SVG

```
<svg viewBox="0 0 100 100" width="50">
  <circle r="25" cx="50" cy="50" fill="transparent" stroke="#655" stroke-width="50" />
</svg>
```
```scss
@function PI() {
  @return 3.14159265359;
}

circle {
  $progress: 45;
  $radius: 50;
  $girth: PI()*$radius;
  stroke-dasharray: #{$progress/100*$girth} $girth;
  transform: rotate(-90deg);
  transform-origin: center;
}
```

<iframe height="170" style="width: 100%;" scrolling="no" title="饼图 SVG" src="https://codepen.io/foreverZ133/embed/vYYwdYz?height=170&theme-id=default&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/vYYwdYz'>饼图 SVG</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>