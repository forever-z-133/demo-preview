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

## min/max-width

## bfc

## transform

## flex-grow/shrink
https://mp.weixin.qq.com/s/WtGzVMzh1RupixD_4474mg

## flex-basic
