# 左右布局与左中右布局

也即自适应多栏布局，例如圣杯双飞翼等都在此范畴内。<br />

还需考虑的其他问题：
* dom 元素的顺序
* 左右间隙
* 多个左右元素
* 多个自适应元素

*至于 `flex` 和 `grid` 类的布局就不写咯。*

## 一、BFC

注意：1. 浮动元素必须在自适应元素之前； 2. `margin` 不可设在自适应元素上。

```scss
.wrap {
  overflow: hidden; /* 处理高度塌陷 */

  & > .left {
    float: left;
  }
  & > .right {
    float: right;
  }
  & > .grow {
    overflow: hidden; /* 独立于 float */
  }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="自适应布局 BFC" src="//codepen.io/foreverZ133/embed/wvwzvEj/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/wvwzvEj/'>自适应布局 BFC</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 二、table

注意：1. 高度都是最高元素的高度； 2. 间隙得靠子孙元素。

```scss
.wrap {
  width: 100%;
  display: table;

  & > * {
    display: table-cell;
  }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="自适应布局 table" src="//codepen.io/foreverZ133/embed/abombXv/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/abombXv/'>自适应布局 table</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 三、定位

注意：1. 多个元素时要计算； 2. 父级高度只有自适应元素的高度。<br />
但相对比较稳定，且更容易加入滚动效果。

```scss
.wrap {
  position: relative;
  padding-left: 150px;
  padding-right: 250px;

  & > .left, & > .right {
    position: absolute;
    top: 0;
  }
  & > .left { left: 0; }
  & > .right { right: 0; }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="自适应布局 定位" src="//codepen.io/foreverZ133/embed/OJLRPvq/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/OJLRPvq/'>自适应布局 定位</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 四、圣杯布局

间隙要算，多个元素也不好搞，也不知当初是怎么流行的。

```scss
.wrap {
  overflow: hidden;
  padding-left: 150px;
  padding-right: 250px;

  & > * {
    float: left;
  }
  & > .left {
    margin-left: -150px;
  }
  & > .right {
    margin-right: -250px;
  }
  & > .grow {
    width: 100%;
  }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="自适应布局 圣杯" src="//codepen.io/foreverZ133/embed/vYBXEaz/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/vYBXEaz/'>自适应布局 圣杯</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 五、双飞翼布局

和圣杯布局其实差不多，父级宽度不一样且不宜添加 `overflow`，故还需要加上 `.clearfix` 类。

```scss
.wrap {
  margin-left: 150px;
  margin-right: 250px;

  .clearfix {
    clear: both;
    float: none;
  }
  & > * {
    float: left;
  }
  & > .left {
    margin-left: -150px;
  }
  & > .right {
    margin-right: -250px;
  }
  & > .grow {
    width: 100%;
  }
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="自适应布局 双飞翼" src="//codepen.io/foreverZ133/embed/YzKGawX/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/YzKGawX/'>自适应布局 双飞翼</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>