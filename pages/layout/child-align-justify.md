# 三种子级均匀留隙的实现

### 一、flex + space-between
```css
.wrap {
  display: flex;
  justify-content: space-between
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级留隙 flex + space-between" src="//codepen.io/foreverZ133/embed/KEMeNZ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/KEMeNZ/'>子级留隙 flex + space-between</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 二、flex + margin: auto
```css
.wrap { display: flex; }
.wrap > :not:(:last-child) {
  margin-right: auto;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级留隙 flex + margin: auto" src="//codepen.io/foreverZ133/embed/NJrzdg/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/NJrzdg/'>子级留隙 flex + margin: auto</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 三、grid
```css
.wrap {
  display: grid;
  grid-template-columns: 50px 1fr 50px 1fr 50px;
}
.item:nth-child(1) { grid-column-start: 1; }
.item:nth-child(2) { grid-column-start: 3; }
.item:nth-child(3) { grid-column-start: 5; }
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级留隙 grid" src="//codepen.io/foreverZ133/embed/XGKYeo/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/XGKYeo/'>子级留隙 grid</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 四、inline-block + text-align

类似当做文本处理的做法，可看 [案例](./pages/layout/text-align-justify)

### 其他、calc 计算

类似下方代码中这样，纯计算的。可能会挑战性能。

```css
.wrap > :not(:last-child) {
  margin-right: calc((100% - 100px - 200px) / 2);
}
```

### 其他、margin

给固定的间隙，特别适用于等宽分栏的布局，但需要三层的结构才行。

```css
.wrap {
  overflow: hidden; /* 防止 .list 溢出 */
}
.list {
  overflow: hidden; /* 清除浮动的 */
  margin: 0 -5%;
}
.item {
  float: left;
  margin: 0 2.5%;
  width: 45%;
  height: 50px;
  background: red;
}
```

### 其他：float

需注意元素顺序，且不适用于更多元素，但至少空隙是有了，虽然不均匀。

```scss
.wrap {
  overflow: hidden;
  & > .left { float: left; }
  & > .right { float: right; }
  & > .middle { overflow: hidden; margin: 0 auto; }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级留隙 float" src="https://codepen.io/foreverZ133/embed/YzKdRVw?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/YzKdRVw'>子级留隙 float</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
