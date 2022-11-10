# 命令式弹窗的实现

像 `Modal.confirm` 是怎样实现的，各框架的实现略有不同，进行一波梳理。


```jsx
// React 的实现方式
const showFontSizeDialog = props => {
  const div = document.createElement('div');
  document.body.appendChild(div);
  // 或者改 props、用 ReactDom.unmountComponentAtNode
  const close = () => document.body.removeChild(div);
  ReactDom.render(<FontSizeDialog {...props} close={close} />, div);
};
```

```js
// Vue2 的实现方式
const dialog = Vue.extend(FontSizeDialog);
const showFontSizeDialog = props => {
  context = new dialog({
    el: document.createElement('div')
  });
  document.body.appendChild(context.$el);
  const close = () => (context.visible = false);
  Vue.nextTick(() => {
    context.close = close;
    context.visible = true;
  });
};
```

```js
// Vue3 的实现方式
import { createVNode, render } from 'vue';
const showFontSizeDialog = props => {
  const div = document.createElement('div');
  document.body.appendChild(div);
  const close = () => (vnode.component!.exposed!.visible.value = false);
  const vnode = createVNode(FontSizeDialog, { ...props, close });
  render(vnode, div);
};
```

```js
// 小程序的实现方式
const showFontSizeDialog = props => {
  const context = (pages = getCurrentPages())[pages.length - 1];
  const dialog = context.selectComponent('#FontSizeDialog');
  dialog.open(props);
};
```
