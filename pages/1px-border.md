# 四种 1px 毛细线样式

毛细线样式，即研究 1px 问题，  
即是处理高 dpr 设备上 1px 的 border 看上去像 2px 一样粗。


``` 一、@media
```css
.border-bottom { border: 1px solid #999 }
@media screen and (min-device-pixel-ratio: 2) {
    .border { border-bottom: 0.5px solid #999 }
}
@media screen and (min-device-pixel-ratio: 3) {
    .border { border-bottom: 0.333333px solid #999 }
}
```