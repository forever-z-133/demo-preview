# webpack 打包时对 import 和 require 的处理 

<a name="df368884"></a>
## 前言

import 和 require 的区别

1. import 是 es6 的，require 是 commonJs 的
2. import 是静态导入，require 是动态导入

既然 import 是静态导入，那 import 就不能放在 block 中，例如

```javascript
{
  import xxx from "xxx";
}
// 会抛出一个受检异常
```

再例如：

```javascript
if (localStorage.getItem("random")) {
  import xxx from "xxx";
}
// 会抛出一个受检异常
```

同样会抛出一个受检异常，第二个例子是不是更好理解呢。<br />
因为 **import** 是静态导入，所谓的静态就是在 **buildtime** 完成文件解析处理，那么由于 **buildtime** 并不会执行 **runtime** js 代码，更没有 localStorage 这个东东，怎么知道该不该导入 **xxx** 这个依赖呢，所以 **import** 不能放在 block 中。

> 相反，require 是 commonjs 中的动态导入，可以放在代码块中。



<a name="ced8ec7f"></a>
## 问题来了

webpack 对 import 和 require 都做了支持，那么如果 webpack 碰到动态加载的 require 会怎么样呢。

例子：

教学易中有个场景是通过指定编译需要注册三个不同的登录页路由。

```javascript
const loginPath = "xxx"; // 登录页路径
const loginRoute = (
  <Route path={loginPath} component={require(`./routes/${loginPath}/`)} />
);
```

假设 **routes** 下面有 `login.js`, `zxLogin.js`, `sklLogin.js` 三个登录页，项目运行时 **loginPath** 变量才会有值，其值为三个登录页中的一个。

打包好后，系统运行正常，路由正常，也实现了登录页的按需注册。那么问题来了，webpack 打包时**loginPath** 变量还没有值呢，他怎么知道该帮我打哪个包呢。（我们都知道 webpack 是根据 entry 来打包的，没有 import 或 require 的文件是不会打包进来的）

好奇心驱使看了下打包后的 index.js 文件，原来 webpack 在解析 **require** 时把所有 **routes** 路径下的文件都给我打包进来了，**buildtime** 不知道变量 **loginPath** 的值干脆全给你打包进来吧。

需求是不想把所有文件都打包进来的，那怎么办呢，那就试试异步加载吧。


<a name="45745294"></a>
## 异步加载

import 和 require 都有对应的异步加载方式，分别为：

```javascript
// import()  提案
import("xxx").then(() => {});

// require.ensure
require.ensure([], function(require) {
  var xxx = require("./xxx");
});
```

其中 `require.ensure` 已经被 webpack 所支持，`import()` 则需要借助 babel 插件 `babel-plugin-syntax-dynamic-import`

react-router 结合异步加载的代码：

```javascript
const loginPath = "xxx"; // 登录页路径
<Route
  path={loginPath}
  getComponent={(nextState, callback) => {
    import(`./routes/${loginPath}/`).then(res => callback(null, res));
  }}
/>;
```

经过上述改良后，webpack 会帮你打包 4 份文件，分别为 `index.js`, `0.js`, `1.js`, `2.js`。其中数字是 webpack 默认的 **chunk** 命名方式，分别就是三个登录页 js，这样就能在 **runtime** 中根据需要去异步加载对应的 js。

**require.ensure** 虽然也能实现异步加载，但是最终还是要在函数中同步使用**require**引用，结果跟没优化前差不多，只是将三个登录页文件打包到了一个 `0.js` 文件中。

