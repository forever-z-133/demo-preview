# 如何判断是否点击在元素内部

在使用 html 或 svg 时给节点绑上点击事件，在点中时会轻松触发回调，<br />
但在使用 canvas 时，这种点中效果就需要我们自己来计算了。<br />

```js
/// 普通方块的点击判断
function containPoint(rect, point) {
  return (point.x >= rect.getMinX() && point.x <= rect.getMaxX() && point.y >= rect.getMinY() && point.y <= rect.getMaxY()) ;
}
```

普通情况下都是 `Rect` 的，但遇上旋转或多边形时，就稍稍复杂一丢丢咯。<br />

## 像素法

主要依赖于 `getImageData` 中 `rgba` 的 `a`，为 0 则不管，为 1 则可能属于该元素，再判断下位置即可。<br />

## 交点法

在鼠标位置画一条直线，若与相邻两点连成线段的交点数量在同一侧为奇数（任一侧），则在元素内。

<img src="https://s3.ax1x.com/2020/11/25/DaMEzn.jpg" width="300" alt="交点法">

该方案也存在着几款变种，比如以 y 轴画直线，或者，两端交点数量一致也能表示。<br />
其他案例：https://test.forever-z.cn/single/click-inside.html

## 角度法

若鼠标位置与每个顶点的构成的夹角，相加刚好等于 360°，则在元素内。

## 总结

显然，几种方案都各有优劣，比如像素法能判圆环和弧形比较简单，角度法最好理解，交点法判线条相交的图形比较稳妥，需根据场景取舍。<br />
