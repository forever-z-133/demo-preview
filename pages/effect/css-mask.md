# 遮罩/蒙版/裁剪效果

![遮罩/蒙版的概念示意](https://s1.ax1x.com/2020/10/23/BAjdg0.jpg)

遮罩/蒙版/裁剪三个名词非常容易混淆，通俗点讲，

* 遮罩：图片在形状之上，形状有颜色，则显示图片内容;
* 蒙版：形状在图片之上，形状无颜色，则显示图片内容；
* 裁剪：将图片裁剪成相应形状，与颜色无关。

### -webkit-mask（css）
```scss
.mask-star {
  /* $starUrl 为星形的 base64 */
  /* 图形越不透明，则显示越清晰 */
  -webkit-mask: url($starUrl) center / contain no-repeat;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 -webkit-mask" src="//codepen.io/foreverZ133/embed/ZwMeaB/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/ZwMeaB/'>遮罩蒙版 -webkit-mask</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### mask（svg）
SVG 的 `mask` 如果与内容宽高适配还没摸清楚，继续研究中。
```
<svg xmlns="http://www.w3.org/2000/svg">
  <symbol id="star" viewbox="0 0 190 180">
    <polygon points="100,0 160,180 10,60 190,60 40,180" />
  </symbol>
  <defs>
    <mask id="mask">
      <!-- 图形颜色越白，则显示越清晰 -->
      <use xlink:href="#star" fill="rgba(255,255,255,1)" />
    </mask>
  </defs>
  <image id="image" xlink:href="https://picsum.photos/200/200" mask="url(#mask)" />
</svg>
```
<iframe height="265" style="width: 100%;" scrolling="no" title="BMORmQ" src="//codepen.io/foreverZ133/embed/BMORmQ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/BMORmQ/'>BMORmQ</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### clip-path（css）
注意：不推荐使用 CSS2 的 `clip`，不仅要脱离文档流且只支持 `px` 单位。
```scss
.mask-diamond {
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}
```

<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 CSS clip-path" src="//codepen.io/foreverZ133/embed/vbzZNy/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/vbzZNy/'>遮罩蒙版 CSS clip-path</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### clip-path（svg）


### clip（canvas）
注意：在微信小程序中此方案性能十分不佳
```js
ctx.save();
ctx.rect(x, y, width, height); // 只需要路径
ctx.clip();
// do something
ctx.restore();
```
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 canvas clip" src="//codepen.io/foreverZ133/embed/mvvvRx/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/mvvvRx/'>遮罩蒙版 canvas clip</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### globalCompositeOperation（canvas）
注意：本 save 范围之外有底色，也会被看作有底色，且底色透明度都可继承，实在是不太妙的一种方案。  
拓展：`globalCompositeOperation` 的其他属性值也很有意思，[预览](https://foreverz133.github.io/demos/single/globalCompositeOperation.html)。
```js
ctx.save();
ctx.fillRect(x, y, width, height);  // 需要有底色
ctx.globalCompositeOperation = 'source-in';
// do something
ctx.restore();
```
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 canvas globalCompositeOperation" src="//codepen.io/foreverZ133/embed/jJaRoB/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jJaRoB/'>遮罩蒙版 canvas globalCompositeOperation</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### temp canvas（canvas）
```js
const _canvas = document.createElement('canvas');
_canvas.width = width;
_canvas.height = height;
const _ctx = _canvas.getContext('2d');
_ctx.translate(-x, -y);
// do something
ctx.drawImage(_canvas, x, y, width, height);
```
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 临时 canvas " src="//codepen.io/foreverZ133/embed/xBPNbQ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/xBPNbQ/'>遮罩蒙版 临时 canvas </a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


### 其他、-webkit-background-clip（css）
```css
.wrap {
  background: linear-gradient(to right, pink, darkred);
  -webkit-background-clip: text;
  color: transparent;
}
```
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 background-clip" src="//codepen.io/foreverZ133/embed/vPWwbX/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/vPWwbX/'>遮罩蒙版 background-clip</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 其他、镂空图片的裁剪（css）
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 镂空图片裁剪" src="//codepen.io/foreverZ133/embed/VRrJpb/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/VRrJpb/'>遮罩蒙版 镂空图片裁剪</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 其他、box-shadow（css）
<iframe height="265" style="width: 100%;" scrolling="no" title="遮罩蒙版 box-shadow" src="//codepen.io/foreverZ133/embed/gEXNNB/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/gEXNNB/'>遮罩蒙版 box-shadow</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 其他、shape-outside（css）
多半搭配 CSS 的 `clip-path`，使剪切后的元素的点击范围与元素形态也变得与形状一致。

### 扩展阅读
https://github.com/foreverZ133/blogs/issues/21  
https://www.zhangxinxu.com/wordpress/2016/03/better-black-mask-guide-overlay-method/  
https://codepen.io/airen/pen/VPKQxb  
https://foreverz133.github.io/demos/works/mask/
