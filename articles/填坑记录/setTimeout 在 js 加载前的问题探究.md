# setTimeout 在 js 加载前的问题探究

同事放出一道题目，深究一下发觉很有意思

```js
<script>
  setTimeout(() => {
    alert('2');
  }, 0);
</script>
<script src="https://test.tms-uat.xuebangsoft.net/plugins/jquery-1.10.2.min.js"></script>
<script>
  alert('1')
</script>
```

众所周知， setTimeout 会进入事件循环，等待主线程空闲时才运行。<br />所以理论上来看，肯定都会先弹 1 再弹 2。

而当我 shift+F5 强刷时（或小概率），是先弹 2 的，后续直接 F5 还是先弹 1。<br />而如果将中间的 js 加载去掉，也不会先弹 2，<br />假如，setTimeout 前面也加上 js 加载，也不会先弹 2。

可见，此处好像 js 加载产生了一点微妙的化学反应。

后来通过 chrome 的 Performance 面板进行性能分析，<br />发现两种情况下的代码先后顺序是不同的：

先弹 2 时的代码顺序：<br />![](https://s1.ax1x.com/2020/03/12/8memFI.png)

先弹 1 时的代码顺序：<br />![](https://s1.ax1x.com/2020/03/12/8megt1.png)

此时答案就比较明显了：<br />先弹 2 是由于 js 加载阻断了后面 alert 代码的运行，<br />而使得主线程还未有代码，因此 setTimeout 立马循环到而先跑完了；<br />而先弹 1 则是 js 已被缓存，后续代码立马进入主线程，setTimeout 在事件循环中就靠后了。

虽然实操中不太可能会遇到这样的使用场景，<br />但对浏览器的运行规则又有了一些别样的眼光。

---

在试验中，会更换多个浏览器查看，结果发现另一个有趣的现象：<br />各浏览器对事件循环的处理好像并不标准呢，而且 ajax 在 chrome 竟然也是和 Promise 一样的微循环哟。

```js
setTimeout(function() {
  alert(1);
}, 0);

for (var i = 0; i < 1e8; i++) {}
new Promise(resolve => {
  resolve();
}).then(function() {
  alert(2);
});

for (var i = 0; i < 1e8; i++) {}
requestAnimationFrame(function() {
  alert(3);
});

for (var i = 0; i < 1e8; i++) {}
$.get('./list.html', function() {
  alert(4);
});

for (var i = 0; i < 1e8; i++) {}
alert(5);

// chrome 结果 5 2 3 4 1
// firefox 结果 5 4 2 1 3
```
