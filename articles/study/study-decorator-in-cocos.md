# 从 cc._decorator 中学习 ts 装饰器

在我们所写的所有 `cc.Component` 之前都会有 `@ccclass` 这玩意，感觉是个不错的分享选题，来讲讲装饰器。

- [cc._decorator 装饰器初体验](#KlJBS)
- [cc._decorator 有哪些装饰器](#ZVS9k)
- [装饰器是什么](#4f5TE)
- [如何写自己的装饰器](#4f5TE)

## cc._decorator 装饰器初体验

在 CocosCreator 中新建 ts 文件时，你有了下面这段代码：
```typescript
const { ccclass, property } = cc._decorator;
 
@ccclass
export default class NewClass extends cc.Component {
  @property(cc.Label)
  label: cc.Label = null;
}
```
而如果你新建的是 js 文件，则是这样的代码：
```typescript
cc.Class({
  extends: cc.Component,
  properties: {
    label: { default: null, type: cc.Label }
  },
});
```
撇开干扰项 `export default`，装饰器的感性认识还是挺直接的。  
阔以认为它是一种语法糖，将原本的包裹关系，用 `@` 标识符简化了一丢丢写法，看上去更平面化了。  
将其看做是返回一个函数的偏函数也行，更妙的是属性也能装饰。  

同理，我们感性认识上就很容易想到装饰器的美妙用法了。  

```typescript
@readonly
someDataObject: {}

@eventLog('事件埋点')
handleClick(e) {}

@watcher('data.a.b')
onDataABChange(newValue, oldValue) {}

@priceWithDot
@priceWithUnit('$')
@toFixed(2)
price: '1000'
```
## cc._decorator 有哪些装饰器
众所周知，使用 VScode 的 `Ctrl + 左键` 是有一定的代码溯源功能的，就能看到该属性或方法的 declare 声明或实际实现。  

那么在 `cc._decorator` 上进行溯源，也就能看到它内部在当时设计时会有哪些东西了。  

![image.png](https://cdn.nlark.com/yuque/0/2021/png/158440/1614567222238-3895545f-72d6-474d-9f73-561a1cd1854c.png#align=left&display=inline&height=632&margin=%5Bobject%20Object%5D&name=image.png&originHeight=632&originWidth=1247&size=85640&status=done&style=none&width=1247)

常用的 `@ccclass` 和 `@property` 就不提了。

#### @menu(path)
在组件菜单中插入一项，方便快速引入。
#### @requireComponent(comp)
表示必须带上某个组件，在引入时也会同时引入。  
比如某个需要画图的就带上 `cc.Graphics`，比如某个题目必会带上 `NormalMessage` 之类的。

![1.gif](https://cdn.nlark.com/yuque/0/2021/gif/158440/1614591263950-54fe1266-5d83-4979-917d-a2e0167a2f7d.gif#align=left&display=inline&height=497&margin=%5Bobject%20Object%5D&name=1.gif&originHeight=497&originWidth=560&size=97156&status=done&style=none&width=560)

#### @disallowMultiple
一个节点不可带有两个相同组件，类似 `cc.Sprite` 不可多个那样的功能
#### @executeInEditMode
通常是在运行时渲染，加上后可在 CocosCreator 中也渲染，如图所示。  
_但需注意，对于没有把握的代码，比如刚玩 gif 渲染做测试就不要开启，不然很容易造成编辑器卡死。_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/158440/1614650986873-7c241f4a-8010-4d2c-9a3f-d5eebe54668d.png#align=left&display=inline&height=580&margin=%5Bobject%20Object%5D&name=image.png&originHeight=580&originWidth=857&size=62191&status=done&style=none&width=857)

#### @playOnFocus
需与 `@executeInEditMode` 合用，当鼠标选中时渲染效果更优，暂时没遇到好场景。
#### @executionOrder
渲染顺序，默认为 `0`，越小越先，越大越后。除 `onLoad` `start` 外，且每次 `onEnable` 都如此。  
举个场景，比如先渲染子组件，然后父组件才开始使用它。  
#### @inspector(path)
直接修改编辑器的属性检查器面板，需要写一些 `Vue.component` 注册的代码，`path` 指向该 js 的路径。  
_个人暂未进行试验，有兴趣的阔以按官网推荐进行试玩：_  

[http://docs.cocos.com/creator/manual/zh/extension/extends-inspector.html](http://docs.cocos.com/creator/manual/zh/extension/extends-inspector.html)

#### @help(path)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/158440/1614670618714-5817a14e-9e5b-4bdf-950e-5a29ad257121.png#align=left&display=inline&height=423&margin=%5Bobject%20Object%5D&name=image.png&originHeight=423&originWidth=472&size=24269&status=done&style=none&width=472)

会多一个帮助文档按钮，`@help('https://www.baidu.com')` 你懂的

## 什么是装饰器
官方解释如下，基于上文的感性认知与案例后，想必也能够理解了。  
> 装饰器是一种特殊类型的声明，它能够被附加到类声明，方法，访问符，属性或参数上。  
> 装饰器使用 @expression 这种形式，expression 必须是一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。  

在其他编程语言中也常有，因为确实是很方便的语法糖。  
但需要区分的是，它是语言自带的还是阔以自建的，  
比如 `@override` 就是 dart 语言系统自带的，但它不能创建 proposal-decorators。  
而 typescript 只需在配置中进行开启，就阔以书写自定义的了。  
拓展阅读：[https://babel.docschina.org/blog/2018/09/17/decorators/](https://babel.docschina.org/blog/2018/09/17/decorators/)


此外，装饰器是设计模式装饰模式中的一种广泛实践，它与外观模式和代理模式等是有所不同的，感兴趣的阔以再去拓展熟悉一下。

## 如何写自己的装饰器
### 准备工作
命令行 `npm i -g typescript` 后，新建 `index.ts`、`tsconfig.json` 和 `index.html`，
```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```
```shell
## 命令行运行将生产 ts 编译后的 index.js
tsc -w
```
```html
<!-- 引入 index.js 后打开网页，即可看到 ts 中的打印啦 -->
<script src="index.js"></script>
```
### 类装饰器
```typescript
function withLog<T extends { new(...args: any[]): {} }>(target?: T) {
  return class extends target {
    log = 'test log';
  }
}
function defaultName(name: string) {
  return function<T extends { new(...args: any[]): {} }>(target: T) {
    return class extends target {
      defaultName = name;
    }
  }
}

@defaultName('zyh') // 带参的装饰器
@withLog // 不带参的装饰器
class Demo {
  name: string = '';
  constructor(name: string) {
    this.name = name;
  }
}

console.log(new Demo('hello'));
```
只需最终 `return` 的也是个 `class` 就行，如果需要带参就多套一层。


回头再看一眼 `@ccclass` 有两种形态，他们的源码实现应该也就不难猜到了。
```typescript
cc.Class({})
@ccclass class NewScript extends cc.Component {}
@ccclass('NewScript') class NewScript {}
```
### 方法装饰器
```typescript
function middleware(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const oldFunction = target[propertyKey]; // 获取方法引用
  const newFunction = function (...args: any[]) {
    console.log('call function ', propertyKey);
    oldFunction.call(target, ...args);
  }
  descriptor.value = newFunction; // 替换原声明
}

class Demo {
  @middleware
  demo() { console.log('call demo'); }
}

const x = new Demo();
x.demo();
```
其中，`target` 为所在的类的实例，`propertyKey` 为装饰的方法名，`descriptor` 为对象描述信息。算是方法装饰器比较特别的吧。  
_此时，[对象描述信息](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) 又是一个可以去拓展学习的点了 ，其实也是就是对象是否可配置可枚举可改写的那些配置。_


重新审视下上面的装饰器，你有发现什么问题吗？


是有问题的，假如我又写了个 `middleware2` 装饰在 `demo` 上，此时 `target[propertyKey]` 其实都是原函数，那么第二次修改的 `descriptor.value` 其实把第一次修改的相当于给覆盖掉了。   
具体分析可自行写代码来理解：[https://codepen.io/foreverZ133/pen/MWbGVqL](https://codepen.io/foreverZ133/pen/MWbGVqL)


再来一个问题， `@middleware @middleware2 demo() {}` 时，哪个会先执行？答案是 `@middleware2`。
### 属性装饰器
```typescript
function middleware(target: any, propertyKey: string) { }
```
语法相似，`target` 与 `propertyKey` 同上。


其实这玩意在我知道它只在初始化时才进行装饰时，有点希望破灭了的感觉，好像没卵用了呀。  
当其实看多一些 React 项目就不难发现，mbox 的 `@observable` 等不就是属性装饰器嘛。  
可能它更多的还是用在一些比较考验第三方能力的场景下，或者我还未能接触到的依赖注入等场景，思考不深。
```typescript
class Demo {
  @observable price = 0;
  @observable amount = 1;
  @computed get total() {
    return this.price * this.amount;
  }
  @timeCountDown countTime: 60;
}
```
### 函数参数装饰器
```typescript
function middleware(target: any, methodName: string, paramIndex: number) { }
```
语法稍有不同，methodName 为装饰的形参所在的函数名称，paramIndex 为形参所在位置。


实话实说，我尚未找到这类装饰器的有用场景。
## 总结

* cc._decorator 中的装饰器已用到了项目中
* 阔以搭个 ts 环境写写自定义装饰器
* 欢迎分享更多装饰器的使用场景
