# Vue 的逻辑代码复用方案

在代码复用这方面，模块化和组件已经做得不错了，  
但在组件内部其实还有很多逻辑代码可以拆分的，举个例子：

```js
export default {
  name: "search-bar",
  data: () => ({
    // searchData
    // sortData
  }),
  methods: {
    // searchMethods
    // sortMethods
  },
};
```

如果逻辑更多，代码量还不算大问题，模块化出去就行。  
但 `data/methods/computed/props/mounted` 中各包含一部分逻辑，  
使得查看代码，几乎要从最上面找到最下面，这就是不妙的地方了。

这便是本文的主题，将逻辑代码更友好地拆分复用，最好是放到一块去。

## mixin

流传比较多的就是存在争议的 mixins，确实很好的区分了业务，  
但也容易产生冲突，没有清晰的依赖关系，也无法传入数据来更加通用。

```js
const searchMixin = {
  data: () => ({})
  methods: {}
}
const sortMixin = {
  data: () => ({})
  methods: {}
}
export default {
  mixins: [searchMixin, sortMixin]
}
```

## mixin factories

给 mixins 加命名空间，是可以适当弥补的。

```js
// mixins/factories/search.js
export default function searchMixinFactories({}) {}
// mixins/factories/sort.js
export default function sortMixinFactories({}) {}
// index.vue
import searchMixinFactories from "@mixins/factories/search";
import sortMixinFactories from "@mixins/factories/sort";
export default {
  mixins: [
    searchMixinFactories({
      namespace: "indexSearch",
    }),
    sortMixinFactories({
      namespace: "indexSort",
    }),
  ],
};
```

## scoped slots

这个就比较骚操作了，能解决上面的问题，但实属不太灵活，性能应该也堪忧。

```vue
<template>
  <div>
    <slot v-bind="{ searchData }"/>
  </div>
</template>
<script>
// components/generic-search.vue
export default {
  props: [],
  data: () => ({})
}
</script>
```
```vue
<template>
  <GenericSearch :is-index="true" v-slot="{ searchData }">
    <GenericSort :is-index="true" v-slot="{ sortData }">
      <span>查询关键词：{{ searchData }}；并按 {{ sortData }} 排序</span>
    </GenericSort>
  </GenericSearch>
</template>
```

## Composition Api

在 vue3 则提出了 composition function 方案来视图解决上述问题，  
撇开细节只看代码结构，确实是现在最好的逻辑代码拆分方案了。

```js
// use/search.js
export default function useSearch() {}
// use/sort.js
export default function useSort() {}
// index.js
import useSearch from '@use/search';
import useSort from '@use/search';
export default {
  setup() {
    const indexSearch = useSeach({ isIndex: true });
    const indexSort = useSort({ isIndex: true });
    return { indexSearch, indexSort };
  }
}
```
