# 服务端渲染 SSR 梳理

其实 SSR 的难点并不多，只是需要对各框架有一定熟悉即可，可能你看完代码就懂了。

*如果你采用了 `next.js` 或 `nuxt.js` 之类的同构框架，那直接梭哈即可，只是 `build` 与 `export` 的表现稍稍有点理解难度。*

## 架构思路

其实自建一个服务端渲染也是相当容易的，先拿 React 技术栈举例，  
只需要再加上 `webpack` `@babel/preset-react` `express` 即可，  
下面的思路一摆出来，想必你已经阔以按自己的想法尽情发挥去实现了。  

1. 启动服务，监听路由
2. 对应路由下解析 `jsx` 注入数据
3. 导出 `html`

对应到实操虽然牵扯框架较多，但也是很清晰的：

* 先学习 `express` 和 `koa` 如何启动服务和监听路由，并知道如何返回数据
* 路由过程中需要调用 `react-dom/server` 的 `renderToString` 函数去转化得到 `html`
* 但此时 `node server.js` 会报错不认识 `jsx`
* 于是你需要 `@babel/preset-react` 来帮忙解析
* 为此你还得用上 `webpack` 或 `rollup` 之类的构建工具
* 先打包把 `jsx` 转成函数，再启动服务，整个流程就通了
* 为了更方便调试，再加上 `nodemon` `npm-run-all` 等工具包慢慢优化即可

## React 栈 SSR 展示

> ##### 新建项目，安装基础依赖。

```
npm init -y
npm i react react-dom
npm i express
npm i -D webpack webpack-cli webpack-node-externals
npm i -D @babel/core @babel/preset-env @babel/preset-react babel-loader
```

> ##### 配置 babelrc，完成 client 端代码

```json
// .babelrc
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

```js
// webpack.client.js
const path = require("path");
module.exports = {
  entry: "./src/client.js",
  output: {
    filename: "bundle-client.js",
    path: path.resolve(__dirname, "public"),
  },
  module: {
    rules: [
      { test: /\.js$/, loader: "babel-loader", exclude: /node_modules/ },
    ],
  },
};
```

```jsx
// src/pages/Home.js
import React from "react";
const Home = props => <h1>{props.text || 'hello'}</h1>;
export default Home;

// src/client.js
import React from "react";
import { render } from "react-dom";
import Home from "./pages/Home";
render(<Home />, document.getElementById("root"));
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>
    <script src="public/bundle-client.js"></script>
  </body>
</html>
```

```json
// package.json
{
  "scripts": {
    "prod:build:client": "webpack --mode production --config webpack.client.js",
    "test:client": "start ./index.html", // 若使用 mac 需把 start 替换为 open
  }
}
```

```
npm run prod:build:client && npm run test:client
```

> ##### 完成 server 端代码

```js
// webpack.server.js
const path = require("path");
const webpackNodeExternals = require("webpack-node-externals");
module.exports = {
  target: "node",
  entry: "./src/server.js",
  output: {
    filename: "bundle-server.js",
    path: path.resolve(__dirname, "public"),
  },
  module: {
    rules: [
      { test: /\.js$/, loader: "babel-loader", exclude: /node_modules/ },
    ],
  },
  externals: [webpackNodeExternals()],
};
```

```js
// src/server.js
import React from "react";
import express from "express";
import { renderToString } from "react-dom/server";
import Home from "./pages/Home";
const app = express();
app.use(express.static("public"));
app.get("*", (req, res) => {
  const content = renderToString(<Home text="hello ssr" />);
  const html = `<html><body>${content}</body></html>`;
  res.send(html);
});
app.listen(8001, () => console.log("listening on port 8001"));
```

```json
// package.json
{
  "scripts": {
    "prod:build:server": "webpack --mode production --config webpack.server.js",
    "test:server": "node ./public/bundle-server.js"
  }
}
```

```
npm run prod:build:server && npm run test:server
```

实例代码：https://github.com/forever-z-133/test-react-ssr/tree/simple-ssr

## 其他框架下的配置

#### CRA 脚手架中的配置

在 `create-react-app` 脚手架中，服务端的打包也是类似的。

```
npm i -D react-scripts react-app-rewired
```

```js
// config-overrides.js
const path = require("path");
const webpackNodeExternals = require("webpack-node-externals");
const isSSR = process.env.TARGET === 'ssr ';
module.exports = {
  webpack: config => {
    if (isSSR) {
      config.target = 'node'; // 导出 node 环境代码
      config.entry = path.resolve(__dirname, 'src/server.js');
      config.output.filename = 'bundle-server.js'; // output 不分包命名
      delete config.optimization.splitChunks;
      delete config.optimization.runtimeChunk;
      const rules = config.module.rules[1].oneOf; // 加入 @babel/preset-react
      rules.forEach(rule => {
        if (!rule.loader || !rule.loader.includes('babel-loader')) return;
        rule.options.presets.push('@babel/preset-react');
      });
      if (!config.externals) config.externals = []; // 规避不必要的 node_modules 导出
      config.externals.push(webpackNodeExternals());
      const excludes = ['HtmlWebpackPlugin', 'InlineChunkHtmlPlugin', 'InterpolateHtmlPlugin', 'ManifestPlugin']; // 去除不必要的 plugins
      config.plugins = config.plugins.filter(plugin => !excludes.includes(plugin.constructor.name));
    }
    return config;
  },
};
```

```json
// package.json
{
  "scripts": {
    "build:server": "set TARGET=ssr && react-app-rewired build",
    "test:server": "node ./build/bundle-server.js"
  }
}
```

```
npm run build:server && npm run test:server
```

实例代码：https://github.com/forever-z-133/test-react-ssr/tree/cra-ssr

#### koa 服务框架的配置

```
npm i -D koa koa-router koa-mount koa-static
```

```js
// src/server.js
import React from "react";
import path from "path";
import "regenerator-runtime";
import Koa from "koa";
import Router from 'koa-router';
import mount from 'koa-mount';
import staticly from 'koa-static';
import { renderToString } from "react-dom/server";
import Home from "./pages/Home";
const app = new Koa();
const router = new Router();
router.get("/", async ctx => {
  const content = renderToString(<Home text="hello ssr" />);
  const html = `<html><body>${content}</body></html>`;
  ctx.body = html;
});
app.use(router.routes()).use(router.allowedMethods());
app.use(mount('/', staticly(path.join(__dirname, './build'), { maxage: 24 * 60 * 60 * 1000 })));
app.listen(8001, () => console.log("listening on port 8001"));
```

#### 快速调试

若想监听文件变化进行 hot reload，则得靠 `webpack --watch` 和 `nodemon` 了。

```
npm i -D nodemon npm-run-all
```

```json
// package.json
{
  "scripts": {
    "dev": "npm-run-all --parallel dev:server:* dev:client",
    "dev:client": "webpack --mode development --config webpack.client.js --watch",
    "dev:server:build": "webpack --mode development --config webpack.server.js --watch",
    "dev:server:start": "nodemon ./build/bundle-server.js"
  }
}
```

## 更多服务端渲染问题

#### 样式


#### 事件


#### 路由

其实有两套方案，各有优劣，可依实际需求而定：

* 访问后返回该页的渲染，并带上所有代码，后续只有刷新才会重新请求服务端渲染
* 导出每页的渲染，非单页应用，后续每次跳页都是会请求服务端渲染

## Vue 栈 SSR 展示
