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

### 其他、inline-block + text-align

类似文字的做法，可看 [案例](./pages/text-align-justify)

### 其他、已知空隙 calc 计算
栅栏布局时代，`col-xs-3` 想补空隙就需要 **加子级** 再 `margin`。  
而 `calc` 稍微能减轻一丢丢吧，但这是张鑫旭大佬并不推荐的做法，宽度设置最好分离开。
```less
/* 求分栏且留隙时，每栏的宽度 */
.col-width(@column, @gap: 0) {
  width: calc(100% / @column - @gap * 2px);
  margin-left: @gap * 1px;
  margin-right: @gap * 1px;
}

.wrap { overflow: hidden; }
.item {
  float: left;
  .col-width(3, 10);
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级留隙 calc 计算" src="//codepen.io/foreverZ133/embed/moEKdy/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/moEKdy/'>子级留隙 calc 计算</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>