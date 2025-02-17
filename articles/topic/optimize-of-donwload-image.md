# 批量下载图片的速率优化

最近发现个不错的写真展示网站，就想爬一下然后批量下载上面的图片。  
记录一下几种方案的实践过程，最后再推荐一种现阶段觉得相当快的批量下载方案。

## 方案一：使用 cURL 命令

一般 NodeJS 原生写下载有些麻烦，还得判断链接的 `protocol` 来决定引入 `node:http` 还是 `node:https` 之类的。

所以想着直接用现成的命令，即 cURL 脚本的。

Windows 也可以 [下载 exe 文件](https://curl.se/windows/) 然后置于 `C:/Windows/System32` 中即可。

```ts
import { execSync } from 'node:child_process'

// 单张下载 - curl 版
async function download(uri: string, output: string) {
  execSync(`curl -s -O ${uri}`, { cwd: output })
}
```

## 方案二：使用 axios 请求

但是呢，体感觉得好像并不快，不禁思考 `execSync` 需要启动子进程，而终究没有原生快。

所以使用 `axios` 这种使用 NodeJS 原生代码实现可能会快一丢丢。

```ts
import path from 'node:path'
import axios from 'axios'
import fs from 'fs-extra'

const ajax = axios.create()

// 单张下载 - axios 版
async function download(uri: string, output: string) {
  const buffer = await ajax.get(uri, { responseType: 'arraybuffer' }).then(res => res.data)
  const targetFile = path.join(output, path.basename(uri))
  fs.writeFileSync(targetFile, buffer, 'utf8')
}
```

结果来看，确实快了 10-30ms，行吧。

### 方案三：并发请求

以上都是单张图片的下载，如果可以同时发起多张下载，也能快一些。

但一次性下载 100 张多半会有问题，一方面是网卡的限制，一方面是文件写入可能错乱(改用 `WriteStream` 也有性能问题)。

所以就需要一个排队策略，比如有空闲就补齐保持同时下载 8 张。

```ts
// 并发运行，设有最大并发量
type ConcurrencyFunction<R = any, A = any> = (arg: A, index: number) => Promise<R>
async function runWithConcurrency<R = any, A = any>(func: ConcurrencyFunction<R, A>, data: A[], maxConcurrent = 8) {
  const results: R[] = []
  const running: Promise<R>[] = []

  for (const i in data) {
    const arg = data[i]
    const index = Number(i)
    const promise = func(arg, index).then((result) => {
      running.splice(running.indexOf(promise), 1)
      results.push(result)
      return result
    })
    running.push(promise)

    if (running.length >= maxConcurrent) {
      await Promise.race(running)
    }
  }

  if (running.length > 0) {
    await Promise.all(running)
  }

  return results
}

// 批量下载
async function downloadMulti(urls: string, output: string) {
  await runWithConcurrency(download, urls, 8)
}
```

大概从 20s 提升到了 12s，是不错的表现，但报错率也莫名变高了。

暂未查出问题，可能来源于网站的拦截，或是函数中报错拦截不充分。

### 方案四：使用无头浏览器多标签

在方案三使用两天后，我发现我手动去浏览器访问然后右键还比我跑代码更快，不禁令人诧异。

于是抱着试一试的态度，我想试试无头浏览器是否真的会更快一些。

```ts
import path from 'node:path'
import fs from 'fs-extra'
import puppeteer from 'puppeteer-core'

let resolveTrigger = (_buffer: any) => {}
let rejectTrigger = (_e: Error) => {}

// 初始化无头浏览器
const browser = await puppeteer.launch({
  executablePath: 'C:/Program Files/Google/Chrome/Application/chrome.exe',
  headless: false,
  defaultViewport: { width: 1185, height: 800 },
  args: ['--no-sandbox', '--disable-setuid-sandbox'],
  timeout: 60000,
})

// 浏览器访问到页面时触发回调，返回资源 buffer
const page = await browser.newPage()
page.on('response', async (res) => {
  const uri = res.url()
  if (!/\.jpe?g|png|webp$/.test(uri)) return
  try {
    const buffer = await res.buffer()
    resolveTrigger(buffer)
  } catch (e: any) {
    console.log(uri, e)
    rejectTrigger(e)
  }
})

// 访问资源，等待浏览器回调，回调完成获得资源 buffer 下载为文件
async function download(uri: string, output: string) {
  return new Promise((resolve, reject) => {
    resolveTrigger = (buffer) => {
      const targetFile = path.join(output, path.basename(uri))
      fs.writeFileSync(targetFile, buffer, 'utf8')
      resolve(undefined)
    }
    rejectTrigger = reject
    page.goto(uri)
  })
}
```

结果来看确实还快了 1-2s，但它就无法直接使用并发函数了，只能给 `page` 标签实例加上对象池策略才能并发。

### 方案五：使用无头浏览器单标签

而脑瓜一转，直接在一个标签实例中渲染多个 `<img />` 节点不也能实现并发嘛。

```ts
function createTempHTML(urls: string[]) {
  const imgsHtml = urls.reduce((re, uri) => `${re}<img src="${uri}" />\n`, '')
  const html = `<!doctype html>
    <html lang="zh-cmn-Hans"><head><meta charset="UTF-8" /></head>
    <body>${imgsHtml}</body>
  </html>`
  return html
}

async function downloadMulti(urls: string[], output: string) {
  const html = createTempHTML(urls) // 拼凑 html 代码
  const tempFile = createTempHTMLFile(html) // 生成 html 文件

  return new Promise((resolve, reject) => {
    resolveTrigger = async (uri, buffer) => {
      const targetFile = path.join(output, path.basename(uri))
      fs.writeFileSync(targetFile, buffer, 'utf8')
      resolve(undefined)
    }
    rejectTrigger = reject
    page.goto(`file://${tempFile}`)
  })
}
```

最终 50 张 600k 的图片全部下载完，从原本的 20s 现在只花了 3.8s，这可真是个令人惊喜的结果。

可能往后如果做截图服务、SSG 生成等，我都会想用无头浏览器来做个方案对比了，说不定也有意外表现。

而浏览器跑并发为何更快的具体原因，还待以后多多研究。
