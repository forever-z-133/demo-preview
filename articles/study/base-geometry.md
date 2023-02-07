# 前端开发应该掌握的计算几何基础

> ### 点 [x, y] 绕点 [ox, oy] 旋转 angle 弧度

矩阵公式：[x, y, 1] x transform1 x transform2 x transform3

```
transform1     transform2                    transform3
[1   0   0]    [cos(angle)  sin(angle) 0]    [1  0  0]
[0   1   0]    [-sin(angle) cos(angle) 0]    [0  1  0]
[-ox -oy 1]    [0           0          0]    [ox oy 1]
```

```
[
  ((x - ox) * Math.cos(angle)) - (y -oy) * Math.sin(angle) + ox,
  ((x - ox)* Math.sin(angle)) + (y - oy) * Math.cos(angle) + oy
]
```

> ### 点 [x, y] 延 angle 弧度方向偏移 d 距离

矩阵公式：[x, y, 1] x transform1 x transform2

```
transform1   transform2
[1 0 0]      [cos(angle)  sin(angle) 0]
[0 1 0]      [-sin(angle) cos(angle) 0]
[d 0 1]      [0           0          1]
```

```
[x + d * Math.cos(angle), y + d * Math.sin(angle)]
```

> ### [[x1, y1], [x2, y2]] 线段长度

```
Math.sqrt(Math.pow(x2- x1, 2) + Math.pow(y2 - y1, 2))
```

> ### [[x1, y1], [x2, y2]] 线段角度

```
Math.atan2(y2 - y1, x2 - x1)
```

> ### 线段 [point1, point2] 绕点 origin 旋转 angle 的新线段

```
[pointRotate(point1, origin, angle), pointRotate(point2, origin, angle)]
```

> ### 线段 [point1, point2] 延 angle 偏移 d 距离生成的新线段

```
[pointTranslate(point1, d, angle), pointTranslate(point2, d, angle)]
```

> ### 多边形 polygon 面积

多边形面积可以记为每条边和原点形成的三角形面积之和 (x1 * y2 - x2 * y1) / 2，逆时针记为正，顺时针记为负

```js
function polygonArea(points){
  let area = 0;
  for (let i = 0, l = points.length; i < l; i++) {
    const [x1, y1] = points[i];
    const [x2, y2] = points[i === l - 1 ? 0 : i + 1];
    area +=x1 * y2;
    area -=  x2* y1;
  }
  return Math.abs(area / 2);
}
```

> ### 多边形 polygon 周长

上文已介绍线段长公示，此处累加每边长即可

> ### 多边形 polygon 旋转、偏移

上文已介绍点的旋转偏移公示，对多边形所有顶点应用即可

> ### 点 [x, y] 是否在线段 [[mx, my], [nx, ny]] 左侧

按照向量外积定义，线 [[mx, my], [nx, ny]] 与 线 [[mx, my], [x, y]] 夹角 θ > 0 时，点在线左。在线右测同理

```
sinθ = (nx - mx) x (y - mx) - (ny - my) x (x - mx)
```

> ### [[ax, ay], [bx, by]] 与 [[mx, my], [nx, ny]] 两线段相交

按照向量外积定义，线 [[ax, ay], [bx, by]] 与线 [[mx, my], [nx, ny]] 夹角 θ = 0 时，两线平行或重合。

```
sinθ = (ny - my) x (bx - ax) - (nx - mx) x (by - ay)
```

再算 [ax, ay] [mx, my] [bx, by] 三点构成的三角形面积 area1 与 [ax, ay] [mx, my] [nx, ny] 构成的 area2，交点分线段比例为 num1 = area2 / sinθ 和 num2 = area1 / sinθ，若 num1 和 num2 均处于 [0, 1] 区间则线段相交

```
area1 = (bx - ax) x (ay - my) - (by - ay) x (ax - mx)
area2 = (nx - mx) x (ay - my) - (ny - my) x (ax - mx)
```

> ### 点 [x, y] 在多边形 polygon 内部

https://forever-z.cn/#/articles/study/justify-click-inner
