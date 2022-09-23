# 黑暗模式效果的实现

首先要有一个白底，然后再有一个 `mix-blend-mode: difference` 即可。

```css
.dark-mode-back-layer,
.dark-mode-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  pointer-events: none;
}
.dark-mode-back-layer {
  z-index: -1;
}
.dark-mode-layer {
  background: transparent;
  transition: background 1s linear;
}
.dark-mode-layer.on {
  mix-blend-mode: difference;
  background: #fff;
  z-index: 9999;
}
.dark-mode-ignore {
  position: relative;
  z-index: 9999;
  /* isolation: isolate 不知咋没用 */
  /* 另外注意 relative 覆盖其他样式的问题 */
}
```

或者用 `@media` 来处理 `.on` 状态。

```css
:root {
  /* 告诉浏览器我支持两种模式 */
  color-scheme: light dark;
}
@media (prefers-color-scheme: dark) {
  .dark-mode-layer {
    mix-blend-mode: difference;
    background: #fff;
    z-index: 9999;
  }
}
```

```html
<body>
  <div class="dark-mode-back-layer"></div>
  <div class="dark-mode-layer"></div>
  <div id="app">
    <button id="change">切换黑暗模式</button>
    <div class="image-box dark-mode-ignore">
      <img src="https://picsum.photos/300/200" width="300" height="200" />
    </div>
    <p>文字文字文字文字文字文字文字文字文字文字</p>
  </div>
</body>
```

```js
const $change = document.querySelector("#change");
const $darkMode = document.querySelector(".dark-mode-layer");
$change.addEventListener("click", () => {
  $darkMode.classList.toggle("on");
});
```

<iframe height="390" style="width: 100%;" scrolling="no" title="暗黑模式切换" src="https://codepen.io/foreverZ133/embed/RwWWrPZ?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true" loading="lazy">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/RwWWrPZ'>暗黑模式切换</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

其他参考：<br />
https://github.com/forever-z-133/others/issues/170<br />
https://darkmodejs.learn.uno/<br />
