# 四种 1px 毛细线样式

毛细线样式，即研究 1px 问题，<br />
即是处理高 dpr 设备上 1px 的 border 看上去像 2px 一样粗。

<iframe height="500" style="width: 100%;" scrolling="no" title="1px border" src="//codepen.io/foreverZ133/embed/OJLNezZ/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/OJLNezZ/'>1px border</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 一、@media
```css
.border { border-bottom: 1px solid #999 }
@media screen and (min-device-pixel-ratio: 2) {
  .border { border-bottom: 0.5px solid #999 }
}
@media screen and (min-device-pixel-ratio: 3) {
  .border { border-bottom: 0.333333px solid #999 }
}
```

### 二、background

有三种设置 `background` 的办法，更推荐 base64 图片，<br />
渐变或 svg 亦可，并且可修改颜色。

```scss
@media screen and (min-device-pixel-ratio: 2) {
  .border {
    border-bottom: none;
    background-size: 100% 1px;
    background-repeat: no-repeat;
    background-position: top center;
    background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAACCAYAAACZgbYnAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAAQSURBVBhXY5g5c+Z/BhAAABRcAsvqBShzAAAAAElFTkSuQmCC');
    // background-image: linear-gradient(to top, transparent 50%, #999 50%);
    // background-image: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" height="1px"><rect y=".5" width="100%" height="0.5" fill="%23999" /></svg>');
  }
}
@media screen and (min-device-pixel-ratio: 3) {
  .border {
    // background-image: linear-gradient(to top, transparent 66.666%, #999 66.666%);
    // background-image: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" height="1px"><rect y=".666" width="100%" height="0.333" fill="%23999" /></svg>');
  }
}
```

### 三、scale
```scss
@media screen and (min-device-pixel-ratio: 2) {
  .border {
    position: relative;
    border-bottom: none;

    &:before {
      content: '';
      position: absolute;
      bottom: 0; left: 0; right: 0;
      border-bottom: 1px solid #999;
      transform: scaleY(.5);
    }
  }
}
@media screen and (min-device-pixel-ratio: 3) {
  .border {
    &:before {
      transform: scale(.333);
    }
  }
}
```