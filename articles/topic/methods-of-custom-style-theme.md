# 如何实现主题换肤功能

## 方案一：全局覆写主题样式

```css
/* ./styles/theme/blue.less */
html.blue { color: blue; }

/* ./styles/theme/green.less */
html.green { color: green; }
```

```js
// ./main.js
import './styles/theme/blue.less'
import './styles/theme/green.less'
```

```html
<!-- ./index.html -->
<html class="blue"></html>
<!-- 当然你也可以靠 `js` 注入和修改主题 `document.documentElement.classList.add('blue')` -->
```

设定一个优先级比较高的样式类，去覆写全局样式。

但是，因为是全量引入，样式文件体积会较大。

## 方案二：动态引入导出多份主题文件

```js
const theme = 'blue';
import(`./styles/theme/${theme}.less`);
document.documentElement.classList.add(theme);
```

虽然此处只使用了 `blue` 变量，但由于 [打包时 import 处理](/articles/topic/require-async-in-webpack.md) 的机制，<br />
最终 `./theme/*` 文件夹下的所有文件都会分别打包，比如获得 `assets/blue.css` 和 `assets/green.css`，所以还需注意文件结构。

每次只加载了一份主题样式文件，以此减少了包体积。

另外，切换主题时，由于无法比较简单的清除之前的主题样式，所以还是需要一个顶层的主题样式类。

## 方案三：云端编译

可以把编译主题样式文件的功能放到云上处理，用 `SSR` 也好，用前端网关也好，比如以下简易代码。

```js
/* cloud/index.mjs */
import Koa from 'koa';
import { URL } from 'node:url';
const app = new Koa();

app.use((ctx) => {
  if (ctx.originalUrl.includes('/styles/theme.css')) {
    const params = new URL(ctx.href).searchParams;
    ctx.type = 'text/css';
    ctx.body = `:root { color: ${params.get('color')}; }`;
  }
});

app.listen(5172);
```

```html
/* ./index.html */
<link rel="stylesheet" href="http://localhost:5172/styles/theme.css?color=red">
```

当然，`localhost:5172` 你可以用代理给去掉；<br />
`ctx.body` 可以改为 `CSS` 预处理器命令进行编译；<br />
`app.use` 也可以加上缓存，相同入参数就走 `CDN` 模式；<br />
`color=red` 也可以改为其他更符合业务需求的参数，等等。

此方案的优点是没有了主题样式类，直接就是当前主题的结果；<br />
主题或者说样式完全独立维护，且可以分场景去动态编译。

只不过，可能调试起来相对麻烦一丢丢。

## 方案四：使用 filter: invert() 反转颜色

如果你只需要黑白两色主题，也可以使用 `filter` 属性来反转颜色。

```less
html.light {}
html.dark {
  filter: invert(0.9) hue-rotate(.5turn);
  img, video { filter: none; }
}
```

此方案足够简单。但可以会有预期外的表现，比如避免 `img` 等资源是负片效果就需要再次反转处理。

## 方案五：使用 mix-blend-mode: difference 反转颜色

此方案与方案四类似，也只支持黑白两色主题，也会有预期外的表现。

```less
/** ./styles/index.css */
/** 注意：一定要有个白底 */
body {
  background: #fff;
}
.dark-mode-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
html.light {}
html.dark {
  .dark-mode-layer {
    background: #fff;
    mix-blend-mode: difference;
    z-index: 99999;
  }
  img, video {
    position: relative;
    z-index: 99999;
  }
}
```

```html
<html class="light">
  <body>
    <div class="dark-mode-layer"></div>
    <div id="app"></div>
  </body>
</html>
```

由于一定要有白底，且有层叠方面的骚操作，会比方案四的限制还多些，不太推荐。

## 方案六：使用 less 方法在运行时编译

在运行 `js` 时再赋予主题色的变量值，所以能支持任意主题色。

#### less.modifyVars

```html
<!-- ./index.html -->
<link rel="stylesheet/less" href="/index.less" />
```

```js
import { modifyVars } from 'less';

modifyVars({ '@color': 'red' });
```

由于 `less` 文件 `vite` 框架引入后都会被编译，暂未找到办法跳过，所以只能将 `index.less` 置于 `public` 文件夹中。

#### less.render

```js
import('./styles/index.less?raw').then(async (res) => {
  const cssText = res.default || '';
  const styleId = 'theme-style';
  const cacheStyle = document.querySelector(`#${styleId}`);
  const style = cacheStyle || document.createElement('style');
  style.id = styleId;
  const options = {
    modifyVars: { '@color': 'yellow' },
  }
  const result = (await render(cssText, options)).css;
  style.innerHTML = result;
  if (!cacheStyle) document.head.appendChild(style);
});
```

TODO: 在 `less` 中使用 `@import` 会报找不到路径的错误，尚在摸索。

## 方案七：使用 CSS 变量

```less
/* 定义阶段 */
/* ./styles/theme/blue.less */
.blue() { --color: blue; }

/* ./styles/theme/green.less */
.green() { --color: green; }
```

```less
/* 使用阶段 */
/* ./styles/index.less */
@import './theme/blue.less';
@import './theme/green.less';

html.blue { .blue(); }
html.green { .green(); }
html { color: var(--color); }
```

此方案，定义阶段更为纯粹，使用阶段也更灵活和语义化，且业务样式无需关联主题样式。

另外，仅仅多了一些变量定义而已，所以包体积也很小。

再者，如果定义写得够细的话，后续调整或新增主题也十分方便。

```js
// ./main.js
document.documentElement.style.setProperty('--color', 'red');
```

而且，此方案可使用任意主题色，或在运行时动态赋予不同的变量值。

所以，此方案是市面上最流行的实现方式。

```less
/* 个人不推荐拿 Less 变量再转一遍 */
/* ./styles/vars.less */
@color: var(--color);

/* ./styles/pages-a.less */
@import './vars.less';
.wrapper { color: @color; }
```

至于 `CSS` 变量要不要再转为 `Less` 变量，方便了解变量来源，我个人觉得没有必要。
