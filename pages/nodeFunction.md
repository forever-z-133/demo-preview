# NodeJS 相关的公共方法

* [removeDirSync](#-删除文件夹)（删除文件夹）
* [download](#-下载文件)（下载文件）

## * 清空文件夹
```js
// 包括其下所有文件及文件夹
function emptyDirSync(path) {
  const files = [];
  try {
    files = fs.readdirSync(dir)
  } catch (err) {
    return mkdir.mkdirsSync(dir)
  }
  files.forEach(file => {
    let curPath = path + "/" + file;
    if (fs.statSync(curPath).isDirectory()) {
      emptyDirSync(curPath); // 递归删除文件夹
      fs.rmdirSync(curPath);
    } else {
      fs.unlinkSync(curPath); // 删除文件
    }
  });
}
```

## * 删除文件夹
```js
function removeDirSync(path) {
  emptyDirSync(path);
  fs.rmdirSync(path);
}
```

## * 新建文件夹
```js
function makeDirSync(path) {
  !fs.existsSync(path) && fs.mkdirSync(path);
}
```

## * 下载文件
```js
function download(url, output, fileName, callback) {
  fileName = fileName || url.slice(url.lastIndexOf('/') + 1);
  if (!fileName) throw new Error('没找到文件名');
  const filePath = path.join(output, fileName);
  const stream = fs.createWriteStream(filePath);
  https.get(url, (res) => {
    res.on('data', (chunk) => {
      stream.write(chunk);
    });
    res.on('end', () => {
      stream.end();
      callback && callback(filePath);
    });
  });
}
async function downloadSync(url, output, fileName) {
  return await new Promise(resolve => {
    return download(url, output, fileName, resolve);
  });
}
```

## * 移动文件夹
```js
function moveDirSync() {
}
```

## * 复制文件
```js
function copyFileSync() {
}
```

## * 复制文件夹
```js
function copyDirSync() {
}
```

## * 读取 JSON
```js
function readJson() {
}
async function readJsonSync() {
}
```