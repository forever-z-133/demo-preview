# 拓展底层功能的骚代码

- [on](#-给元素原型增加方法)（给元素原型增加方法）
- [for-of](#-让对象也能遍历)（让对象也能遍历）
- [mixin](#-类的多继承)（类的多继承）

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

## 为 label[for] 加拓展

https://gitee.com/zhangxinxu/smart-for

## 其他

```js
if (window.NodeList && !NodeList.prototype.forEach) {
  NodeList.prototype.forEach = Array.prototype.forEach;
}
```
