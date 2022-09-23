# 字体文件的引用与压缩

在最新项目中，由于要频繁使用艺术字，<br />
而用户设备没有此字体，所以以往的都是使用图片的。

在同事的瞩目期许之下，我便开始实验研究其他的解决方案

## 前言

CSS3 的 `@font-face` 超屌的，使用也方便，兼容性如今也完全没问题。

```
@font-face {
  font-family: 'xxxx';
  src: url('./font/汉仪秀英体简.TTF');
}
.font {
  font-family: 'xxxx', Arial, sans-serif;
}
```

但是随着项目发布，还是出现了问题，<br />
由于字体文件比较大(3.8M)，文字会经历先不显示再显示默认字体再变为艺术字的过程，视觉效果不太美妙。<br />
这可能是浏览器对字体文件的加载策略吧。咱们便来视图解决这个需求。

## 干掉显示默认字体这个过程

### 字体加载后才反应

在探究思路时，设计师表示如何没有显示默认字体这个过程，直接是空白文字然后变为艺术字也是不错的。

那么我只需先让文字全部消失，比如 `text-indent:-999em`；<br />
然后在知道字体加载完成后修改这个状态类就可以了。

使用 onload，但 fontLoaded 需要是挂在 window 上的函数。

```html
<link href="fonts.css" onload="fontLoaded" rel="stylesheet" />
```

使用 FontFace API：https://developer.mozilla.org/zh-CN/docs/Web/API/FontFace

```js
new FontFace('fontFamily', `url(font.woff2) format('woff2')`).load().then(fontLoaded);
// 不用标准用开源项目也行 https://github.com/bramstein/fontfaceobserver
```

使用 font-display：https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face/font-display<br />
这个 CSS 属性也是来解决这个问题，但在移动端兼容性上也不太美妙。<br />


```css
body {
  font-display：optional; /* 为字体提供一个非常小的阻塞周期并且没有交换周期 */
}
```

## 优化加载速度

假如字体加载得足够快，那也是可以避免上述问题的咯。

一方面是单独搞个存储空间来干这事，走 CDN 等等方式来优化网路传输。

另一方面则是使用 preload 和缓存等操作来提升下次加载的效果。

```html
<link rel="preload" as="font" href="font.woff" crossorigin />
```

当然，这肯定也不算是能真正解决问题的方案。

## 压缩字体文件大小

其实当字体文件大小并不大时，比如 300-500k 左右，并不会有明显的视觉问题，<br />
所以直接减少字体文件的体积也是种不错的办法。

但市面上有个许多种不同的方案，在此罗列一下：

### 挑选文字后打包

比如有名的 [iconFont](https://www.iconfont.cn/#!/webfont/index) 或 java 版的 [FontZip](https://github.com/forJrking/FontZip)；<br />
它们都是确定或上传字体后，选择部分文字来打包，以此来减少体积的。<br />
市面上应该还有很多类似的工具，比如 [fontmin](http://github.com/ecomfe/fontmin) 等等，欢迎补充。

### 自动根据 dom 打包

这时可以分为线下版和线上版两种。

线下版的代表有 [Font Spider](http://font-spider.org/)。<br />
根据现有 html 提取其 dom 中的文字，然后把他们打包，非常自动化。

```cmd
npm install font-spider -g

font-spider ./demo/*.html
```

线上版的代表有 [有字库](https://www.webfont.com/)。<br />
特别适用于异步生成的文字，比如接口请求后的列表等，<br />
调用 draw 后根据类名将其 dom 中的文字重新打包。

但有字库比较商业化，有数量限制且需设置域名白名单，常用的话需要充钱的。<br />
DEMO: https://forever-z-133.github.io/demos/single/FontFamily.html

```js
// <script src="https://cdn.webfont.youziku.com/wwwroot/js/wf/youziku.api.min.js"></script>
$youziku.load('.text2', '492565fa765c4ad381d6bdfba2fc918b', 'minijianyaya');
$youziku.draw();
```

## 字体分包来优化效果

实验时突然的灵光一现，让我发现了一个绝妙的方案。

当一个字体文件包含“牛”字，一个字体文件包含“逼”字，那同时引用两个字体文件会怎样呢？

```css
@font-face {
  font-family: '站酷高端黑体-1';
  src: url(./font/niu2.ttf);
}
@font-face {
  font-family: '站酷高端黑体-2';
  src: url(./font/bi2.ttf);
}
.text1 {
  font-family: '站酷高端黑体-1';
}
.text2 {
  font-family: '站酷高端黑体-2';
}
.text3 {
  font-family: '站酷高端黑体-1', '站酷高端黑体-2';
}
```

![字体分包图示](https://i.loli.net/2020/03/01/GCrFwmt8oO47VSz.png)

浏览器的加载与字体匹配原则并不太清楚，为什么会出现这种同一个 dom 中能渲染两种字体的效果，<br />
但可以知道的是 font-family 是有执行顺序的，合理分包和排序后，可以既快速显示又兼顾全字体下载。

可见，把 3M 大小的字体拆为常用文字 500k 和剩下的 2.5M，甚至更多份，也是个可行的方案。<br />
另一方面，这种原生标准的方案也更容易实现缓存和优化加载等效果。

6000 个常用汉字：https://github.com/forever-z-133/others/issues/100

## 其他问题

### 字体链接顺序

比如 woff2 比 woff 的兼容支持要更好，woff2 要比 ttf 的内存要更小，<br />
所以实践下来，官方案例中的顺序其实并非最佳的，以下顺序才是：

```css
@font-face {
  font-family: 'webfont';
  src: url('webfont.woff2') format('woff2'),
       url('webfont.woff') format('woff'),
       url('webfont.ttf') format('truetype');
  font-style: normal;
  font-weight: normal;
}
```

### 字体格式转换

往往在网上下载的字体并不包含所有格式，如果只有 ttf 就会不兼容 IE，因此需进行字体格式转换。

推荐网站：<br />
https://www.fontke.com/tool/convfont/<br />
https://convertio.co/zh/

### 压缩失败问题

https://www.w3cplus.com/css/comprehensive-webfonts.html<br />
https://mp.weixin.qq.com/s/uOadALqR6-MpSrV2WYZHPw<br />
根据上文描述，字体 `表(table)` 之间是有关联的，有的压缩并不能很好的处理此过程。<br />
在实践中就遇到了好几种 OTS parsing error 的报错：

```
1) Failed to parse metrics in vhea
2) cmap: Failed to parse format 4 cmap subtable 0
3) invalid version tag
```

像此类问题除了尝试更多的压缩工具外，恐怕只能更换相似字体再尝试或其他技术实现的办法了。

这也是影响开发流程的操作，可与团队商榷考虑将此流程卡点加入流程管理中，<br />
比如当项目需要用到艺术字时，先请前端尝试压缩能否成功再进行后续设计和开发。

### 小程序环境

小程序的 wxss 样式中只允许远程链接，但各公司不见得有资源服务器，<br />
所以可以将字体文件转为 base64 这种方式来实现本地引用。

推荐网站：https://transfonter.org/

### 免费字体

前公司其实并没有设置 font-family 结果也被微软黑体给律师函警告了，<br />
但公司体量达到一定程度后，难免会遇到这类行为，所以必然要整站切换字体。

那么当然就需要寻找这些免费商用字体啦：<br />
http://www.fonts.net.cn/album/free-chinese-fonts-1.html<br />
http://font.chinaz.com/tag_font/MianFei.html

更多人也喜欢用 https://fonts.google.com/ 这个谷歌 404 服务，<br />
可以使用 https://cdn.baomitu.com/index/fonts 来寻求代理的帮助。
