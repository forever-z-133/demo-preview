# 如何做前端权限控制

可能会控制的内容有：

1. 配置（域名/菜单/主题/元素的显隐或禁用）
2. 操作（走不同函数/流程顺序）

在权限数据存储方面，有的是就告诉个职位全凭前端写死逻辑肯定是不适用于长期的大项目的，所以推荐采用 authId 之类的精确标记，再分配到职位上，这样职位的权限即可动态许多。

## 菜单权限

菜单配置可前端写死也可后端返回，权限可后端筛选也可返回前端筛选，这是三条路，你品品。

* 后端存全部配置，后端筛选出当前配置传给前端
* 前端存全部配置，后端传当前配置给前端
* 前端写好配置关系，后端传身份给前端，前端筛选出当前配置

需求应该不难做，数据格式普遍会有两种，优劣嘛我还没遇到太多所以还判断不了。

```js
// 格式一
[{ name: '用户管理', children: [{ name: '用户列表' }] }]

// 格式二
[{ id: 1, name: '用户管理', parentId: null }, { id: 11, name: '用户列表', parentId: 1 }]
```

## 元素权限

若使用的是 vue 框架，自定义指令是真的香。<br />
若使用的是 React 框架，必然高阶组件了，只是嵌套起来略微难看吧。<br />
若使用的是多页面框架，那写个 checkAuth 全局方法每次调用去直接操作 dom 好了。

```js
// Vue 自定义指令，比如 <button v-auth='edit'>
Vue.directive("auth", {
  inserted: function (el, binding, vnode) {
    const optName = binding.arg;
    const authName = `${routeName}-${optName}`;
    const btnAuthList = store.state.auth.btnAuthList;
    if (btnAuthList[authName] === 1) { // 1 删除 2 禁用 0 正常
      el.parentNode.removeChild(el);
    } else if (btnAuthList[authName] === 2) {
      vnode.componentInstance.disabled = true;
    }
  },
});
```

## 路由权限

当职位不同，可能会有不同的界面和交互流程，在 beforeEach 生命周期进行拦截然后处理（React 可用 onEnter）。

比如销售改优惠会多一步审核再完成，禁掉了菜单但没禁路由，下级看到的是预览上级才可编辑，等等。

## 系统基础配置

比如同个软件用于多个项目，各项目的域名/项目名/主题/语言等可能不尽相同。

若这些也是动态的就比较考验架构的稳定了，比如先后顺序与首屏渲染等问题。

## 流程权限

路由权限也有流程权限的一部分，而函数的运行顺序或分支也可做流程权限，而这最好要带上工厂模式等设计模式了。
