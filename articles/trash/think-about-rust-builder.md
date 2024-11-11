# 关于 Rust 锈化趋势的思考

众人都说：Rust 对前端工程领域的侵入已成不可逆之势。

比如 `SWC` 对标 `Babel`、`RsPack` 对标 `Webpack`、`OxLint` 对标 `EsLint` 等等，还有 `TurboPack`、`RollDown` 等等。

但是，就和大多数的互联网中的吹捧一样，还是得有自己的思考和判断。

## 技术偏见与沉默成本

我已工作八年，一直书写 `JavaScript` 和 `TypeScript`，偶尔使用 `Python` 或 `Dart`。

在理解、优化和调试等这方面，积累的都是这些语言的，各种细节上投入了大量的职业生涯，它是迄今为止我最舒适的语言。

当然这不是借口，因为其实已经学会了，只是蛮少有 `Rust` 实战机会，或遇到了问题并不能得心应手。

所以我得承认我有着偏见，甚至因此会对前端的未来感到怀疑，如此“快”的语言对用户或绩效的体感是否也能如此显现。

或者说，继续在 `JavaScript` 中探索并不会遇到 `ActionScript` 那样的落幕，也不会害怕被 `Flutter` 等取代。

以下是我的客观思考，也是对锈化造神运动的反驳。

## JS 性能并没有被穷尽

首先，我真的不认为我们已经穷尽了使 `JavaScript` 工具更快的所有可能性。

比如 CPU 更擅长处理数字，那么不必将数字转换为字符串。

```js
function toFixed(num, precision) {
	const pow = 10 ** precision;
	return Math.round(num * pow) / pow;
}

2.3491.toFixed(2);
toFixed(2.3491, 2); // 数字运算更快 5%
```

比如不必要的内联函数、递归等等。

```js
const _loop = function _loop(query, dependents) {
  doSomething();
};
for (var [query, dependents] of precedenceLookup.entries()) {
  _loop(query, dependents);
}

// 去掉无用的函数调用，更快 10%
for (let [query, dependents] of precedenceLookup.entries()) {
  doSomething();
}
```

比如正则的回溯机制、贪婪匹配、捕获机制等等。

```js
.match(/(?:<input)(.*)(?:>)/)[1]
.match(/<input(.*?)>/)[1] // 更快 8%
```

## 锈化的重写程度

我认为性能差距来自几个不同的因素。首先，就是前面提到的低效率问题，现在我们已经达到了一个饱和点。

而现在的 `RollDown`、`OxLint` 等，可能其实都是在重写，而非优化。

https://www.youtube.com/live/0F9t_WeJ5p4?t=4234s

## 字节码 vs JIT

性能差距的第二类来自浏览器免费提供给我们的东西，这些东西我们很少考虑：字节码缓存 和 `JIT`（`Just-In-Time` 编译器）。

当你第二次或第三次加载一个网站时，如果 `JavaScript` 正确缓存，则浏览器不需要再将源代码解析并编译成字节码。

如果一个函数是“热”的（频繁执行），它将被进一步优化成机器代码。这就是 `JIT` 的作用。

在 `Node.js` 脚本的世界里，我们根本无法获得字节码缓存的好处。

## 社区与贡献

`JavaScript` 是一种工人阶级语言,它有相当多的开发者在使用它实现业务功能。

它容易上手，在浏览器中即时编译所见即所得，甚至可以在 `node_modules` 中直接修改。

如果 `Oxc` 和 `VoidZero` 等框架出现问题，对未学习过 `Rust` 语言的开发者来说，他们可能坐等更新而无计可施。

https://sorrycc.com/why-im-skeptical-of-rewriting-javascript-tools-in-faster-languages/
