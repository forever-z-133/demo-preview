# 解决 inline-block 的空隙问题

<iframe height="380" style="width: 100%;" scrolling="no" title="inline-block 空隙问题" src="//codepen.io/foreverZ133/embed/ywJQVr/?height=380&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/ywJQVr/'>inline-block 空隙问题</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 一、HTML 不换行
```html
<div class="wrap">
  <div class="item">1</div><div class="item">2</div>
</div>
```

### 二、letter-spacing: -1em
```css
.wrap { letter-spacing: -1em; }
.wrap > * { letter-spacing: 0; }
```

### 三、font-size: 0
```css
.wrap { font-size: 0; }
.wrap > * { font-size: 16px; }
```

### 四、display: table;
或者 `display: flex`，不过也可能破坏原预期效果
```css
.wrap { display: table; }
```

### 五、float: left
同上，也可能破坏原预期效果
```css
.wrap { float: left; }
```

### 其他、margin-left
不推荐，因为有的浏览器并没有间隙，这样写非常不兼容
```css
.wrap > :not(:first-child) { margin-left: -.3em }
```