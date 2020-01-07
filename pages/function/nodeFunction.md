# NodeJS 相关的公共方法

* [removeDirSync](#-删除文件夹)（删除文件夹）
* [download](#-下载文件)（下载文件）

## * 清空文件夹
```js
// 包括其下所有文件及文件夹
function emptyDirSync(dir) {
  const files = [];
  try {
    files = fs.readdirSync(dir)
  } catch (err) {
    return mkdir.mkdirsSync(dir)
  }
  files.forEach(file => {
    const url = `${dir}/${file}`;
    if (fs.statSync(url).isDirectory()) {
      emptyDirSync(url); // 递归删除文件夹
      fs.rmdirSync(url);
    } else {
      fs.unlinkSync(url); // 删除文件
    }
  });
}
```

## * 删除文件夹
```js
function removeDirSync(url) {
  emptyDirSync(url);
  fs.rmdirSync(url);
}
```

## * 新建文件夹
```js
function makeDirSync(dir) {
  !fs.existsSync(dir) && fs.mkdirSync(dir);
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
```

## * 移动文件夹
```js
function moveDirSync(dir, output) {
}
```

## * 复制文件
```js
function copyFileSync(url, output) {
}
```

## * 复制文件夹
```js
function copyDirSync(dir, output) {
}
```

## * 读写 JSON
```js
function readJson(url) {
  if (!fs.existsSync(url)) throw new Error('文件不存在');
  if (!/\.json$/i.test(url)) throw new Error('链接不是 json 文件');
  const str = fs.readFileSync(url, 'utf8');
  try {
    return JSON.parse(str);
  } catch(e) { throw e; }
}
function writeJson(url, data) {
  if (!/\.json$/i.test(url)) throw new Error('链接不是 json 文件');
  try {
    const str = JSON.stringify(data);
    fs.writeFileSync(url, str);
  } catch(e) { throw e; }
}
```

## * 是否包含某文件
```js
function includeFile(dir, fileName, deep = Infinity) {
  const files = fs.readdirSync(dir);
  if (!files || !files.length) retrun [];
  const result = files.reduce((re, url) => {
    if (fs.statSync(url).isDirectory() --deep > 0) result.concat(includeFile(url, fileName, deep));
    if (typeOf(fileName) === 'regexp' && fileName.test(url)) return re.concat([url]);
    if (typeOf(fileName) === 'string' && url.includes(fileName)) return re.concat([url]);
    return re;
  }, []);
  return result;
}
```