# 记一次底部菜单扩展性优化

如下图的交互，会有很多个底部栏，底部栏之间可能会有相关性。

<img src="https://z3.ax1x.com/2021/10/12/5mytG8.png" style="width:100%;max-width:400px;">

另外还会遇到以下需求：

- 需要支持多主题
- 可权限控制按钮显隐
- 定制化会有不同的样式与交互
- 底部栏的弹起要操作方便

本次优化经验可以应用在大多数状态颇多的场景，<br />
比如 订单/合同/审批 里的按钮，都是至少 5 种状态对应不同的 ui 和事件，面临和本文类似的问题。<br />
希望 “业务下沉” 的思路对你也有所帮助。

## 组件粒度拆分

先介绍下项目前提，最初的底部栏代码如下，通俗易懂，易于增删。

```jsx
const ADD_FOOTER_BUTTONS = [
  { type: 'add-text', icon: '', label: '添加文字' },
  { type: 'add-image', icon: '', label: '添加图片' },
];

function AddFooter() {
  const onClick = item => () => {};
  return (
    <div className="add-footer">
      {ADD_FOOTER_BUTTONS.map(item => (
        <div
          className={classnames('item', item.type)}
          onClick={onClick(item)}
        >
          <img src={item.icon} alt="">
          <p>{item.label}</p>
        </div>
      ))}
    </div>
  );
}
```

然而，你会发现，这样写在行为和样式上都不易于拓展。<br />

- 若部分按钮的 结构与样式 不一样时，对现有结构破坏性很大
- `onClick` 根据 `item.type` 区分，会写得非常庞大
- 部分元素才有 `onTouchStart` 的话，区分起来比较难看

虽然你可以继续补漏，把样式字符串放进配置、把事件放进配置、给事件加工厂 等等，但既然都这样了何不直接做成组件呢。

可见，**复杂的场景下，业务需要下沉，分拆到子组件中去**。

比如经过下面的改造，各按钮间的 结构/样式/事件 都独立开来了，清晰且易于改造。

```jsx
function AddFooter() {
  return (
    <div className="add-footer">
      <AddText />
      <AddImage />
    </div>
  );
}
function AddText(props) {}
function AddImage(props) {}
```

<img src="https://s1.ax1x.com/2022/11/08/xz1rgP.jpg" alt="改造后的文件目录" style="width:100%;max-width:100px">

---

上述所说的“复杂的场景”是很笼统的，还需要你更多的实际体验后可能才有所感悟。 

比如上传图片组件，虽说有 3-5 种上传状态对应不同 UI，但实际并不复杂，可能只需要拆 UI 组件就已足够。

再举个运营组件的例子，比如点击 banner 或落地页按钮，<br />
每种业务有不同的结构与行为，但特例也不算多，可能仅仅多一层壳套就足够了。

<img src="https://z3.ax1x.com/2021/11/12/ID0DG8.png" alt="运营位业务的设置" style="max-width:500px" />

```jsx
// 运营业务公共部分的抽离
function useBusinessButton(initialValue) {
  const { type, jumpUrl, params } = initialValue;
  const onClick = e => {
    switch(type) {} // 事件
  }
  const BusinessButton = props => {
    const { children, ...rest } = props;
    const _props = type === 'share' ? { 'open-type': 'share' } : {};
    switch(type) {} // 样式
    return <button {..._props} {...rest}>{children}</button>
  }
  return [BusinessButton, onClick];
}

// 具体使用
function IndexSwipeItem(props) {
  const { item } = props;
  const [BusinessButton, onClick] = useBusinessButton(item);
  return (
    <BusinessButton onClick={onClick}>
      <span>业务触发按钮</span>
    </BusinessButton>
  );
}
```


再比如 `switch` 行数太多，但其他的不复杂，那用 **模块化** 或 **设计模式** 更好。<br />

## 业务下沉后的数据传递

由于本项目的事件处理分支实在太多了，特例也多，故而采用了业务下沉方案。

然而，当业务下沉后，各子组件间其实就可能要面临数据通信更复杂的问题了。

比如原本所有的逻辑和状态都在 `AddFooter` 这层处理的，但下沉后父级数据就下沉到兄弟层级了，那么兄弟间的数据沟通就得换方式了。<br />
其次，数据从父级向子级传递的流程可能会变得很长，且中间过程的传递可能并无意义，另外过程中的组件的 `prop-types` 也不必要。<br />

因此你极有可能是需要一个上下文状态的，使用 `Context API` 或者 `mobx` 等来轻松处理它。<br />

此外，例如 React 父级的刷新会让子级都渲染一次，还得做好这方面的优化。

## 相同功能的组件

假如 `TextFooter` 和 `AddressFooter` 都有 `FontSize` 子组件，<br />
为了文件目录的一目了然，肯定是都要保留的，但代码却不需两份。<br />

你可以用 `symlink`，即文件的软连接。既引用得到，又不存在两份相同代码。

但它有个问题，软连接上 git 仓库并不美妙，所以你需要写 node 脚本去处理这件事。

## 对应的弹窗管理

编辑器按钮会对应不同的弹窗弹出，比如是字号相关的表单，或是时间选择。<br />
另外，一个按钮可能触发多个弹窗，比如添加日期按钮需要先选日期再选样式模板后再添加进视图。

最初的方案是弹窗与按钮放一起，状态操作都在这一层完成，比较直观。<br />
但在小程序端无法 `appendToBody`，而 `fixed` 布局又容易被层叠上下文影响而失效，是个缺陷。

```jsx
function FontSize(props) {
  return (
    <IconButton label="字号" />
    <FontSizeDialog appendToBody />
  );
}
```

所以弹窗组件只能放至父级，那么子级想打开弹窗就需要调 `setFooterDialogStatus` 等函数去搞状态控制。<br />
也不是不能用，但用多了还是能感受到不美妙：state 结构不直观，props 传递冗长，频繁刷新父级 等。

```jsx
function FontSize(props) {
  const { setFooterDialogStatus, setFooterDialogData } = props;
  const onClick = () => {
    setFooterDialogStatus({ fontSize: true });
    setFooterDialogData({ fontSize: { onConfirm: () => {} } })
  };
  return <IconButton label="字号" onClick={onClick} />;
}

function App() {
  const [footerDialogStatus, setFooterDialogStatus] = useState({ fontSize: false });
  const [footerDialogData, setFooterDialogData] = useState({ fontSize: {} });

  return (
    <Footer
      setFooterDialogStatus={setFooterDialogStatus}
      setFooterDialogData={setFooterDialogData}
    />
    <FontSizeDialog
      visible={footerDialogStatus.fontSize}
      data={footerDialogData.fontSize}
    />
  );
}
```

所以命令式弹窗是比较雅观的方式，让按钮与弹窗有紧密的相关性，也少了状态传递的负面影响。

至于命令式弹窗各端有所不同，React 用 `createPortal()` 或 `render()`, 小程序用 `selectComponent().method` 等，具体可参考 [命令式弹窗的实现](/articles/trash/open-dialog-with-command)。

```jsx
import { openFontSizeDialog } from '/FontSizeDialog/module';
function FontSize(props) {
  const onClick = () => {
    openFontSizeDialog({ onConfirm: () => {} });
  };
  return <IconButton label="字号" onClick={onClick} />;
}
```
