# 移动端视频播放方面的坑

## 封面图的显示问题

众所周知，`<video>` 标签有着 `poster` 属性可设置视频的封面图，在 PC 上表现不错，但在移动端就很不尽如意了。

![](https://s1.ax1x.com/2020/03/23/8HEvGT.png)

如上图所示，部分视频以及部分手机即使设置了 `poster` 也并未生效，视频暂停时比例失衡的封面图也不美妙。<br />
在兼容 `poster` 时还会存在一些问题，比如部分浏览器的 `<video>` 是始终高于 dom 元素的。

因此，直接显示 `<img>` 为封面图，隐藏 `<video>` 直到封面图被点击，这个方案要禁受得住需求。

```scss
.video-box {
  position: relative;
  /* 至于，该容器是 定宽定高 还是 自适应等比例，随君喜欢了 */
  video { position: absolute; left: -9999em; }
  img { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: contain; }
  .btn-play {}
  .timestamp {}
}
```

## 封面图的产生问题

可能你还会面临个需求，封面图我不要特定的，我就要视频帧封面 _（坑逼玩意儿）_。

对于不跨域的视频来说，用 canvas 截图可以获取是可行的。

```js
$video.addEventListener('loadeddata', () => {
  createVideoPoster(this)
});
function createVideoPoster($video) {
  var oldCanvas = $video.parentNode.appendChild(canvas);
  oldCanvas && oldCanvas.remove();
  $video.setAttribute('crossOrigin', 'Anonymous');
  var canvas = document.createElement('canvas');
  var winW = canvas.width = $video.parentNode.offsetWidth;
  var winH = canvas.height = $video.parentNode.offsetHeight;
  var w = $video.clientWidth;
  var h = $video.clientHeight;
  if (winW / winH > w / h) {
    hh = winH; ww = w / h * winH;
  } else {
    ww = winW; hh = h / w * winW;
  }
  canvas.getContext('2d').drawImage($video, winW/2-ww/2, winH/2-hh/2, ww, hh);
  $video.parentNode.appendChild(canvas);
}
```

如果你使用的是阿里云存储之类的文件管理方案的话，是有取帧画面的功能的，只需要在视频链接中加上部分字段即可。<br />
文档：https://help.aliyun.com/document_detail/64555.html?spm=a2c4g.11186623.6.1031.BuoKRf<br />
DEMO：https://oss-uat.xuebangsoft.net/13bef702-20f7-438c-97fb-8bacbaba18eb.mp4?x-oss-process=video/snapshot,t_500

```
?x-oss-process=video/snapshot,t_500
```

如果还不被满意，就只能通过后端或产品来协作了，<br />
比如通过 ffmpeg 之类的工具进行截取，或者在视频上传时要求用户自己上传封面。

## 播放时自动全屏问题

在大多数移动端浏览器中，视频播放时是不会自动全屏的，但微信浏览器会。<br />
另外，不全屏的视频组件也可能会始终高于 dom 元素，因此虽然有多个方案却也需要一定的取舍。

一种是保持视频原尺寸以弹窗形式播放；<br />

```scss
.video-play-modal {
  position: absolute; /* 移动端特别不推荐用 fixed，还请设法为 absolute 进行处理 */
  top: 0; left: 0; right: 0; bottom: 0;
  video {
    position: absolute;
    top: 50%; right: 50%;
    width: 100%; height: auto;
    transform: translate(-50%, -50%);
  }
  .btn-close {}
}
```

还有的是使用 `requestFullscreen` 来让播放时类似于全屏。

```js
document.querySelector('.xxx video').requestFullscreen();
```

在微信浏览器，也即 X5 浏览器中，可以通过特有属性来直接全屏。

```html
<video src="" x5-video-player-type="h5" x5-video-player-fullscreen webkit-playsinline playsinline />>
```

另外，后者与前两者有个较大的不同，就是微信浏览器在这种模式下视频上就可以覆盖元素了。<br />
所以假如你的需求是可点可滑动的互动视频之类的，那恐怕只有微信上能较容易实现，且播放时只能是全屏的视频。<br />
如果只是播放普通视频，个人更推荐第一种，可以节约 video 元素，且不会为切换视频时的各种暂停消音等问题烦扰。

## 视频播放完毕有广告问题

暂时还只发现当未设置 x5 属性时微信浏览器有这个问题，且不会触发 `onended` 回调。<br />
没有回调就让我们很难搞，暂时我也没想到很好的办法。

而如果加上 x5 属性又会使视频播放时全屏，因此，这个问题只能与产品进行沟通妥协。

## 显示视频时长问题

理论上，在有 `preload` 属性时，或 `oncanplaythrough` 回调时，可以获得 `$video.duration` 的时长值。

但各浏览器差异很大，有时就算 play 起来了也获取不到，<br />
所以想在封面图时显示视频时长恐怕只能找后端处理了，移动端对视频的规范真的不友好。

## 退出全屏的回调问题

比如微信浏览器带 x5 属性的视频在关闭全屏的视频时，<br />
没触发 `onfullscreenchange` 回调，用 `document.fullscreen` 和 `document.fullscreenElement` 也无效。

这个问题我也还没有有效的方案，只有一个十分破坏交互的策略。<br />
在关闭全屏时至少倒是有个 `onpause` 回调，如果用这个的话就分不清是真暂停还是退出全屏了。

## 部分视频无法播放或无声音问题

多半是视频编码问题，但前端手段有限，不实际播放无法得知好坏，上传视频的用户也可能不知道，用阿里云也是偶现，暂时无解。

----

以上即是我对移动端视频播放的踩坑记录，在 PC 上应该都没啥问题，移动端兼容任重道远呀。
