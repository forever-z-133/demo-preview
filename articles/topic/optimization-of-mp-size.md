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
  - mock
    - huge-data.js
  - pages
    - main-page
      - index.vue
    - sub1
      - index.vue
    - sub2
      - index.vue
  - App.vue
  - main.js
  - pages.json
  - manifest.json
```

## 1. 及时检查资源使用情况，或资源都放 CDN

假如你往 `/static` 文件夹中放上一张 5M 的图片，恭喜你，主包超 2M 而无法预览和上传了。<br />
因为 UniApp 会将 static 文件夹直接拷贝进 dist，而更核心问题是：小程序上传并不会检查未使用的资源。<br />
同理，你在 `/src/sub1` 文件夹中放上超大图片也是同理，分包会超 2M 而无法继续。

_但小程序会排除无法识别的资源，比如 .html .md 或 .ts 等。_

为了避免包体积超限，你需要及时检查资源使用情况，不让未使用的资源占据体积。<br />

通常来讲会放进小程序内的资源都不会太大，所以走远程加载对视觉的影响不会很大。<br />
资源都放 CDN 我反而觉得是较优解。
