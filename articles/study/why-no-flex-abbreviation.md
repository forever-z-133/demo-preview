# 为何我反对使用 flex: 1 简写

众所周知 `flex: 1` 其实是 ` flex: 1 1 0` 的简写，而我为什么会反对这样写呢，<br />
当然是因为 `flex-basic: 0` 的表现咯，在浏览器支持的理解上是不同的。<br />

另外文本宽度在 `flex-grow` 和 `overflow: hiiden` 的搭配下也有些许不同。<br />

本文将细究一下浏览器在宽度计算时的规律和策略，以此来理解为何 `flex-basic: 0` 是有问题的。<br />

## width: auto

在张鑫旭大佬的《CSS世界》书中有提到，display 盒子其实可以看做是多个盒子共同作用的结果，<br />

比如 `inline-block` 可认为是 `inline` 包了个 `block` 合成的，`list-item` 是 `marker box` 和 `block` 结合才有了圆点；<br />
更甚至，`block` 下的伪元素可能也是 `block > (before-inline + block + after-inline)` 这样的插在其中的嵌套结构。<br />
_注意，这里不是 display: inline 或 inline element 意思，而是 inline box 的概念。_<br />

大多文章都把 `dom tree` 和 `cssom tree` 合成 `render tree` 这里一笔带过，<br />
我看 w3c 官方文档也不多，以下论述 **仅个人总结和理解**，还请大佬们勘误。<br />

`block box` 的初始宽度实为父级宽度，然后才读取的 `width` 等属性再给出新的实际宽度，<br />
`inline box` 的初始宽度实为本元素宽度，然后判断父级剩余空间，若超出则以父级宽度为准。<br />

听上去就很容易死循环不是吗，比如子级宽度变了，父级可能会变，父级变了祖父级可能也得变。<br />

我大胆猜测一下，其实只要有个稳定的父级宽度，即可避免掉上面这种死循环的，<br />
比如当本元素宽度变化时，向父级寻找到 `block element` 即可，该父级之下的所有元素直接重新计算和渲染。<br />

## min/max-width

如果说有什么能比 `width: 10px !important` 权限更高的话，那 `max-width` 就会说“没错，正是在下”了。<br />

## BFC

当 `float` 时，元素宽度的处理，及其对父级宽度的影响稍有不同。<br />

至于 BFC 中的 `position` `inline-block` `overflow` 等属性的情况，也有所不同，但篇幅有限就不展开了。<br />

## transform



## flex-shrink

```html
<!-- 后续演练皆以此结构为例 -->
<style>
  .list { display: flex; width: 500px; }
  .list > div { height:50px; }
  .item-1 { background: pink; }
  .item-2 { background: lightblue; }
  .item-3 { background: gray; }
</style>

<div class="list">
  <div class="item-1"></div>
  <div class="item-2"></div>
  <div class="item-3"></div>
</div>
```

`flex-shrink` 默认为 `1`，表示弹性子元素在空间不足时等比压缩。<br />

这意味着，假若总宽为 `300px` 且子元素各有 `5, 10, 15` 个字且宽度总和超出父级宽度，<br />
那么子元素的宽度将会被分到 `50px, 100px, 150px` 的收缩后宽度，保持着等比。<br />

<iframe height="169" style="width: 100%;" scrolling="no" title="flex-shrink: 1" src="https://codepen.io/foreverZ133/embed/abZyRQK?height=169&theme-id=light&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/abZyRQK'>flex-shrink: 1</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

而当

## flex-grow

`flex-grow` 默认为 `0`，表示弹性子元素不参与分配剩余空间。<br />

## flex-basic

猜猜看，下面三个子元素的宽度分别会是多少？
```css
.item-1 { flex: 1 1 0%; }
.item-2 { flex: 1 1 auto; }
.item-3 { flex: 1 1 200px; }
```

<img src="https://s1.ax1x.com/2020/10/28/B1N9sI.jpg" alt="案例1"><br />
结果共计 `500px`，各占了 `100px, 100px, 300px`，<br />
尚未分配时，子元素宽度为 `0 0 200px`，空间有余，再按 `1, 1, 1` 的 `flex-grow` 分配。

显然，flex-basic 会更与分配前的宽度计算更相关，比如 max-width 那些。



## 文件

https://lisongfeng.cn/2018/08/04/flexbox-essentials.html<br />
