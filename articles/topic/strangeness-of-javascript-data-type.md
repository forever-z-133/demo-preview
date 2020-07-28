# JS 数据类型方面的蹊跷

现在去做前端面试题都是心虚的，
本来可以做对的题，想想好像有坑，然后答错了。举个例子：

```js
Number([0]); // 0
[0] == true; // false
if ([0]) alert('ok'); // "ok" // 恩? 不都是 false 吗
```

所以本文将尽可能多的去试图挖一下 javascript 数据类型方面的蹊跷。

- [数据相等](#)
  - [判等于](#)
  - [判大小](#)
  - [null 和 undefined](#)
  - [-0 问题](#)
- [数据类型判断](#)
  - [if 判断](#)
  - [typeof 判断](#)
  - [instanceof 判断](#)
  - [constructor 判断](#)
  - [is 方法判断](#)（如 isNaN()）
  - [其他判断](#)
    - [Object.is() 方法](#)
    - [key in obj 判断](#)
    - [prototype 判断](#)
- [强制数据类型转换](#)
  - [运算时自动转换](#)（如 +'6'）
  - [对象式转换](#)（如 Number([1])）
  - [函数式转换](#)
    - [原型函数](#)（如 toString()）
    - [parse 系列函数](#)（如 parseInt([1])）
    - [参数的隐形转换](#)
    - [其他强行转换转换](#)（如 JSON.stringify()）

## 数据相等

### 判等于

```js
0 == '0'; // true
0 == []; // true
'0' == []; // false
```

以上现象都被人做成了表情包在网络上流传，<br />
其实是因为它的转换是按以下过程依次进行的，让两边变为同类型后再对比。<br />
_此结论为笔者的猜测，暂未找到相应文档支持，嘤嘤嘤。_

```js
Number < String < Array
Number < Boolean
```

这里有几个特别注意点：

a. 两边是依次转换的，而不是一次性搞成同种类型的。

```js
Boolean([0]) // true
[0] == true;  // false
// 实为 '0' == 1 再 0 == 1 的判断
```

b. 转为相同类型即停止转换并开始比较

```js
['0'] == ''; // false
// 实为转到 '0' == '' 便停止了转换
// 这也是 '0' == [] 为 false 的原因
```

c. 转到的最终是 Number 的转换
其实认为 Boolean < Number 这样转换也是阔以的，但最终转为 Number 要更容易铭记和解释。<br />
比如下章的 `2 > true` 为 true 并不是 `true > true` 为 false 这样的结果。

```js
'0' == true;  // false
// 实为 0 == 1 或 false == true 都是没问题的，但后者不容易解释。
```

而全等 `===` 就不会进行这样的过程。

### 判大小

#### 普遍情况类似

而 `<=` 这类数值判断，也是上章的转换流程。

#### 字母字符串判大小稍有不同

此处要多提一下字符串比大小时的不同，<br>
它会依次用两个字符的 `codePointAt` 比较。

```js
'a' > 'A'; // true，实则为 97 > 65
'-A' > '+A'; // true，实则为 45 > 43
'-1' > ''; // true，实则 45 > undefined
```

而字母字符串与数字的比较并非是 codePointAt 转换而是 Number 转换

```js
'a' > 1; // false，实为 NaN > 1 而非 97 > 1
```

### null 和 undefined 比较

非常诡异，暂未总结出明显的规律。

```js
null > 0;     // false
null == 0;    // false
null >= 0;    // true

undefined == undefined;  // true
undefined <= undefined;  // false
```

### -0 问题

此处稍微提一下 `-0` 的存在，会造成 `Infinity` 与 `-Infinity` 的不同。
但我们多半会做分母不为零的判断的，恩大概会的吧。

```js
0 === -0; // true
1 / -0 === 1 / 0; // false
```

## 数据类型判断

### if 判断

包括 `if` `?:` `||` `&&` `!` 这堆做表达式表达式的。<br />
而实践中大致为出现以下可能，除了最后一个应该都很方便理解，实为运行结果再 Boolean 的对比。

```js
if (a <= b)
if (!a)
if (a)
if (a())
if (a = 1)
```

![image](https://s2.ax1x.com/2020/02/11/1oibgf.png)

注意 if 判断与等于判断的不同哟，前者转到是 Boolean 为目标，后者是 Number 为目标。

```js
if ([]) console.log(x); // true，[] 实为 Boolean([]) 为 true
[] == false; // true，[] 实为 Number(String([])) 为 0
```

还需注意的是容易惯性思维的编码习惯，在使用时需要规避，<br >
比如 `0` 和 `NaN` 的区分，`''` 和 `[]` 的区分等等。

```js
x.split(','); // 应改为 x ? x.split() : []

x || 1; // 应改为 var xx = parseFloat(x); isNaN(xx) ? 1 : xx;
```

### typeof 判断

```js
typeof [] === 'object';
typeof NaN === 'number';
typeof null === 'object';
```

可见这个判断其实还有众多情况无法得到区分。

### instanceof 判断

用 `[] instanceof Array` 判数组真的很方便，但这块也还是有坑的。

```js
'a' instanceof String; // false
new String('a') instanceof String; // true
```

除此之外，还有原型链上的一点问题：

```js
function Foo() {}
var foo = new Foo();
console.log(foo instanceof Foo); // true

Foo.prototype = new Aoo();
var foo2 = new Foo();
console.log(foo2 instanceof Foo); // true
console.log(foo2 instanceof Aoo); // true
```

说实话，除了部分应用场景，用这个来判原型其实并不是很好的方法。
参考：https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/

### constructor 判断

`constructor` 相比 `instanceof` 有一点优势，就是它不随 `__proto__` 的改变

```js
function A() {}
var a = new A();
var b = new A();
a.__proto__ = {};

a instanceof A; // false
b.constructor === A; // true
```

以往 es5 做继承时还要自己给 `A.prototype.constructor` 设新值，
有了 es6 的 `class` 后已经很简单了，用 `constructor` 来判原型也稳妥了起来。
至于基础数据类型嘛，也不太推荐用此方法。

### is 方法判断

```js
isFinite();
isNaN();
Number.isNaN();
Array.isArray();
```

### 其他判断

#### Object.is() 判断

其实 Object.is() 是类似 === 的，但又有点不一样，它是真真正正的绝对相等。

```js
+0 === -0; // true
Object.is(+0, -0); // false

NaN === NaN; // false
Object.is(NaN, NaN); // true
```

#### key in object 判断

这个判断需稍微分清一下原型与实例。

```js
'0' in [1, 2]; // true
'now' in Date; // true
'getFullYear' in Date; // false
```

至于项目是使用以下哪种判断就见仁见智了。

```js
if (Array.prototype.includes) {}
'includes' in [];
```

#### prototype 判断

`obj.hasOwnProperty(key)` 和 `obj.isPrototypeOf(obj2)` 等相关方法，整理中

## 强制数据类型转换

### 运算式自动转换

```js
+' 014';  // 14
+ '0x12'; // 18

1 + '14'; // '114'
1 + '0x12'; // '10x12'
1 + +'14'; // 15
'14' + 1; // '141'

1 + [1, 1]; // '11,1'
1 + {}; // '1[object Object]'

1 + null; // 1
1 + undefined; // NaN
```

很鲜明，当有单独的运算符存在时（单加括号是不行滴），<br>
会帮忙 `Number` 转换，否则 `String` 转换。<br>
还请注意上例中后 4 种特殊的情况。

进行 `++` 运算时并不会帮忙转换为数字，还容易报错。<br>
所以使用时这里得留个心眼哟。

```js
++'14'; // ReferenceError
```

还有两个特立独行的数字运算，即 `Infinity` 和 `0` 的正负号。

```js
Infinity + Infinity; // Infinity
-Infinity + -Infinity; // -Infinity
Infinity + -Infinity; // NaN

+0 + +0; // 0
-0 + -0; // -0
+0 + -0; // 0
```

再看一个特例娱乐下（基本上是遇不到的情况），
`{} + []` 理论上应该是 `'[object Object]' + ''` 才对，就算不是也应该是 `NaN + 0` 吧，结果我又猜错了。
遇事不决问百度，`{}` 在前被当成空的代码块了，实为 `+[]` 自然就是 `0` 了。

```js
[] + {}; // '[object Object]'
{} + []; // 0
```

### 对象式转换

#### Number() 转换

`Number` 与 `parseInt` 的不同，将于下文 [parseInt 系列方法](#) 讲述

#### String() 转换

探讨一下 `String` 和 `toString` 的不同吧。

#### 有些 toString 会报错

```js
String(null); // 'null'
null.toString(); // Uncaught TypeError
undefined.toString(); // Uncaught TypeError
```

#### 数字的 toString 会有进制转换

```js
(30).toString(16); // "1e"
'30'.toString(16); // "30"
```

#### 其他

至于 `Date` `Error` `RegRxp` 的字符串化，基本不会出啥幺蛾子。

用原型的 `toString` 来判数据类型也是种很巧妙常用的方法。

```js
function typeOf(obj) {
  var typeStr = Object.prototype.toString.call(obj).split(' ')[1];
  return typeStr.substr(0, typeStr.length - 1).toLowerCase();
}
typeOf([]); // 'array'
```

### 函数式转换

#### 原型方法

`toString` 在上文已有介绍，但还得再区分一下数组的。

```js
[1, [2, 'abc', '', 0, null, undefined, false, NaN], 3].toString();
// "1,2,abc,,0,,,false,NaN,3"
```

也即是下例想表达的意思：

```js
null.toString(); // Uncaught TypeError
[null].toString(); // ''
```

`toString` 与 `valueOf` 大致是相同的，但是否有不同，整理中...

再则 `(1).toFixed` `Date.parse` 等，应该不会有啥常见错误。
只需注意那些是会 **对入参进行隐形转换** 的，下文 [参数的隐形转换](#) 将介绍

#### parseInt 系列方法

主要来看 `Number` 与 `parseInt` 的不同，挺迷的，
它们并不是单纯的数据类型转化那么简单，举个例子：

```js
Number(''); // 0
parseInt(''); // NaN
```

`parseInt` 就很花哨，还会再多进行一些暗箱操作来判断和适配成数字。
可见，用 `Number` 转非整数时会是更好的选择。

```js
parseInt(' 10 '); // 10  // 自动去空格，通用
parseInt('10.2'); // 10  // 数字后的全剔除，Number 和 parseFloat 没问题
parseInt('1e2'); // 1   // 区分不出科学计数法，Number 和 parseFloat 没问题
parseFloat('0x5'); // 0   // 区分不出进制，Number 和 parseInt 没问题
```

当参数为数组时，当然也是先转 `String` 的咯，
而 `parseInt` 又能去除 `,` 后的字符，所以就有下面的情况。

```js
Number([1, 2]); // NaN
parseInt([1, 2]); // 1
```

还有一点小知识需注意：

```js
parseInt === Number.parseInt   // true
isNaN === Number.isNaN;        // false
```

#### 参数的隐形转换

比较典型的 `isNaN` 是先用 `Number` 转了一次，但 `Number.isNaN` 就没有。

```js
isNaN('1x'); // true
Number.isNaN('1x'); // false
```

这方面没做什么整理，遇到了再补吧。

```js
'12'.replace(1, ''); // "2"
Math.max(0, '1.'); // '1'
```

#### 其他强行转换

##### JSON.stringify()

`JSON.parse(JSON.strigify())` 深拷贝时可得注意了哟。
_其实递归加对象解构来做深拷贝要更好一些哟。_

```js
JSON.stringify(Infinity); // 'null'
JSON.stringify(NaN); // 'null'
JSON.stringify(undefined); // undefined （注：非字符串）
JSON.stringify([undefined]); // '[null]'
JSON.stringify({ a: undefined }); // '{}'
JSON.stringify({ a: null }); // '{"a":null}'
JSON.stringify(() => {}); // 'undefined'
```

##### encode 系列

`encodeURI` 方法不会对下列字符编码 `ASCII字母、数字、~!@#$&*()=:/,;?+'`

`encodeURIComponent` 方法不会对下列字符编码 `ASCII字母、数字、~!*()'`

所以 `encodeURIComponent` 比 `encodeURI` 编码的范围更大。

##### 其他

```js
Array.from('foo'); // ["f", "o", "o"]
Object.assign([1], [2, 3]); // [2, 3]
```

---

大致就是这些了，写完要自闭一会，整个过程充满了怀疑与揣测。
虽然做了较为系统的拆分，但还是得承认没写好，敬请期待后续吧。

我还有一个 [BUG 库](https://github.com/foreverZ133/bugs)，不妨也分享出来一起看看吧。
