# 遮罩/蒙版/裁剪效果

![遮罩/蒙版的概念示意](https://user-images.githubusercontent.com/22021589/32707107-92dea508-c85c-11e7-9d86-bb09ca2c40c7.jpg)

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

SVG 的 `mask` 如果与内容宽高适配还没摸清楚，继续研究中。

<iframe height="265" style="width: 100%;" scrolling="no" title="BMORmQ" src="//codepen.io/foreverZ133/embed/BMORmQ/?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/BMORmQ/'>BMORmQ</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### clip-path（css）
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
### globalCompositeOperation（canvas）

拓展：其他属性值的效果，[预览](https://foreverz133.github.io/demos/single/globalCompositeOperation.html)

### background-clip（css）
### 扩展阅读
https://github.com/foreverZ133/blogs/issues/21  
https://www.zhangxinxu.com/wordpress/2016/03/better-black-mask-guide-overlay-method/  
https://codepen.io/airen/pen/VPKQxb  
https://foreverz133.github.io/demos/works/mask/
