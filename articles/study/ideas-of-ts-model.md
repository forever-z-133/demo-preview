# 编写 ts 类型建模的心得分享

## 定义与使用不一致问题

```ts
/** 新建项目-接口入参 **/
interface HttpReqCreateProject {
  // 项目名称
  name: string
  // 项目类型
  type: number
}
```

假设你正在做新增项目的需求，你定义好了接口入参的类型建模，然后开始在业务中使用。  
但是，需求与模型可能有所冲突，比如表单需要初始为空，但接口入参为必填。  
如果把入参 `type?: number` 改为可选，显然也不行，接口入参校验那边就拦不住了。

```ts
type FormData = HttpReqCreateProject
const formData = ref<FormData>({
  name: '',
  type: undefined, // 报错：不能将类型“undefined”分配给类型“number”
})
```

所以你不可避免的需要将接口模型与表单模型区分开：

```ts
// 这样写，新增字段得改多处，修改/删除字段查验不出
type FormData = Omit<HttpReqCreateProject, 'type'> & { type?: number }

// 只能分开得更彻底
interface FormData {
  name: string
  type?: number
}
```

有个骚办法，可以强制转类型，但缺点是不校验初始值类型了，仅适用于初始值必填字段较少的场景，如下：

```ts
const formData = ref({
  name: '',
  type: undefined,
} as unknown as FormData)
```

## 频繁调用的数据判空问题

有的数据需要很多次使用，若定义为可为空，那调用时就需要频繁判空了，但空值其实仅初始时存在：

```ts
const canvas = ref<HTMLCanvasElement>()
console.log(canvas.value?.width) // 每次使用都得判空
```

那么我会强制该数据初始就有值。  
假如实在意外初始化会赋值失败，加个字段去管理状态即可。

```ts
const canvas = ref({} as HTMLCanvasElement)
console.log(canvas.value.width)
```

同理，需要延迟加载的第三方 SDK 也可以这样来初始化。

```ts
let mapIns: TMap.Map
onMounted(() => {
  loadScript('gljs-tools', 'https://sdk').then(() => {
    mapIns new window.TMap.Map('map', {})
  })
})
function handleClick() {
  console.log(mapIns.getZoom())
}
```

## 联动数据对应不同的模型

比如勾选了某项是选择，勾选另一项是输入，类似的需求，对应的建模就会不同，如下：

```ts
type Components = { id: string } & ({ type: 'text', content: string } | { type: 'image', src: string })

// 或者展开来写
interface Base {
  id: string
}
interface Text extends Base {
  type: 'text'
  content: string
}
interface Image extends Base {
  type: 'image'
  src: string
}
type Components = Text | Image
```

这样在你传给联动字段不同的值时，类型会调整为相应的建模。

```ts
const text: Components = { // 报错：类型中缺少属性 "id"，但类型 "Text" 中需要该属性。
  type: 'text',
  src: '', // 报错：对象字面量只能指定已知属性，并且“src”不在类型“Text”中
}
```

这里提一嘴以前犯过的傻事，想把后期赋予的值在定义阶段使用，如下错误的代码：  

```ts
// 无法实现根据第一个参的值来判断第二个参的类型
function createComponent<T extends Components['type']>(type: T, data: { type: T } & Components) {}
```

非要实现上述需求的话，只能在定义阶段靠重载，固定住第一个参的类型。

```ts
class ComponentMethod {
  createComponent(type: 'text', data: Omit<Text, 'type'>): Text
  createComponent(type: 'image', data: Omit<Image, 'type'>): Image
  createComponent(type: any, data: any): any {
    if (type === 'text') return { type, ...data }
    if (type === 'image') return { type, ...data }
    return undefined
  }
}

const methods = new ComponentMethod()
methods.createComponent('text', { id: '', src: '' }) // 报错：类型“"text"”的参数不能赋给类型“"image"”的参数
methods.createComponent('xx', { id: '', src: '' }) // 报错：类型“"xx"”的参数不能赋给类型“"image"”的参数。
methods.createComponent('text', { id: '', content: '' }) // 正常
```

但当然，利用最简单的字段匹配模型，本不该搞这么复杂的 ( •̥́ ˍ •̀ )

```ts
function createComponent(data: Components) {}
createComponent({ type: 'text', id: '', content: '' }) // 正常
```

## 其他小技巧

### 提取列表中特定字段的值

主要靠 `as const satisfies` 这个特性

```ts
interface Option<T = any> { label: string; value: T }
type ArrayElement<A> = A extends (infer T)[] ? T : never

const JOB_STATUS_OPTIONS = [
  { label: '待开始', value: 'TO_START' },
  { label: '发布成功', value: 'SUCCEED' },
  { label: '发布失败', value: 'ERROR' },
] as const satisfies Option[]
type JOB_STATUS = ArrayElement<typeof JOB_STATUS_OPTIONS>['value']

const status: JOB_STATUS // "TO_START" | "SUCCEED" | "ERROR"
```

### 语义不详的空值

```ts
// 其中的 null 是和语义看不出来
type State = null | UserInfo

// 哪怕将 null 赋给语义在显示时也会自动转换为 null
type UnLogin = null
type State = UnLogin | UserInfo // 展示为：type State = UserInfo | null

// 所以，最终方案如下
enum UnLogin {}
type State = typeof UnLogin | UserInfo
const state: State = UnLogin
```
