# UniApp 的全局组件注入的问题记录

## 可用方案1

```ts
// main.ts
import { createSSRApp } from 'vue'
import BaseIcon from './global-components/base-icon.vue'
import App from './App.vue'

export function createApp () {
  const app = createSSRApp(App)
  app.components('base-icon', BaseIcon)
  return { app }
}
```
在这种方式下，打包后组件才位于 `app.json` 中，是真正的全局组件
```
{
  "usingComponents": {
    "base-icon": "/global-components/base-icon"
  }
}
```

## 错误方案1

新建 `/global-components/index.ts` 去集合所有组件，然后 `import` 全部，再遍历注册。  
但是，最终没有注册成功。

```ts
// ./global-components/index.ts
export { default as BaseIcon } from './base-icon.vue'
```

```ts
// main.ts
import { createSSRApp } from 'vue'
import type { Component, App as Vue } from 'vue'
import * as globalComps from './global-components/index'

type GlobalComponents = Record<string, Component & { __name: string }>
function installGlobalComponents (app: Vue, comps: GlobalComponents) {
  Object.keys(comps).forEach((key) => {
    const comp = comps[key]
    const name = comp.__name || key
    console.log('全局注册组件:', name, comp)
    app.component(key, comp)
  })
}

export function createApp () {
  const app = createSSRApp(App)
  installGlobalComponents(app, globalComps)
  return { app }
}
```

## 错误方案2

使用 `import.meta.glob` 也是与上面类似的思路。  
但是，最终也没有注册成功。

```ts
import type { Component, App as Vue } from 'vue'

function installGlobalComponents (app: Vue) {
  type GlobalComponents = Record<string, { default: Component }>
  const modules: GlobalComponents = import.meta.glob('./global-components/*.vue', { eager: true })

  Object.keys(modules).forEach((url) => {
    const baseName = url.split('/').pop() || ''
    const name = baseName.slice(0, -4)
    const comp = modules[url].default

    if (!name) return

    console.log('全局注册组件:', name, comp)
    app.component(name, comp)
  })
}
```

## 错误方案3

此时有点意识到，恐怕只是将 `app.component()` 提取到别处，都会是注册不成功的表现。

```ts
// main.ts
import installGlobalComponents from './global-components/install.ts'

export function createApp () {
  installGlobalComponents(app)
}
```
```ts
// ./global-components/install.ts
import { default as BaseIcon } from './base-icon.vue'

export default function installGlobalComponents (app: any) {
  app.component('base-icon', BaseIcon)
}
```

结果来看，果然如此，不得不吐槽 `UniApp` 竟然限定如此死，只能在 `main.ts` 中注册。  
*以后得闲再翻翻它的源码看下。*

## 可用方案2

https://uniapp.dcloud.net.cn/collocation/pages.html#easycom  
`UniApp` 中有个 `easycom` 的机制，可以以此来自动导入组件。需在 `pages.json` 中进行配置：

```json
{
  "globalStyle": {},
  "easycom": {
    "autoscan": true,
    "custom": {
      "^base-(.*)": "@/global-components/base-$1.vue"
    }
  }
}
```

但它与 *可用方案1* 的区别在于，打包后该组件是位于 `src/pages/xx/x.json` 页面中的。

```json
{
  "navigationBarTitleText": "某页面",
  "usingComponents": {
    "base-icon": "../../global-components/base-icon"
  }
}
```

## 总结

鉴于以下几点，最终选择 *可用方案2*：

1. 主包体积比分包体积更珍贵，所以组件置于页面中比全局更好
2. *可用方案1* 只能在 `main.ts` 中不断增加 `import`，可能不太好看

