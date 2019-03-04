# 两种文本均匀留隙的实现

也可以叫做两端对齐吧，类似 `justify-content: space-between` 那样的效果。
```css
.wrap { width: 5em; display: inline-block; border: 1px solid red; vertical-align: middle; }
```

还需注意的其他问题：
* 子级个数为 1 时的情况
* 子级宽度超出时的情况

## 一、inline-block + text-align
```css
.wrap {
  text-align: justify;
  height: 1.5em;  /* 需要定高，不然会有诡异的空隙 */
  line-height: 1.5em;
}
.wrap:after {
  content: "";
  width: 100%;
  height: 0;
  display: inline-block;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="文本留隙对齐 inline-block + text-align" src="//codepen.io/foreverZ133/embed/pYbKyp/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/pYbKyp/'>文本留隙对齐 inline-block + text-align</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 二、text-align-last
```css
.wrap {
  text-align-last: justify;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="文本留隙对齐 text-align-last" src="//codepen.io/foreverZ133/embed/jJrKrG/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jJrKrG/'>文本留隙对齐 text-align-last</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 其他、将文字拆分为元素

那就得去看元素篇的均匀留隙咯，[案例](./pages/child-align-justify)