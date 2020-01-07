# 兼容性的 Polyfill 代码

* [on](#-绑定事件)（绑定事件）

## * 绑定事件
```js
// 可简化 addEventListener 后期再加入事件代理模式
// 直接 window.on('click', () => {});
(function(arr) {
  arr.forEach(function(item) {
    if (item.on) return;
    Object.defineProperty(item, 'on', {
      configurable: true,
      enumerable: true,
      writable: true,
      value: function on(event, fn) {
        this.addEventListener(event, fn);
      }
    });
  })
})([Element.prototype, window])
```
