# NodeJS 相关的公共方法

- [forEachDir](#-遍历所有文件)（遍历所有文件）
- [emptyDirSync](#-清空文件夹)（清空文件夹）
- [removeDirSync](#-删除文件夹)（删除文件夹）
- [makeDirSync](#-新建文件夹)（新建文件夹）
- [getFileName](#-获取文件名)（获取文件名）
- [getFileExtName](#-获取文件后缀)（获取文件后缀）
- [getFileSizeString](#-返回文件体积字符串)（返回文件体积字符串）
- [getUrlNetType](#-获取文件网络类型)（获取文件网络类型）
- [ajax](#-发起请求)（发起请求）
- [downloadFile](#-下载文件)（下载文件）
- [readUrl](#-读取文件)（读取文件）
- [moveDirSync](#-移动文件夹)（移动文件夹）
- [copyDirSync](#-复制文件夹)（复制文件夹）
- [readJson / writeJson](#-读写-JSON)（读写 JSON）
- [includeFile](#-是否包含某文件)（是否包含某文件）

## 遍历所有文件

```js
function getFilesInDirSync(dir) {
  let files = [];
  try {
    files = fs.readdirSync(dir);
  } catch (err) {}
  return files;
}
function forEachDir(dir, dirCallback, fileCallback, depth = 99) {
  if (depth < 1) return;
  let files = getFilesInDirSync(dir);
  files.forEach((file) => {
    let _path = path.join(dir, file);
    if (fs.statSync(_path).isDirectory()) {
      // 如果自己是文件夹，则先继续递归，然后再处理自身
      forEachDir(_path, dirCallback, fileCallback, depth - 1);
      dirCallback && dirCallback(_path);
    } else {
      fileCallback && fileCallback(_path);
    }
  });
  depth--;
}
```

## \* 清空文件夹

```js
// 包括其下所有文件及文件夹
function emptyDirSync(dir) {
  forEachDir(
    dir,
    (_dir) => {
      fs.rmdirSync(_dir); // 递归删除文件夹
    },
    (_file) => {
      fs.unlinkSync(_file); // 删除文件
    }
  );
}
```

## \* 删除文件夹

```js
function removeDirSync(url) {
  emptyDirSync(url);
  fs.rmdirSync(url);
}
```

## \* 新建文件夹

```js
function makeDirSync(dir) {
  !fs.existsSync(dir) && fs.mkdirSync(dir);
}
```

## \* 获取文件名

```js
// 获取 x.js 或 /x.js 中的 x.js
// 但不区分 "/x" "x" ".." 等情况哈
function getFileName(filePath) {
  filePath = filePath.replace(/[?#].*/, '');
  return filePath.split(/[\/\\]/).slice(-1)[0];
}
```

## \* 获取文件后缀名

```js
// 获取 x.js 中的 .js
function getFileExtName(url) {
  const match = url.match(/\.[^\.\/]+$/);
  return match ? match[0] : '';
}
```

# \* 返回文件体积字符串

```js
const FILE_SIZE_UNIT = ['b', 'kb', 'M', 'G', 'T'];
const getFileSizeString = size => {
  let index = 0;
  let num = size / (1024 ** index);
  let result = '';
  while (num >= 1) {
    result = `${toFixed(num, 3)}${FILE_SIZE_UNIT[index]}`;
    index += 1;
    num = size / (1024 ** index);
  }
  return result;
};
```

## \* 获取文件网络类型

```js
function getUrlNetType(url) {
  if (/^https/.test(url)) return 'https';
  if (/^http/.test(url)) return 'http';
  return 'local';
}
```

## \* 发起请求

```js
const axios = require('axios');
async function ajax(url, method, data, options = {}) {
  if (/^get$/i.test(method)) {
    return axios.get(url, { params: data }, { ...options });
  } else if (/^post$/i.test(method)) {
    return axios.post(url, data, { ...options });
  } else if (/^form-?data$/i.test(method)) {
    const fd = new FormData();
    for (let key in data) fd.append(key, data[key]);
    return axios.post(url, fd, { ...options, 'Content-Type': 'multipart/form-data' });
  }
  return axios({ url, method, ...options });
}
```

## \* 下载文件

```js
function ajax(url, method = 'get', callback) {
  const netType = fileNetType(url);
  const ajax = netType === 'https' ? require('https') : require('http');
  return ajax[method](url, callback);
}
function downloadFile(url, output, fileName, callback) {
  if (typeof fileName === 'function') {
    callback = fileName;
    fileName = null;
  }
  fileName = fileName || getFileName(url);
  if (!fileName) throw new Error(' 没找到文件名');
  let filePath = path.join(output);
  if (filePath.lastIndexOf(fileName) < 0) {
    filePath = path.join(output, fileName);
  }
  const stream = fs.createWriteStream(filePath);
  ajax(url, 'get', (res) => {
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

## \* 读取文件

```js
// url 可为本地，可为网络
function readUrl(url, callback) {
  const netType = getFileNetType(url);
  if (netType === 'https' || netType === 'http') {
    ajax = netType === 'https' ? require('https') : require('http');
    ajax.get(url, res => {
      let body = '';
      res.on('data', chunk => (body += chunk));
      res.on('end', () => {
        const result = body.toString('utf8');
        callback && callback(result);
      });
    });
  } else {
    const result = fs.readFileSync(url).toString('utf8');
    callback && callback(result);
  }
}
```

## \* 移动文件夹

```js
function moveDirSync(dir, output) {}
```

## \* 复制文件夹

```js
function copyDirSync(dir, output) {}
```

## \* 读写 JSON

```js
function readJson(url) {
  if (!fs.existsSync(url)) throw new Error('文件不存在');
  if (!/\.json$/i.test(url)) throw new Error('链接不是 json 文件');
  const str = fs.readFileSync(url, 'utf8');
  try {
    return JSON.parse(str);
  } catch (e) {
    throw e;
  }
}
function writeJson(url, data) {
  if (!/\.json$/i.test(url)) throw new Error('链接不是 json 文件');
  try {
    const str = JSON.stringify(data, '', 4);
    fs.writeFileSync(url, str);
  } catch (e) {
    throw e;
  }
}
```

## \* 是否包含某文件

```js
function includeFile(dir, fileName, deep = Infinity) {
  const files = fs.readdirSync(dir);
  if (!files || !files.length) retrun [];
  const result = files.reduce((re, url) => {
    if (fs.statSync(url).isDirectory() && --deep > 0) result.concat(includeFile(url, fileName, deep));
    else if (typeOf(fileName) === 'regexp' && fileName.test(url)) return re.concat([url]);
    else if (typeOf(fileName) === 'string' && url.includes(fileName)) return re.concat([url]);
    return re;
  }, []);
  return result;
}
```
