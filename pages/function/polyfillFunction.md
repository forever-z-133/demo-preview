# 拓展底层功能的骚代码

- [on](#-给元素原型增加方法)（给元素原型增加方法）
- [for-of](#-让对象也能遍历)（让对象也能遍历）
- [mix](#-类的多继承)（类的多继承）
- [label[for]](#-为-labelfor-加拓展)（为 label[for] 加拓展）
- [NodeList.forEach](#-nodelistforeach)（NodeList.forEach）
- [softBind](#-软绑定-softBind)（软绑定 softBind）
- [codePointAt / fromCodePoint](#-utf16-字符串获取)（utf16 字符串获取）

## \* 给元素原型增加方法

```js
// 给所有 html 元素附加方法
(function (arr) {
  arr.forEach(function (item) {
    if (item.on) return;
    //例如：window.on('click', () => {});
    Object.defineProperty(item, "on", {
      configurable: true,
      enumerable: true,
      writable: true,
      value: function on(event, fn) {
        this.addEventListener(event, fn);
      },
    });
  });
})([Element.prototype, window]);
```

## \* 让对象也能遍历

```js
Object.prototype[Symbol.iterator] = function* () {
  for (const [key, value] of Object.entries(this)) {
    yield { key, value };
  }
};
// 例如：const obj = { a: 1, b: 1 }
// for (let item of obj) console.log(item, item.key, item.value);
```

## \* 类的多继承

```js
// 例如：class Dog extends mix(Animal, Friendly) {}
function mix(...mixins) {
  class Mix {
    constructor(...ags) {
      for (let mixin of mixins) {
        copyProperties(this, new mixin(...ags));
      }
    }
  }
  for (let mixin of mixins) {
    copyProperties(Mix, mixin);
    copyProperties(Mix.prototype, mixin.prototype);
  }
  return Mix;
}
function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if (key !== 'constructor' && key !== 'prototype' && key !== 'name') {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```

## \* 为 label[for] 加拓展

https://gitee.com/zhangxinxu/smart-for

## \* NodeList.forEach

```js
if (window && window.NodeList && !NodeList.prototype.forEach) {
  NodeList.prototype.forEach = Array.prototype.forEach;
}
```

## \* 软绑定 softBind

```js
// 1. 解决只能 bind 一次，后续调整 this 无效的问题
// 2. 在 bind 时可科里化，提前或分批传入参数
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function (obj) {
    var fn = this;
    var curried = Array.prototype.slice.call(arguments, 1);
    var bound = function () {
      return fn.apply((!this || this === (window || global)) ? obj : this, curried.concat.apply(curried, arguments));
    }
    bound.prototype = Object.create(fn.prototype);
    return bound;
  }
}
```

## \* utf16 字符串获取

```js
// '𠮷'.codePointAt(0); 
// String.fromCodePoint(0x20BB7);
if (!String.prototype.codePointAt) {
  String.prototype.codePointAt = function(idx = 0) {
    const code = this.charCodeAt(idx);
    if(code >= 0xD800 && code <= 0xDBFF) {
      const high = code;
      const low = this.charCodeAt(idx + 1);
      return ((high - 0xD800) * 0x400) + (low - 0xDC00) + 0x10000;
    }
    return code;
  }
}
if (!String.fromCodePoint) {
  String.fromCodePoint = function(...codePoints) {
    let str = '';
    for(let i = 0; i < codePoints.length; i++) {
      let codePoint = codePoints[i];
      if(codePoint <= 0xFFFF) {
        str += String.fromCharCode(codePoint);
      } else {
        codePoint -= 0x10000;
        const high = (codePoint >> 10) + 0xD800;
        const low = (codePoint % 0x400) + 0xDC00;
        str += String.fromCharCode(high) + String.fromCharCode(low);
      }
    }
    return str;
  }
}
```
