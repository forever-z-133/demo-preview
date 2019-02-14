# 三种文本溢出省略

```html
<div class="box text-overflow">
  内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容
</div>
```
```css
.box { width: 10em; padding: 5px 12px; background: #eee; border-radius: 4px; }
```

## 方法 1：text-overflow

```css
.text1 {
  word-wrap: normal;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

<iframe height="265" style="width: 100%;" scrolling="no" title="文本溢出省略 text-overflow: ellipsis;" src="//codepen.io/foreverZ133/embed/RvYovR/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/RvYovR/'>文本溢出省略 text-overflow: ellipsis;</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法2：-webkit-box + -webkit-line-clamp

**需注意：** 兼容性问题，比如 firefox 至今不支持

```css
.text-overflow {
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: break-word;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
```

<iframe height="265" style="width: 100%;" scrolling="no" title="文本溢出省略 -webkit-box + -webkit-line-clamp" src="//codepen.io/foreverZ133/embed/jdvVRz/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/jdvVRz/'>文本溢出省略 -webkit-box + -webkit-line-clamp</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 方法 3：float + 伪元素

**需注意：** 1. 背景色需控制； 2. 需要多一层嵌套； 3. 需根据行数修改高度

```html
<div class="box text-overflow">
  <span>内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容</span>
</div>
```
```scss
.text-overflow {
  overflow: hidden;
  height: 3em;
  line-height: 1.5em;
  
  &:before {
    content: "";
    float: left;
    width: 1px;
    height: 100%;
  }
  & > *:first-child {
    float: right;
    width: 100%;
    margin-left: -5px;
  }        
  &:after {
    content: "\02026"; /* 字符 ... */
    position: relative;
    float: right;
    top: -25px;
    left: 100%;
    width: 3em;
    margin-left: -3em;
    padding-right: 5px;
    text-align: right;
    background: linear-gradient(to right, rgba(255, 255, 255, 0), white 50%, white);
  }
}
```

<iframe height="265" style="width: 100%;" scrolling="no" title="三种文本溢出省略 float + 伪元素" src="//codepen.io/foreverZ133/embed/Odobeg/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/Odobeg/'>三种文本溢出省略 float + 伪元素</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 扩展阅读：不同宽度时显示不同文本
```html
<span class="overflower control">
  <span class="overflower-short">Short text here is.</span>
  <span class="overflower-long">Some long text that could become shorter.</span>
</span>
```
```css
.control {
  resize: horizontal;
}
.overflower {
  height: 1.5em;
  line-height: 1.5em;
  display: inline-block;
  overflow: hidden;
  max-width: 100%;
  white-space: nowrap;
  text-overflow: ellipsis;
  display: inline-flex;
  flex-wrap: wrap;
}
.overflower-short {
  width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  flex-grow: 1;
}
.overflower-long {
  display: inline;
  flex-basis: 100%;
}
```

在子级中继续嵌套 `.overflower` 可实现更多宽度变化下的不同文本。

<iframe height="265" style="width: 100%;" scrolling="no" title="不同宽度显示不同文字" src="//codepen.io/foreverZ133/embed/XOPpYb/?height=265&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/XOPpYb/'>不同宽度显示不同文字</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>