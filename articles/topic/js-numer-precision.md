# JS 小数的精度问题的总结

经典问题 `0.1 + 0.2` 不等于 `0.3`，都说是精度问题，但这个问题可以再深入一点。  

可以从 存储、运算、显示 三个方面来看。

## 存储

对于计算机，存储下来肯定是 `0` 和 `1`，所以我们可以靠 `.toString(2)` 来进行一个初体验。

```js
0.2.toString(2) // 0.001100110011001100110011001100110011001100110011001101

0.25.toString(2) // 0.01
```

上面的例子能看出，`0.2` 存储时成了循环小数，而 `0.25` 则没有。  
而循环小数不可能一直循环嘛，所以就会存在一定的截断，因此有了精度问题。

以上为二进制的表现，官方则提供了 `toPrecision` 这个方法供我们了解十进度下的精度表现，更方便理解。  
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/toPrecision

```js
0.2.toPrecision(32) // '0.20000000000000001110223024625157'

0.3.toPrecision(32) // '0.29999999999999998889776975374843'
```

上面的例子能看出，`0.2` 在十进制中的存储结果其实比 `0.2` 大一丢丢，而 `0.3` 会小。

更为具体的细节，则可去了解 IEEE-754 规范，但有精确到位数的描述。  
http://www.softelectro.ru/ieee754_en.html

## 运算

```js
0.1.toPrecision(32) // '0.10000000000000000555111512312578'
0.2.toPrecision(32) // '0.20000000000000001110223024625157'
0.1 + 0.2 // 0.30000000000000004

0.2.toPrecision(32) // '0.20000000000000001110223024625157'
0.3.toPrecision(32) // '0.29999999999999998889776975374843'
0.2 + 0.3 // 0.5
```

由此，才可知为何 `0.1 + 0.2` 会比 `0.3` 大，  
因为 `0.1` 和 `0.2` 的存储结果都比预期的大上一点，所以结果也就大一点。  
而 `0.2` 与  `0.3` 一个大一个小刚好抵消，所以是符合预期的结果。

至于为什么精确的 `0.5` 加上不精确的 `0.3` 结果为精确，那就是位数的问题了

另外，同理，当你使用 `toFixed` 等官方函数时，也是有类似的精度问题。

```js
2.55.toPrecision(32) // '2.5499999999999998223643160599750'
2.55.toFixed(1) // '2.5'

1.55.toPrecision(32) // '1.5500000000000000444089209850063'
1.55.toFixed(1) // '1.6'

1.45.toPrecision(32) // '1.4499999999999999555910790149937'
1.45.toFixed(1) // '1.4'
```

有个题外小故事，为了保持 `1234` 与 `56789` 在四舍五入时概率配平，  
网传 `toFixed` 使用的 “银行家算法” 来均匀地分配，但其实按 ECMA-262 标准文档来看并不是哈。  
https://tc39.es/ecma262/#sec-number.prototype.tofixed

## 显示

```js
2.5499999999999998223643160599750 // 2.55
```

当你在开发者空间的 `Console` 去打印时，你会发现它帮忙去掉了精度误差。

## 避免方案

粗劣的办法，就是将小数转为字符串，以整数的形式去运算再变回小数。

但上述方案会遇到整数结果过大而超出安全数范围的问题，  
那就只能靠一位一位来处理的方式了，也即 `decimal.js` 等库的实现方式。  
https://www.npmjs.com/package/decimal.js
