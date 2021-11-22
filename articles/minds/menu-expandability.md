# 记一次底部菜单扩展性优化

如下图的交互，会有很多个底部栏，底部栏之间可能会有相关性。

<img src="https://z3.ax1x.com/2021/10/12/5mytG8.png" style="width:100%;max-width:400px;">

另外还会遇到以下需求：

- 需要支持多主题
- 可权限控制按钮显隐
- 定制化会有不同的样式与交互
- 底部栏的弹起要操作方便

## 组件粒度拆分

最初的底部栏代码如下，通俗易懂，易于增删。

```jsx
const ADD_FOOTER_BUTTONS = [
  { type: 'add-text', icon: '', label: '添加文字' },
  { type: 'add-image', icon: '', label: '添加图片' },
];

function AddFooter() {
  const onClick = () => {};
  return (
    <div className="add-footer">
      {ADD_FOOTER_BUTTONS.map(item => (
        <div className={classnames('item', item.type)} onClick={onClick}>
          <img src={item.icon} alt="">
          <p>{item.label}</p>
        </div>
      ))}
    </div>
  );
}
```

然而，你会发现，这样写在行为和样式上都不易于拓展。  

- 部分按钮结构与样式不一样时破坏性很大
- 若 `onClick` 根据 `item.type` 区分，会写得非常庞大
- 部分元素才有 `onTouchStart` 的话，区分起来比较难看

虽然你可以继续补漏，把样式作为组件放进配置、把事件放进配置、给事件加工厂 等等，但既然都这样了何不直接做成组件呢。  

可见，**复杂的场景下，业务需要下沉，分拆到子组件中去**。

比如经过下面的改造，各按钮间的事件与样式就独立开来了，清晰且纯粹。

```jsx
function AddFooter() {
  return (
    <div className="add-footer">
      <AddText className="item" />
      <AddImage className="item" />
    </div>
  );
}
function AddText(props) {}
function AddImage(props) {}
```

此处所说的“复杂”是很笼统的，还需要你更多的实际体验后可能才有所感悟。 

举个例子，假设你已做了拆分，但业务并不复杂就两个跳页按钮，  
在想修改时，你的第一反应如果不是去找那个子模块，那说明其内容还不够复杂和丰富到需要下沉。  

<img src="https://z3.ax1x.com/2021/11/22/IzukZj.png" alt="业务不复杂还是先不要下沉" style="max-width:300px" />

再举个运营位业务拓展的例子，比如点击 banner 会进行不同的事件。  
每种业务也有不同的样式与行为，但由于样式和行为的特例其实并不多，那就不必拆分得很散，可能仅仅多一层壳套就足够了。

<img src="https://z3.ax1x.com/2021/11/12/ID0DG8.png" alt="运营位业务的设置" style="max-width:500px" />

```jsx
// 运营位业务公共部分的抽离
function useBusinessButton(initialValue) {
  const { type, jumpUrl, params } = initialValue;
  const onClick = e => {
    switch(type) {} // 事件
  }
  const BusinessButton = props => {
    const { children, ...rest } = props;
    const _props = {};
    switch(type) {} // 样式
    return <button {..._props} {...rest}>{children}</button>
  }
  return [BusinessButton, onClick];
}

// 具体使用
function IndexSwiperItem(props) {
  const { item } = props;
  const [BusinessButton, onClick] = useBusinessButton(item);
  return (
    <BusinessButton onClick={onClick}>
      <span>业务触发按钮</span>
    </BusinessButton>
  );
}
```

可见，**特例化较少的场景下，不要下沉，可用自定义 hooks 来拆分**。

当 `switch` 行数太多，但又没复杂到需要业务下沉的程度，那用 **模块化** 或 **设计模式** 也是阔以的。  

至于 **UI 组件** 通常都不是优化的核心，也更侧重 UI 而非事件处理，  
能节省代码优化结构，但不见得能很大优化开发体验，且这个提炼过程还挺需要开发水平的。

其实这段写得很纠结，分不清业务与架构的界限，也分不清业务集中或分散的优劣，感觉存在某个临界点才需要做优化，否则是不需要的。

## 业务下沉后数据传递

由于本项目的事件处理分支实在太多了，特例也多，故而采用了业务下沉方案。

事实上，当业务下沉后，各子组件间其实是可能面临数据通信更复杂的问题的。  
例如原本所有的逻辑和状态都在 `AddFooter` 这层互相采用处理的，但下沉后 `AddFooter` 其实就不做事了。  
其次，数据从外向内的流程可能很长，且中间过程的传递可能仅仅就是传递，无论是传递还是 `prop-types` 都太冗余了。  

因此你极有可能是需要一个上下文状态的，使用 `Context API` 或者 `mobx` 等都是阔以的，当然都不用也是阔以的。  

而更麻烦的一件事在于，

## 相同功能的组件

有这样个问题，在文本编辑中会有字号表单组件，在个人信息编辑中也有字号表单组件，那么这个组件要合并吗？

```jsx
function TextEditFooter() {
  return (
    <div className="text-edit-footer">
      <TextContent />
      <FontSize />
      <FontColor />
      <FontAlign />
    </div>
  );
}
function UserinfoEditFooter() {
  return (
    <div className="userinfo-edit-footer">
      <UserinfoContent />
      <FontSize />
      <FontColor />
    </div>
  );
}
```

## 定制化的配置结构

## 底部栏弹起管理

```js
[
  {
    type: 'text',
    children: [
      { type: 'font-size', panel: FontSize },
      { type: 'font-family', panel: FontFamily },
      { type: 'font-style', panel: FontStyle },
    ]
  },
  {
    type: 'image',
    children: [
      { type: 'image-replace', panel: ImageReplace },
      { type: 'image-crop', panel: ImageCrop },
    ]
  }
]
```

优点是一目了然且易做权限，但不适合大型项目和个性化拓展。  
明显能感觉到，这个配置会越来越复杂而变得难以维护。  

## 思路二：命令式弹窗

```js
const res = await showFooterModal('font-size', { defaultValue: '' });
```

优点是模块绝对独立，增删模块或权限配置都很容易，要个人化可再起一个新模块。  
缺点是弹窗的状态管理复杂，比如 A 模块想打开它，而 B 模块又想关闭它，既难共享数据也难操作。

## 思路三：状态统一控制

```js
state = {
  footerModalStatus: {
    fontSize: false,
    fontFamily: false,
    fontStyle: { defaultValue: '' },
  }
};
```

所有子组件需调用祖父组件的 `setFooterModalStatus` 来让弹窗显隐。
