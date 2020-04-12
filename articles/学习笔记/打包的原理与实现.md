# 打包原理与实现

## 模块化

首先每个模块都有独立的作用域，那么它的打包结果可能就下面这样即可。

```js
// dist/bundle.js
(function () {
  // index.js
  var name = 'index';
  console.log(name);
})();
(function () {
  // moduleA.js
  var name = 'moduleA';
  console.log(name);
})();
```

## 先搞 module

假设当我想像下面这样放出一些数据别人时，也就是 module.exports

```js
// moduleA.js
console.log('hello');
module.exports = 'my name is moduleA';
```

那么我们的目标结果应该是下面这样，然后就可以跑在浏览器里了。

```js
// dist/bundle.js
(function () {
  var moduleA = function (require, module) {
    // moduleA.js
    console.log('hello');
    module.exports = 'my name is moduleA';
  };
  var module = { exports: {} };
  moduleA(null, module);
})();
```

## 再搞 require

假如现有两个文件，有引用依赖关系。

```js
// index.js
const moduleA = require('./moduleA');
console.log(moduleA);

// moduleA.js
module.exports = new Date().getTime();
```

那么我的目标结果应该是如下代码

```js
// dist/bundle.js
(function () {
  var moduleList = [
    function (require, module, exports) {
      // index.js
      const moduleA = require('./moduleA');
      console.log(moduleA);
    },
    function (require, module, exports) {
      // moduleA.js
      module.exports = new Date().getTime();
    },
  ];

  var moduleDepIdList = [{}, { './moduleA': 1 }];

  function require(id, parentId) {
    var currentModuleId = parentId !== undefined ? moduleDepIdList[parentId][id] : id;
    var module = { exports: {} };
    var moduleFunc = moduleList[currentModuleId];
    var _require = (id) => require(id, currentModuleId);
    moduleFunc(_require, module, module.exports);
    return module.exports;
  }

  require(0);
});
```

## 打包程序

可见，我的打包其实只是增加了 moduleList 和 moduleDepIdList 而已。

```js
// dist/bundle.js
(function () {
  var moduleList = [
    /* template */
  ];
  var moduleDepIdList = [
    /* template_of_depIds */
  ];
  function require(id, parentId) {
    /* 省略重复 */
  }
  require(0);
});
```

那么用 node 去读取和拼接字段串，然后替换掉 `/* template */` 即可。

只是嵌套调用时的映射稍稍麻烦而已。

## 拓展 ensure

像 `import(url).then` 或 `require.ensure(url).then` 这样的异步引入也是蛮常见的。

```js
// index.js
setTimeout(function () {
  require.ensure('./moduleA').then(function (context) {
    console.log(context);
  });
}, 5000);
```

那么咱们再来搞搞 ensure 函数的实现。

```js
const cache = {};

function ensure(chunkId, parentId) {
  var currentModlueId = moduleDepIdList[parentId][chunkId];
  var currentChunkStatus = cache[currentModuleId];
  if (currentChunkStatus === undefined) {
    var $script = document.createElement('script');
    $script.src = chunkId + currentChunkStatus + '.js';
    document.body.appendChild($script);
    var promise = new Promise((resolve) => {
      var chunkCache = [resolve];
      chunkCache.status = true;
      cache[currentModuleId] = chunkCache;
    });
    cache[currentModuleId].push(promise);
    return promise;
  }
  if (currentChunkStatus.status === true) {
    return currentChunkStatus[1];
  }
  return chunkCache;
}
window.__JSONP = function (chunkId, moduleFunc) {
  var currentChunkStatus = cache[chunkId];
  var resolve = currentChunkStatus[0];
  var module = { exports: {} };
  moduleFunc(require, module);
  resolve(module.exports);
};
```
