# 四种瀑布流布局的实现

```html
<div class="masonry-list-box">
  <div class="masonry-list list"></div>
</div>
```
```css
.masonry-list:empty:before { content: '暂无数据'; display: block; text-align: center; padding: 50px 0; }
.title, .desc { margin: 0; }
```

### 一、CSS3 的 column 属性

非常简陋的瀑布流，且请 **特别注意** 子级顺序哟。

```css
.list img { width: 100%; }
.list {
  column-count: 5;
  column-gap: 20px;
}
.item {
  /* 注意：这里很关键，不然子级会分离到下一栏  */
  height: 100%;
  overflow: auto;
}
```

<iframe height="340" style="width: 100%;" scrolling="no" title="瀑布流布局 CSS column" src="//codepen.io/foreverZ133/embed/oVbGyZ/?height=340&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/oVbGyZ/'>瀑布流布局 CSS column</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 二、仿百度图片的 flex-grow 横排瀑布流

子级宽度是不受控制的哟，且对 `object-fit` 的兼容性有要求。

```css
.list {
  display: flex;
  flex-wrap: wrap;
}
.item {
  position: relative;
  flex-grow: 1;
  height: 100px;
  margin: 5px;
}
.item div {
  height: 100%;
  display: block;
}
.item img {
  height: 100%;
  max-width: 100%;
  min-width: 100%;
  object-fit: cover;
  display: block;
}
```

<iframe height="460" style="width: 100%;" scrolling="no" title="瀑布流布局 CSS flex-grow" src="//codepen.io/foreverZ133/embed/QoyqoP/?height=460&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/QoyqoP/'>瀑布流布局 CSS flex-grow</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 三、仿花瓣的多栏竖排瀑布流

获取高度挺耗性能的，虽然有很多优化途径，但感觉还是不太妙。<br />
再者，如果某栏突然有条超长，其实会比较丑。

```js
var $list = document.querySelector('.masonry-list');
var columns = new Array(5).fill();

// 先加五个分栏
$list.innerHTML = columns.reduce((re, item) => {
  return re + '<div class="column"></div>';
}, '');

// 根据分栏中已有的高度计算添加到哪栏中去
var $columns = $list.querySelectorAll('.column');
var columnWidth = $columns[0].offsetWidth;
columns = columns.map((item, index) => ({ index: index, height: 0 }));
listData.reduce((re, item) => {
  var min = columns.sort((a, b) => b.height - a.height).slice(-1)[0];
  $columns[min.index].innerHTML += getItemHTML(item); // 可优化为一次添加
  min.height += getItemHeight(item); // 这里没算 img 的高度，需优化
  return re;
}, columns);
```

<!-- <iframe height="400" style="width: 100%;" scrolling="no" title="瀑布流 分栏式计算" src="//codepen.io/foreverZ133/embed/rRxYzj/?height=400&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/rRxYzj/'>瀑布流 分栏式计算</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe> -->

<iframe height="265" style="width: 100%;" scrolling="no" title="jquery 版瀑布流" src="https://codepen.io/foreverZ133/embed/vYLpYOY?height=265&theme-id=light&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/vYLpYOY'>jquery 版瀑布流</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 四、纯 js 计算的 float 瀑布流

很难算，整理中...<br />
另外这渲染性能极差，小动一下都会重排。

### 其他：固定宽高的 grid 瀑布流

其实用 `table` 合并单元格也是类似的原理，但数量宽高等都是定死的就对了。

<iframe height="400" style="width: 100%;" scrolling="no" title="瀑布流 CSS grid 固定宽高" src="//codepen.io/foreverZ133/embed/JzGMxg/?height=400&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/JzGMxg/'>瀑布流 CSS grid 固定宽高</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>