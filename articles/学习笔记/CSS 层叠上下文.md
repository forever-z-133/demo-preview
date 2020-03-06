# CSS 层叠上下文

在使用 bootstrap 的模态框时，出现了半透明遮罩始终盖住模态框的情况，<br >
只有把模态框的 HTML 写到 body 之下才恢复正常，<br >
预览：https://codepen.io/foreverZ133/pen/GbMKXd

后来有幸看到了张鑫旭在慕课网 CSS深入理解之 relative 的视频，<br >
才算真正知晓了层叠上下文在 CSS 中的规则和运用技巧。


## 什么是层叠上下文和层叠水平

所有的上下文都代表着存储着某些关联信息，比如执行上下文、块级格式化上下文等等。<br />
当进行定位布局的渲染时，如何标记这堆元素的覆盖关系呢，在指定点击事件时是怎么知道会先碰到最上层的元素的呢？<br />
层叠上下文其实把它理解为 z 轴也没有问题，本文重点研究层叠上下文的表现与应用，暂不探究其原理。<br />
https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context

层叠水平，则是在创建了层叠上下文的情况下，z 轴层级上的表现。

## 形成层叠上下文

通常情况下，兄弟节点总是后者覆盖前者的。<br >
_注意：仅讨论 display: bock 的情况，其他 display 会有另一套规则_

```
body
  - red【普通元素】
  - green【普通元素】
```

<iframe height="265" style="width: 100%;" scrolling="no" title="通常情况" src="https://codepen.io/foreverZ133/embed/WNvOyym?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/WNvOyym'>通常情况</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

但当某元素被创建层叠上下文后，它在 z 轴上会比普通元素高上一层而能覆盖普通元素。<br >
_注：z-index 属性有些特殊，比如：浏览器不区分 z-index: auto 或 z-index 负值只有定位时有效，等_

添加以下属性即可创建层叠上下文 _（随着 CSS3 属性还在增加，本表不全）_ ：

- z-index 不为 auto 的 relative（此条有争议，比如 chrome79 没管 z-index）
- absolute, fixed
- z-index 不为 auto 的 flex 子项
- opacity 不为 1
- transform 不为 none
- perspective 不为 none
- filter 不为 none
- mix-blend-mode 不为 normal
- 带有 isolation: isolate
- will-change 不为 none
- 带有 -webkit-overflow-scrolling: touch

```
body
  - red【含层叠上下文】
  - green【普通元素】
```

<iframe height="265" style="width: 100%;" scrolling="no" title="含层叠上下文" src="https://codepen.io/foreverZ133/embed/dyoRKgV?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/dyoRKgV'>含层叠上下文</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 同级层叠水平后者居上

当存在处于同个 z-index 的层叠上下文时，即同级层叠水平，<br >
那么会表现为 **后者居上**。 _（注意：父级祖父级不能含有层叠上下文）_

```
body
  - gray【普通元素】
    - red【含层叠上下文，z-index: auto】
  - pink【普通元素】
    - green【含层叠上下文，z-index: auto】
```

<iframe height="295" style="width: 100%;" scrolling="no" title="同级层叠水平" src="https://codepen.io/foreverZ133/embed/XWbgYLj?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/XWbgYLj'>同级层叠水平</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 不同级层叠水平大者居上

_注：z-index 好像不同浏览器版本有差异，如需 z-index 生效最好加上定位，如 relative。_

```
body
  - gray【普通元素】
    - red【含层叠上下文，z-index: 1】
  - pink【普通元素】
    - green【含层叠上下文，z-index: auto】
```

<iframe height="295" style="width: 100%;" scrolling="no" title="不同级层叠水平" src="https://codepen.io/foreverZ133/embed/zYGdgOP?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/zYGdgOP'>不同级层叠水平</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 层叠水平的嵌套

当父级或祖父级元素含有层叠上下文时，内部的层叠上下文类似于水平的水平这样的嵌套关系。<br>
这意味着，当父级层叠水平不够高时，子级的层叠水平再高也没用。

```
body
  - gray【含层叠上下文，z-index: auto】
    - red【含层叠上下文，z-index: 999】
  - pink【含层叠上下文，z-index: auto】
    - green【含层叠上下文，z-index: 1】
```

<iframe height="295" style="width: 100%;" scrolling="no" title="层叠水平的嵌套" src="https://codepen.io/foreverZ133/embed/ZEGXdJY?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/ZEGXdJY'>层叠水平的嵌套</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 层叠水平内部覆盖规则改变

起初我并不能理解下图是何含义，但当我发现 z-index: -1 在父级是否含有层叠上下文时的不同表现时才意识到，<br>
层叠水平内部元素的覆盖关系，可能与普通元素的覆盖关系有所不同。

![](https://cdn.nlark.com/yuque/0/2019/png/158440/1561019816571-2406a408-f6c6-4063-8464-b27f2ea40bd7.png#align=left&display=inline&height=375&originHeight=375&originWidth=599&size=0&status=done&width=599)

如果父级不含层叠上下文，

```
body
  - gray【普通元素】
    - red【普通元素】
    - green【含层叠上下文，z-index: -1】
```

<iframe height="265" style="width: 100%;" scrolling="no" title="父级不含层叠上下文的负值水平" src="https://codepen.io/foreverZ133/embed/dyoVBJV?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/dyoVBJV'>父级不含层叠上下文的负值水平</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

```
body
  - gray【含层叠上下文，z-index: auto】
    - red【普通元素】
    - green【含层叠上下文，z-index: -1】
```

<iframe height="265" style="width: 100%;" scrolling="no" title="父级含层叠上下文的负值水平" src="https://codepen.io/foreverZ133/embed/XWbeLZd?height=265&theme-id=dark&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/XWbeLZd'>父级含层叠上下文的负值水平</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 总结

以上便是创建层叠上下文后与众不同的视觉表现的规律。

可以在遇到奇怪的覆盖样式问题时有一定理论思路：https://codepen.io/foreverZ133/pen/GbMKXd

也可利用水平嵌套时 z-index: -1 的表现制作更多优秀效果：https://codepen.io/foreverZ133/pen/EBoWBd

还可以利用层叠上下文高于普通元素的表现区别于普通元素的覆盖问题：https://codepen.io/foreverZ133/pen/jjYOyo

更多案例：<br> 
https://codepen.io/foreverZ133/pen/orpZKR   
https://codepen.io/hexagoncircle/pen/JZNVdE
