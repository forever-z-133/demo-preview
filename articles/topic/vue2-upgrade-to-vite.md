# 记一次 vue2 项目升级到至 vite-vue3-ts 的过程

## 升级 vite

* `npm` 安装 `vite` `typescript` `@vitejs/plugin-vue`
* `packages.json` 中调整 `"type": "module"` 和 `scripts` 脚本
* 新建 `vite.config.ts`
  * `plugins` 加上 `@vitejs/plugin-vue`
  * `define` 加上 `process.env` 环境变量注入
  * `resolve.alias` 加上比如 `@/`
  * `resolve.extensions` 兼容，使得 `import ./App` 可缺失 `.vue`
  * `server.port` 保持 `8088`
  * `server.proxy` 使用新写法
  * `svg` 文件使用 `vite-svg-loader` 加入 `plugins`
  * `fbx` 文件使用 `assetsInclude: ['**/*.fbx']`
* 业务组件 `Uploader` 中使用了 `jsx`，暂用 `el-upload` 代替
* 多入口，配置 `build.rollupOptions.input`
* 打包结果的目录结构与旧项目一致，修改 `build.rollupOptions.output`
* 所有 `require.context` 改为 `import.meta.glob`

## 升级 vue3

* 调整 `router`、`vuex`、`i18n` 的导出与注册写法
* 去掉所有组件内 `filters`，转为 `methods` 写法
* 全局 `filters`，转为组件内引入
* 去掉所有的 `import Vue from 'vue'`
* 所有 `Vue.prototype` 改为 `app.config.globalProperties`
* `template[v-for]` 的 `key` 只能在 `template` 上
* 把 `slot="title"` 改为 `#title`，且 `#title` 只能在 `template` 上
* 把 `slot-scope` 改为 `#default` 等，且 `#default="scope"` 只能在 `template` 上
* 去掉全部 `v-on="$listeners"`
* `.native` 修饰符可能效果已消失，暂无方案
* 把 `beforeDestroy` 改为 `beforeUnmount`
* 把 `destroyed` 改为 `unmounted`
* 把 `mixin` 中的 `beforeRouteEnter` 改为 `mounted`
* 全局 `$set` 失效，尽量去除，临时用 `x[y] = [z]` 代替
* 全局 `$on`、`$emit`、`$off` 失效，自己实现 `eventBus` 注入到全局
* 把 `$createElement` 改为 `import { h } from 'vue'`，以及其组件实例相关问题
* `router.beforeEach` 中 `return` 和 `next` 只能有一种

## 升级 element-plus

* 去掉 `element-ui`，更改其引用，包含初始化、样式地址、`i18n`
* 调整部分组件引用，比如把 `Message` 改为 `ElMessage`
* 部分组件名称变动，比如把 `el-submenu` 改 `el-sub-menu`
* 把 `el-button[type=text]` 改为 `el-button[link]`
* 把 `el-dialog[custom-class]` 改为 `el-dialog[class]`
* 把 `[value]` 改为 `[model-value]`
* 去掉所有 `[size=middle]`，把 `[size=mini]` 改为 `[size=small]`
* 把 `[class="el-icon-warning"]` 改为 `<warning />`
* 把 `el-dialog[visible]` 改为 `el-dialog[model-value]`
* 把 `el-radio-group[@input]` 改为 `el-radio-group[@change]`
* 把 `el-popover[v-model]` 改为 `el-popover[v-model:visible]`
* 把 `el-popover[value]` 改为 `el-popover[visible]`
* 把 `el-popover ref` 的 `doClose` 改为 `hide`
* `el-upload ref` 的 `uploadFiles` 失效，改为使用 `@onSuccess` 返回的 `files` 入参
* `el-select ref` 的 `wrap dom` 改为使用 `.scrollbarRef.wrapRef`

## 其他改造
* 较多组件有问题，`$emit('input')` 改为 `$emit('update:modelValue')`
* `eslint` 问题，更新最新写法
* 把所有 `/deep/` 改为 `:deep()`
* 把 `vuex` 改为 `pinia`
