# 小程序打包体积优化

基础的理论就不讲了，多半你的项目中已经用到了分包。<br />
但当代码量多起来后，你可能就比较难找出被错误打进主包的分包内容了。<br />
因此不妨先用简单代码，来梳理清晰打包情况，然后再回头去规范化我们的代码使用。

_本文以 UniApp vue3 举例，用原生代码或其他框架思路是一致的。_

```
- static
- src
  - components
    - HugeData.vue
  - pages
    - main-page
    - sub1
    - sub2
  - App.vue
  - main.js
  - pages.json
  - manifest.json
```

## 1. 及时检查资源使用情况，或资源都放 CDN

假如你往 `/static` 文件夹中放上一张 5M 的图片，恭喜你，主包超 2M 而无法预览和上传了。<br />
因为 UniApp 会将 static 文件夹直接拷贝进 dist，而更核心问题是：**小程序上传并不会检查未使用的资源**。<br />
同理，你在 `/src/sub1` 文件夹中放上超大图片也是一样，分包会超 2M 而无法继续。

_但小程序会排除无法识别的资源，比如 .html .md 或 .ts 等。_

为了避免包体积超限，你需要及时检查资源使用情况，不让未使用的资源占据体积。<br />

不过，资源全部走远程加载我个人觉得是较优解，资源小对视觉的影响不会很大，多端资源还能统一管理。

## 2. 了解公共代码注入进主包的机制，进行规避

假如指定了分包，但公共代码并不在分包内，代码就还在主包中。如下：

```
- src
  - components
  - pages
    - main-page
    - sub1 引用 components
```

只有当公共代码位于分包中，才算是真正打进了分包。如下：

```
- src
  - pages
    - main-page
    - sub1
      - components
```

## 3. 跨分包引用会报错，可以采用软链接方式组织文件

如果你使用的是 UniApp 或 Taro，这样跨分包引用然后打包并不会报错，但到运行时就会报错了。

```
- src
  - pages
    - main-page
    - sub1
      - components
      - modules
    - sub2 引用 sub1/components
```

此时你可以采用组件异步化处理，但公共的 modules 就不即时了，因此你只能拷贝两套代码。

对于这种相同代码的情况，就特别适合 **软链接** 出场，既引用得到又不真正拷贝多份代码。如下；

```
- src
  - components
  - pages
    - main-page
    - sub1
      - components 软链接
    - sub2
      - components 软链接
```

要实现它你只需要一点 node 脚本，然后放在你的运行脚本之前即可。

```js
// 伪代码 scripts/xx.js
const sourceDir = path.resolve(rootDir, 'src/components');
const { subPackages } = require('app.json');

// 在每个分包的文件夹下，加上 components 的软链接
subPackages.forEach(subPackage => {
  const linkDir = path.resolve(rootDir, 'src', subPackage.root, 'components');
  if (!fs.existsSync(linkDir)) {
    fs.symlinkSync(sourceDir, linkDir, 'junction');
  }
});
```

```json
// package.json
{
  "script": {
    "dev": "node scripts/xx.js && uni -p mp-weixin"
  }
}
```

当然，这个方案有点小缺点，但主包体积比分包体积可贵太多，只能牺牲分包各自加上一份相同代码。

## 4. 主包还是大，那就用组件异步化

以上方式，保证了分包代码不会加入到主包中，但主包依旧有超限的可能。<br />
比如 初始化逻辑太多、状态管理 store 函数太多、公共库不拆进分包、入口意图太多 等等。

因此，可以靠组件异步化，来异步一些组件和方法，来使得主包的初次加载量变少，也能逃避超限。

实例如下，让 main-page 引用分包 main-sub 的组件，然后设置 `componentPlaceholder`。

```
- src
  - components
  - pages
    - main-page
    - main-sub
      - components 软链接
    - sub1
      - components 软链接
```

```json
// main-page/index.json
{
  "usingComponents": {
    "async-comps": "/main-sub/components/async-comps"
  },
  "componentPlaceholder": {
    "async-comps": "view"
  }
}
```

## 5. 除了组件异步，模块意图需要一些考量

组件的改动基本只需要调整文件位置和配置即可，但 modules 的异步问题则可能需要一些考量了。<br />
比如，怎样的模块是需要异步、这些模块是否能至于组件中去触发 onLoad 等等，不见得有比较通用的场景。
