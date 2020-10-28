# 为何我反对使用 flex: 1 简写

众所周知 `flex: 1` 其实是 ` flex: 1 1 0` 的简写，而我为什么会反对这样写呢，  
当然是因为 `flex-basic: 0` 的表现咯，在浏览器支持的理解上是不同的。  

另外文本宽度在 `flex-grow` 和 `overflow: hiiden` 的搭配下也有些许不同。  

本文将细究一下浏览器在宽度计算时的规律和策略，以此来理解为何 `flex-basic: 0` 是有问题的。  
_笔者看 w3c 不多，更常基于自研来理解，故文章所述并不见得正确，欢迎大佬勘误。_  

## width: auto

在张鑫旭大佬的《CSS世界》书中有提到，元素的盒子其实不止表现出来的一种，  
比如 inline-block 可认为是 inline 包了个 block 合成的，list-item 是 marker box 和 block 结合才有了圆点；  
更甚至，block 下的伪元素可能也是 block > (before-inline + block + after-inline) 这样的插在其中的嵌套结构。  

既然如此，inline 套 block ，或 block 套 inline，时候的宽度计算便是我们要思考的重点了。  
_注意，这里不是 display: inline 或 inline element 意思，而是 inline box 的概念。_

block box 的初始宽度实为父级宽度，然后才读取的 width 属性并给定新宽度，  
inline box 的初始宽度实为本元素宽度，然后判断父级剩余空间，若超出则压缩本元素宽度为父级宽度。  
那 inline box 套 block box 听上去就挺死循环的不是吗。

个人暂未找到实际渲染算法方面的资料，只能靠以往写 canvas 图形库的经验来猜测了。  
在 dom tree 阶段我们就拿到了 inline box 的初始宽度和部分宽度属性，  
在 cssdom tree 这边时开始处理 block box 的初始宽度，以及两者的结果宽度。  

```css
.parent { display: inline; }
```

## min/max-width

如果说有什么能比 `width: 10px !important` 权限更高的话，那 `max-width` 就会说“没错，正是在下”了。  

## BFC

当 `float` 时，元素宽度的处理，及其对父级宽度的影响稍有不同。  

至于 BFC 中的 `position` `inline-block` `overflow` 等属性的情况，也有所不同，但篇幅有限就不展开了。  

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

`flex-shrink` 默认为 `1`，表示弹性子元素在空间不足时等比压缩。  

这意味着，假若总宽为 `300px` 且子元素各有 `5, 10, 15` 个字且宽度总和超出父级宽度，  
那么子元素的宽度将会被分到 `50px, 100px, 150px` 的收缩后宽度，保持着等比。  

<iframe height="169" style="width: 100%;" scrolling="no" title="flex-shrink: 1" src="https://codepen.io/foreverZ133/embed/abZyRQK?height=169&theme-id=light&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/abZyRQK'>flex-shrink: 1</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

而当

## flex-grow

`flex-grow` 默认为 `0`，表示弹性子元素不参与分配剩余空间。  

## flex-basic

猜猜看，下面三个子元素的宽度分别会是多少？
```css
.item-1 { flex: 1 1 0%; }
.item-2 { flex: 1 1 auto; }
.item-3 { flex: 1 1 200px; }
```

<img src="https://s1.ax1x.com/2020/10/28/B1N9sI.jpg" alt="案例1">  
结果共计 `500px`，各占了 `100px, 100px, 300px`，  
尚未分配时，子元素宽度为 `0 0 200px`，空间有余，再按 `1, 1, 1` 的 `flex-grow` 分配。

显然，flex-basic 会更与分配前的宽度计算更相关，比如 max-width 那些。



## 文件
https://mp.weixin.qq.com/s/WtGzVMzh1RupixD_4474mg  
