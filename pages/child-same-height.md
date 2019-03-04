# 四种子级等高布局的实现

```html
<div class="wrap">
  <div class="box left-box"></div>
  <div class="box center-box"></div>
  <div class="box right-box"></div>
</div>
```
```css
.left { width: 150px; background: #00FFFF; }
.center { width: 250px; background: #FF0000; }
.right { width: 100px; background: #00FF00; }
```

还需考虑的其他问题：
* 子级是否需定宽
* 如果父级宽度不足是否会换行
* 子级间是否有间隙

### 一、margin + padding
```css
.wrap {
  overflow: hidden;
}
.wrap > .box {
  margin-bottom: -10000px;
  padding-bottom: 10000px;
  float: left;
}
```
<iframe height="320" style="width: 100%;" scrolling="no" title="子级等高 padding + margin" src="//codepen.io/foreverZ133/embed/zbqZzj/?height=320&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/zbqZzj/'>子级等高 padding + margin</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 二、table-cell
```css
.wrap > .box {
  display: table-cell;
}
```
<iframe height="250" style="width: 100%;" scrolling="no" title="子级等高 table-cell" src="//codepen.io/foreverZ133/embed/JzXWre/?height=250&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/JzXWre/'>子级等高 table-cell</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 三、flex / grid 特性
```css
.wrap {
  display: flex;
}
```
<iframe height="250" style="width: 100%;" scrolling="no" title="子级等高 flex" src="//codepen.io/foreverZ133/embed/NJNpXb/?height=250&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/NJNpXb/'>子级等高 flex</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

```css
.wrap {
  display: grid;
  grid-template-columns: repeat(3, auto) 1fr;
}
```
<iframe height="250" style="width: 100%;" scrolling="no" title="子级等高 grid" src="//codepen.io/foreverZ133/embed/jJqBzw/?height=250&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jJqBzw/'>子级等高 grid</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 四、视觉欺骗：背景色

```css
.wrap {
  overflow: hidden;
  background-image: linear-gradient(to right, #00FFFF 150px, #FF0000 150px, #FF0000 400px, #00FF00 400px, #00FF00 500px, rgba(0,0,0,0) 500px);
}
.wrap > .box {
  background: none;
  float: left;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="子级等高 背景色" src="//codepen.io/foreverZ133/embed/KEzWEZ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/KEzWEZ/'>子级等高 背景色</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 其他、视觉欺骗：边框

这是相当拙劣的一种方案，用来充数看看就好，项目中很难会这样来用的。

<iframe height="265" style="width: 100%;" scrolling="no" title="子级等高 边框" src="//codepen.io/foreverZ133/embed/gErmJM/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/gErmJM/'>子级等高 边框</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


### 其他、min-height 定值

当已知某子级最大高度时，给所有子级定值 min-height 也是种办法。  
如果未知或强调精确，JS 获取由于图片高度问题其实也不美妙，所以还是采用上述几种吧。