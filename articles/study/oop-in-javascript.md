# JS 的面向对象

说到 JS 的面向对象就有一堆关键词，类/构造函数/实例/继承/原型/原型链，基操，勿慌。

## 构造函数

在 JS 被设计之初，Brendan Eich 就不太想加入很多面向对象的东西。  
因此也并不想加入 `class` 这种关键字，免得被当成面向对象的语言来开发。

但万物皆对象的世界里能有继承的话能大大提升开发效率，  
于是乎，**从原型对象生成一个实例对象** 这种骚操作就出现了。  
而生成实例前通常还会有个 `constructor`，因此就有了 **构造函数** 这种编写方式。

```js
function Animal(name) {
  this.name = name;
}
var animal = new Animal("动物");
```

看上面这种利用 `new` 关键字，再看下面这种生成对象的写法，你细品。

```js
function Animal(name) {
  return {
    name: name,
  };
}
var animal = Animal("动物");
```

显然用 `new` 写得更明确一些，更有一种基础对象的感觉。

## 原型

但既然是基础对象，那么就应该存在某种数据共享的特性。  
比如 `Animal` 有个 `amount` 存着总数，要改变总不能获取每个实例都改吧。

用全局变量肯定也不妙呀，因此骚操作又来了，也就是 `prototype`。  
专门让构造函数能公用一些数据或方法。

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.amount = 0;

var dog1 = new Animal("大狗");
var dog2 = new Animal("二狗");
Animal.prototype.amount = 2;
console.log(dog1.amount, dog2.amount); // 2 2
```

略作思考，明显这能当低配版的继承了，或者说混合也行，可以理解为下面两个类的合体。

```js
function Animal(name) {
  this.name = name;
}
function AmountManager() {
  this.amount = 0;
}
```

## 继承

如果按正经的面向对象来看，那还应该有 `super` 才对。  
因此就有了一些用 JS 来实现继承的面试考题（虽然当年写 JS 面向对象的真不多）。

过去流行的六种 JS 继承方式：

1. 原型链继承 `Child.prototype = new Parent(arg)`
2. 构造函数继承 `Parent.call(Child, arg)`
3. 组合继承（包含 1 和 2）
4. 原型式继承（拷贝父类实例的原型）
5. 寄生式继承（在 4 的基础上再包一层）
6. 寄生组合式继承（在 4 的基础上让父类原型走 3）

几个方案各有优劣吧，也是最能考验原型知识点的，所以深入研究摸索一下也无坏处。

## 原型链

显然，当继承关系更复杂后，有个能知悉其关系的方式就好了，因此有了原型链的概念。
虽然在没有 class 关键词前它其实并不能很好地帮忙排查，嘤嘤嘤。  

所以基本也就能在面试过程中拿出来装装逼吧。

```js
const foo = { a: 1 };
const bar = Object.create(foo); bar.b = 2;
const baz = Object.create(bar); baz.c = 3;

/// 判断某属性是否在原型链中
function findPrototypeByProperty(obj, propertyName) {
	let proto = obj;
	while(proto != Object.prototype) {
		if (proto.hasOwnProperty(propertyName)) return true;
		proto = proto.__proto__;
	}
	return false;
}
console.log(findPrototypeByProperty(baz, 'a'));
console.log(findPrototypeByProperty(baz, 'b'));
console.log(findPrototypeByProperty(baz, 'c'));
```
