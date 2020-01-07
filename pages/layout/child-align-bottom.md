# 三种底部对齐布局的实现

以下案例改变的是子级的高度，考虑性能要更多一些，  
最后发现，`flex + margin` 竟然是性能最差的。

### flex 系列
```css
.wrap {
  display: flex;
  align-items: flex-end;
}
```
```css
.wrap {
  display: flex;
}
.wrap > * {
  margin-top: auto;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="底部对齐 flex + margin" src="//codepen.io/foreverZ133/embed/jJVObx/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jJVObx/'>底部对齐 flex + margin</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### table-cell
```css
.wrap {
  display: table;
}
.wrap > * {
  display: table-cell;
  vertical-align: bottom;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="底部对齐 table-cell" src="//codepen.io/foreverZ133/embed/LqqMJG/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/LqqMJG/'>底部对齐 table-cell</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### inline-block
```css
.wrap:before {
  content: "";
  height: 100%;
}
.wrap {
  letter-spacing: -1em;
}
.wrap:before,
.wrap > * {
  display: inline-block;
  vertical-align: bottom;
  letter-spacing: 0;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="底部对齐 inline-block" src="//codepen.io/foreverZ133/embed/QYYzeJ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/QYYzeJ/'>底部对齐 inline-block</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### absolute + margin
```css
.wrap {
  position: relative;
}
.wrap > * {
  position: absolute;
  top: 0; bottom: 0;
  margin-top: auto;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="底部对齐 absolute" src="//codepen.io/foreverZ133/embed/Rdowgp/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/Rdowgp/'>底部对齐 absolute</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>