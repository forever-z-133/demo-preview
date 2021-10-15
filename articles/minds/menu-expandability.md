# 记一次底部菜单扩展性优化

如下图的交互，会有很多个底部栏，底部栏之间可能会有相关性。

<img src="https://z3.ax1x.com/2021/10/12/5mytG8.png" style="width:100%;max-width:400px;">

另外还会遇到以下需求：

- 需要支持多主题
- 可权限控制按钮显隐
- 对应底部栏的弹起要操作方便
- 定制化会有不同的交互

## UI 组件粒度划分

通常来讲，`<Item>` 类型的元素都是使用同一个组件。

```jsx
const MAIN_FOOTER_BUTTONS = [
  { type: 'add-text', icon: '', label: '添加文字' },
  { type: 'add-image', icon: '', label: '添加图片' },
];
function AddFooter() {
  const onClick = () => {};
  return (
    <div className="add-footer">
      {MAIN_FOOTER_BUTTONS.map(item => (
        <div className={classnames('item', item.type)} onClick={onClick}>
          <img src={item.icon} alt="">
          <p>{item.label}</p>
        </div>
      ))}
    </div>
  );
}
```

然后你会发现，这样写的话 `onClick` 根据 `type` 区分会写得非常庞大，若还有部分元素才有 `doudleClick` 区分起来也麻烦。  
可见这种配置式的渲染方式并不适用于复杂的事件处理，故而应当拆分独立的组件比较好。

```jsx
function AddFooter() {
  return (
    <div className="add-footer">
      <AddText className="item" />
      <AddImage className="item" />
    </div>
  );
}
function AddText(props) {
  const { className, ...restProps } = props;
  const onClick = () => {};
  return (
    <div className={classnames('add-text', className)} onClick={onClick} {...restProps}>
      <img src="" alt="">
      <p>添加文字</p>
    </div>
  );
}
```

这时候 `AddFooter` 就相当纯粹了，重心转向维护子组件的逻辑处理上。

**总结：回调过于庞大可以将业务下沉**

## 业务下沉后数据传递

## 相同功能的组件

提出这样个问题，在文本编辑中会有字号表单组件，在个人信息编辑中也有字号表单组件，那么这个组件要合并吗？

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
