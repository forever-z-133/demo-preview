# 怎么写组件才不会被喷？

## SRP 原则

每一个类都应该只做一件事。

```jsx
function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) return 'Loading...';
  return isOnline ? 'Online' : 'Offline';
}
```

```jsx
// 改造后
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) return 'Loading...';
  return isOnline ? 'Online' : 'Offline';
}
```

## OCP 原则

软件实体应该对扩展开放，对修改关闭。

```jsx
function Header() {
  const { pathname } = useRouter();

  return (
    <header>
      <Logo />
      <Actions>
        {pathname === '/dashboard' && <Link to="/events/new">Create event</Link>}
        {pathname === '/' && <Link to="/dashboard">Go to dashboard</Link>}
      </Actions>
    </header>
  )
}
function HomePage() {
  return <Header />
}
function DashboardPage() {
  return <Header />
}
```

```jsx
// 改造后
function Header(props) {
  return (
    <header>
      <Logo />
      <Actions>{props.children}</Actions>
    </header>
  );
}
function HomePage() {
  return <Header><Link to="/dashboard">Go to dashboard</Link></Header>
}
function DashboardPage() {
  return <Header><Link to="/events/new">Create event</Link></Header>
}
```

## LSP 原则

子类对象应可以替代基类对象。

```jsx
type Props = { value: string; onChange: (event?: any) => void }

function CustomInput({ value, onChange }: Props) {
  return <input value={value} onChange={onChange} />
}
```

```jsx
// 改造后
type Props = InputHTMLAttributes<HTMLInputElement> & { onUpdate: () => void }

const CustomInput = (props: Props) => {
  return <input {...props} />
}
```

```jsx
// 或改造为
type Props = InputHTMLAttributes<HTMLInputElement>

const CustomInput = ({ onChange, ...props }: Props) => {
  const handleChange = event => {
    onChange && onChange(event);
  };
  return <input {...props} onChange={handleChange} />
}
```

## ISP 原则

不应依赖不使用的接口。

```jsx
type Video = { id: string, coverUrl: string }
type Props = { video: Video }

function Thumbnail({ video }: Props) {
  return <img src={video.coverUrl} />
}
```

```jsx
// 改造后
type Props = { coverUrl: string }

function Thumbnail({ coverUrl }: Props) {
  return <img src={coverUrl} />
}
```

## DIP 原则

依赖抽象而非具体；低层设计依赖高层设计。

```jsx
function LoginForm() {
  const handleSubmit = async () => {
    await api.login();
  }
  return <form><Button onClick={handleSubmit}></Button></form>
}
```

```jsx
// 改造后
type Props = { onSubmit: () => void }

function LoginForm({ onSubmit }: Props) {
  const handleSubmit = () => {
    onSubmit && onSubmit();
  }
  return <form><Button onClick={handleSubmit}></Button></form>
}

const ConnectedLoginForm = () => {
  const handleSubmit = async () => {
    await api.login();
  }
  return (
    <LoginForm onSubmit={handleSubmit} />
  )
}
```
