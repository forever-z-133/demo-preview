# Vue3 性能优化之缓存

## 页面缓存

### v-once
https://cn.vuejs.org/api/built-in-directives.html#v-once

元素/组件仅渲染一次，`v-memo="[]"` 的特殊场景，适用于文章等场景。

```vue
<div v-once v-html="html"></div>
```

### v-memo
https://cn.vuejs.org/api/built-in-directives.html#v-memo

类似 React 的 `memo`，在长列表的场景很适用，或者组件内部有耗时运算则排除部分数据监听。

```vue
<template v-for="item in list" :key="item.id">
  <SomeItem v-memo="[item.id, item.id === selected]" :data="item" :selected="item.id === selected" />
</template>
```

### keep-alive
https://cn.vuejs.org/guide/built-ins/keep-alive.html

适用于 `tab-panel` 切换，或者从详情页返回列表页等场景。

### 异步组件
https://cn.vuejs.org/guide/components/async.html

```vue
<script setup>
import { defineAsyncComponent } from 'vue'
const AdminPage = defineAsyncComponent(() => import('./components/AdminPageComponent.vue'))
</script>

<template>
  <AdminPage />
</template>
```

### 虚拟列表

https://github.com/Akryum/vue-virtual-scroller  
https://github.com/rocwang/vue-virtual-scroll-grid  
https://github.com/07akioni/vueuc  

## 数据缓存

### 运行缓存

使用 [LRUCache](https://www.npmjs.com/package/lru-cache) 算法缓存

```ts
import { LRUCache } from 'lru-cache'
const cache = new LRUCache({
  max: 100, // 保存100个数据
  ttl: 1000 * 60 * 60 * 24, // 保存1天
});
const key = { a: 1 }
cache.set(key, 'a value')
cache.get(key)
cache.clear()
```

### 本地缓存

localStorage 和 IndexDB

<!-- ### 离线缓存 -->
