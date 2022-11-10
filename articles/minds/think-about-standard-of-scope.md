# 代码修改的影响范围的思考

先列个场景，方便开始本篇的话题。

```jsx
const TemplateList = props => {
  const {
    tplType: 1, // 1 为官方模板 2 为自有模板
  } = props;

  return <></>
};
```

假如这是你写的组件，为了代码复用你提供了一个类型的入参，在不同地方被使用并传入不同的入参。

然而，产品来了需求，支持某机构特别的场景，该机构完全不显示官网模板。此时你有两种修改方案：

* 在组件的调用处修改入参，保持本组件的纯粹性

```jsx
const App = () => {
  return <TemplateList tplType={xxx ? 1 : 2}>
};
```

* 在组件的内部搞个新变量，统一直接处理该场景

```jsx
const TemplateList = props => {
  const { tplType: 1 } = props;
  const realTplType = xxx ? tplType : 2;
  return <></>
};
```

这个命题可大可小，可以说是开发习惯也可以说是编程哲学，最近我刚好也有了点想法。

类似的场景还有，vue template 中的判断越写越长是否应该统一使用 computed
