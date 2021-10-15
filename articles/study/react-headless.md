# React 组件的 Headless 化

https://km.woa.com/group/29048/articles/show/481290?kmref=search&from_page=1&no=1

## 简单示例

```js
const Theme = () => {
  const [theme, setTheme] = useState('light');
  return (
    <div>
      <button onClick={() => setTheme('light')}>亮色主题</button>
      <button onClick={() => setTheme('dark')}>暗色主题</button>
      <span>当前主题为：{theme}</span>
    </div>
  );
}
```

## 增加需求

```js
const Theme = props => {
  const { lightIcon, darkIcon,  showCurrentTheme = true } = props;
  const [theme, setTheme] = useState('light');
  return (
    <div>
      <button onClick={() => setTheme('light')}>
        {lightIcon ? lightIcon : '亮色主题'}
      </button>
      <button onClick={() => setTheme('dark')}>
        {darkIcon ? darkIcon : '暗色主题'}
      </button>
      {showCurrentTheme ? (<span>当前主题为：{theme}</span>) : null}
    </div>
  );
}
```

每出现一个需求，我们都需要修改组件内部的 UI 和行为，  
随着迭代推进，开发者将会变得很难维护和跟踪这些组件。  

## 使用 Headless 组件解决问题

通过 Headless 组件暴露出的逻辑行为，可以实现自己想要的任何操作。  
在 React 中可以使用多种方式实现 Headless 组件（Render Props、HOC、自定义 Hook）

```js
const useTheme = props => {
  const [theme, setTheme] = useState('light');
  const setDark = () => setTheme('dark');
  const setLight = () => setTheme('light');
  return { theme, setDark, setLight };
}
const SetOnlyDarkMode = () => {
  const { setDark } = useTheme();
  return <button onClick={setDark}>暗色主题</button>
}
const SetOnlyLightMode = () => {
  const { setLight } = useTheme();
  return <button onClick={setLight}>亮色主题</button>
}
const ShowCurrentTheme = () => {
  const { theme } = useTheme();
  return <span>当前主题为：{theme}</span>
}
const Theme = props => {
  const { lightIcon, darkIcon,  showCurrentTheme = true } = props;
  return (
    <div>
      <SetOnlyLightMode lightIcon={lightIcon} />
      <SetOnlyDarkMode darkIcon={darkIcon} />
      {showCurrentTheme && <ShowCurrentTheme />}
    </div>
  );
}
```

## 拓展聊聊 Headless UI

诸如 [headless UI](https://headlessui.dev/) 或 [tailwind UI](https://tailwindui.com/)

## 顺便聊聊 Headless Chrome
