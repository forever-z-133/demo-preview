# 惰性函数&偏函数&科里化

当某个函数我只想处理一次，然后下次直接拿实例；<br />当某个判断我只想处理一次，然后下次直接往下走；<br />当某个变量我只想是临时的，然后不用到处是全局变量......<br />遇到这些问题，已经有各式各样的答案，甚至设计模式，<br />那么，今天就来盘点这些偷懒的操作。

## 惰性函数

### 初步实践与优点

```javascript
function func() {
  /* 运算/判断/变量 都可以加进来 */
  func = function () {
    /* 然后使用上面结果，又覆盖掉同名函数，来达到无需再进行上面计算的功能 */
  };
}
```

可见，惰性函数充分利用了 js 的动态性，<br />众所周知，使用非 var 变量运行到才赋值定义，<br />使得函数在运行后被同名函数覆盖，<br />但原函数的上下文又得以保存在新函数的上下文中，<br />以此来到达去除重复运算，返回单例，保留公用变量的功能。

举个市面上常见的例子，很好理解<br />这样下次再运行 addEvent 时就不会再进行判断了。

```javascript
function addEvent(type, element, fun) {
  if (document.addEventListener) {
    addEvent = function (type, element, fun) {
      element.addEventListener(type, fun, false);
    };
  } else if (document.attachEvent) {
    addEvent = function (type, element, fun) {
      element.attachEvent('on' + type, fun);
    };
  } else {
    addEvent = function (type, element, fun) {
      element['on' + type] = fun;
    };
  }
}
```

假如不用惰性函数，就会需要使用一个变量才能才能达到相同效果，如下：

```js
let addEvent;
if (xxx) {
  addEvent = function () {};
} else if (yyy) {
}
```

更多例子，同步运算，或者缓存结果，或者某公用但不想全局的变量，也可以用惰性函数来完成。

```javascript
// 比如缓存些固定量
function _electronVersion() {
  const data = require("app-builder-lib/out/electron/electronVersion");
  _electronVersion = function () { return data; };
  return data;
}
// 或者组合些固定量
function getParams(moreParams) {
  const user = { userId: 1 }; // 这是个某不变的量或运行结果
  getParams = function (moreParams) {
    return Object.assign({}, user, moreParams);
  };
  return getParams(moreParams);
}
```

### 与科里化和偏函数的对比

很容易发现，这样的使用场景和科里化与偏函数的使用场景有很多相似之处，也有明显的不同之处。

- 科里化和偏函数是为了包装函数，惰性函数是为了重写函数，有根本上的不同。
- 惰性函数直接调用，科里化和偏函数需返回函数再使用。
- 科里化和偏函数由于需要返回函数，所以必须靠前，惰性函数则不用。
- 惰性函数的形参是定死，没法抽象拓展

## 偏函数

```javascript
function partial(func, ...rawArgs) {
  return function (...args) {
    return func.call(this, ...rawArgs, ...args);
  };
}
```

```javascript
// 原方法
let greet = (x, y) => return `${x} ${y}`;

// 偏函数改造
greet = partial(greet, 'hello');
console.log(greet('zyh'));   // "hello zyh"
```

```javascript
const addEvent = (function () {
  if (document.addEventListener) {
    return function (type, element, fun) {
      element.addEventListener(type, fun, false);
    };
  } else if (document.attachEvent) {
    return function (type, element, fun) {
      element.attachEvent('on' + type, fun);
    };
  } else {
    return function (type, element, fun) {
      element['on' + type] = fun;
    };
  }
})();
```

非常适用于存储变量和计算。<br />也可以再写个  partialAfter 之类的公共方法。

## 函数科里化

市面上的例子常常把前两者都算作科里化的一种实现，<br />但其实 addEnvet 和 addMore 都更像是偏函数，而科里化是偏函数的进一步包装。

```javascript
function curry(func) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func.apply(this, args);
    } else {
      return function (...args2) {
        return curried.call(this, ...args, ...args2);
      };
    }
  };
}
```

```javascript
// 原方法
let log = (date, type, message) => console.log(date, type, message);

// 科里化改造
log = curry(log);
const todayLog = log(new Date());
todayLog('ERROR', 'xxx');
const todayTimeoutLog = log(new Date(), 'TIMEOUT');
todayTimeoutLog('xxx');
```

可见，科里化与偏函数根本的区别就是，<br />科里化使得变量可以更便捷地分批传入，而并不是说偏函数不能够，<br />所以至于实际情况用哪个，见仁见智，都是很不错的。

```javascript
const todayLog = partial(log, new Date());
const todayTimeoutLog = partial(todayLog, 'TIMEOUT');
```

## 其他

### 惰性函数的其他踩坑

- 严格模式下是否能用（能）
- 跨模块调用是否能用（能）
- 是否有适用的异步场景（还在思考）

### 科里化的另一种玩法

面试题中常见的，可以无限加下去的科里化，add(1)(2, 3)...<br />这就只能靠修改函数的 toString 方法来实现了。

```javascript
function curry2(func) {
  let allArgs = [];
  return function curried(...args) {
    allArgs = allArgs.concat(args);
    curried.toString = function () {
      return func.call(this, ...allArgs);
    };
    return curried;
  };
}

var addInfinity = curry2(add);
console.log(addInfinity(1, 2)(3)(4));
```

### 还有种存数据的办法

众所周知，不想存全局变量时，存在 function 变量上也是可以的。

```javascript
// 比如常见的请求拦截，正在请求中则调用无效，不用另外搞个变量了
function getData() {
  if (getData.loading) return;
  getData.loading = true;
  setTimeout(() => {
    delete getData.loading;
  }, 1e3);
}
```
