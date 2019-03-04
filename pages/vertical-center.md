# 九种垂直居中样式

```html
<div class="wrap">
  <div class="box"></div>
</div>
```
```css
.wrap { width: 200px; height: 200px; border: 1px solid }
.box { width: 100px; height: 100px; background: red; }
```

还需考虑的事情：
* 子级高度大于父级高度时是否仍需居中
* 子级是否有多个元素，是否都需垂直居中

### 方法 1：vertical-align: middle;

**其他缺点：** 部分浏览器 `inline-block` 有左右空隙。

```css
.wrap:before {
  content: '';
  height: 100%;
  display: inline-block;
  vertical-align: middle;
}
.wrap > * {
  display: inline-block;
  vertical-align: middle;
}
```

<iframe height="360" style="width: 100%;" scrolling="no" title="QYmRvJ" src="//codepen.io/foreverZ133/embed/QYmRvJ/?height=360&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/QYmRvJ/'>QYmRvJ</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 2：table-cell

**其他缺点：** 1. 对父级布局可能会有一丢丢破坏性; 2. 父级宽度不再支持百分比。

```css
.wrap {
  display: table-cell;
  vertical-align: middle;
}
```

<iframe height="360" style="width: 100%;" scrolling="no" title="垂直居中 table-cell" src="//codepen.io/foreverZ133/embed/NoYZKV/?height=360&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/NoYZKV/'>垂直居中 table-cell</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 3：flex 布局

```css
.wrap {
  display: flex;
  align-items: center;
}
```

子级用 `align-self` 也可以。

```css
.wrap {
  display: flex;
}
.wrap > * {
  align-self: center;
}
```

<iframe height="340" style="width: 100%;" scrolling="no" title="垂直居中 align-items" src="//codepen.io/foreverZ133/embed/gqeNwq/?height=340&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/gqeNwq/'>垂直居中 align-items</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 4：flex + margin

```css
.wrap {
  display: flex;
}
.wrap > * {
  margin: auto 0;
}
```

<iframe height="310" style="width: 100%;" scrolling="no" title="垂直居中 flex + margin" src="//codepen.io/foreverZ133/embed/ErEBWv/?height=310&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/ErEBWv/'>垂直居中 flex + margin</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 5：absolute + translate

**其他缺点：** 子级未设宽度时，宽度可能会有点问题。

```css
.wrap {
  position: relative;
}
.wrap > * {
  position: absolute;
  top: 50%;
  transform: translate(0, -50%);
}
```

<iframe height="340" style="width: 100%;" scrolling="no" title="垂直居中 absolute + translate" src="//codepen.io/foreverZ133/embed/aXYgyV/?height=340&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/aXYgyV/'>垂直居中 absolute + translate</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 6：absolute + margin

**其他缺点：** 子级需定高，哪怕是 inline。

```css
.wrap {
  position: relative;
}
.wrap > * {
  position: absolute;
  top: 0;
  bottom: 0;
  margin: auto 0;
}
```

<iframe height="340" style="width: 100%;" scrolling="no" title="垂直居中 absolute + margin" src="//codepen.io/foreverZ133/embed/bzvPKd/?height=340&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/bzvPKd/'>垂直居中 absolute + margin</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 7：line-height

**其他缺点：** 1. 父级需定高且已知； 2. 子级需为行内元素； 3. 文本换行问题。

```css
.wrap {
  line-height: 200px;
}
.wrap > * {
  display: inline-block;
  vertical-align: middle;
}
```

<iframe height="390" style="width: 100%;" scrolling="no" title="垂直居中 line-height" src="//codepen.io/foreverZ133/embed/BMrgGy/?height=390&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/BMrgGy/'>垂直居中 line-height</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 8：-webkit-box + -webkit-box-align

**其他缺点：** 兼容性问题。

```css
.wrap {
  display: -webkit-box;
  -webkit-box-align: center;
}
```

<iframe height="310" style="width: 100%;" scrolling="no" title="垂直居中 -webkit-box + -webkit-box-align" src="//codepen.io/foreverZ133/embed/mvxNVd/?height=310&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/mvxNVd/'>垂直居中 -webkit-box + -webkit-box-align</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 9：grid 布局

**其他缺点：** 兼容性问题。

```css
.wrap {
  display: grid;
  grid-template-rows: 1fr auto 1fr;
}
.wrap:before {
  content: "";
}
```

子级用 `align-self` 也可以。

```css
.wrap {
  display: grid;
}
.wrap > * {
  align-self: center;
}
```

或者 `grid-row` 等各式花样操作。

```css
.wrap {
  display: grid;
  grid-row: 1 / span 1;
}
```

<iframe height="360" style="width: 100%;" scrolling="no" title="垂直居中 grid + gird-template-rows" src="//codepen.io/foreverZ133/embed/QYrzrg/?height=360&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/QYrzrg/'>垂直居中 grid + gird-template-rows</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 方法 10：已知子级高度

此时是用 `:before + margin` 或者 `padding` 或者 `margin 负值` 或者 `calc 计算` 等等都是可以的，毕竟只需纯计算即可。