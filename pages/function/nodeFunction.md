# NodeJS 相关的公共方法

- [forEachDir](#-遍历所有文件)（遍历所有文件）
- [emptyDirSync](#-清空文件夹)（清空文件夹）
- [removeDirSync](#-删除文件夹)（删除文件夹）
- [makeDirSync](#-新建文件夹)（新建文件夹）
- [getFileName](#-获取文件名)（获取文件名）
- [getFileExtName](#-获取文件后缀)（获取文件后缀）
- [getFileSizeString](#-返回文件体积字符串)（返回文件体积字符串）
- [ajax](#-发起请求)（发起请求）
- [downloadFile](#-下载文件)（下载文件）
- [readJson / writeJson](#-读写-JSON)（读写 JSON）
- [includeFile](#-是否包含某文件)（是否包含某文件）

## 遍历所有文件

```js
import glob from 'fast-glob';

const files = glob.sync('node_modules/**/*.json', {});
console.log(files);
```

## \* 清空文件夹

```js
import fs from 'fs-extra';

fs.emptyDirSync('trash');
```

## \* 删除文件夹

```js
import fs from 'fs-extra';

fs.removeSync('trash');
```

## \* 新建文件夹

```js
import fs from 'fs-extra';

fs.ensureDirSync('dir');
```

## \* 获取文件名

```js
import path from 'node:path';

path.basename(filePath)
```

## \* 获取文件后缀名

```js
import path from 'node:path';

path.extname(filePath)
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

## \* 发起请求

```js
import axiosBase from 'axios'

const axios = axiosBase.create({});

axios.get('/api/get', { params: { id: 1 } });

axios.post('/api/post', { id: 1 });

const formData = new FormData();
formData.append('file', null);
formData.append('files[]', null);
axios.post('/api/post', formData, { headers: { "Content-Type": "multipart/form-data" }, });
```

## \* 下载文件

```js
import { execSync } from 'node:child_process'

execSync('curl https://raw.githubusercontent.com/forever-z-133/learn-node/refs/heads/master/test/youtube/index.mjs >> output.mjs')
```

## \* 读写 JSON

```js
import fs from 'fs-extra';

fs.readJSONSync('package.json')
fs.writeJSONSync('package.json')
```

## \* 是否包含某文件

```js
import glob from 'fast-glob';

const files = glob.sync('node_modules/**/async.d.ts', {});
const included = files.length > 0;
console.log(included);
```
