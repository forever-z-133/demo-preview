# NodeJS 相关的公共方法

* [removeDirSync](#-删除文件夹)（删除文件夹）
* [download](#-下载文件)（下载文件）

## * 删除文件夹
```js
// 包括其下所有文件及文件夹
function removeDirSync(path) {
  let files = [];
  if (fs.existsSync(path)) {
    files = fs.readdirSync(path);
    files.forEach(file => {
      let curPath = path + "/" + file;
      if (fs.statSync(curPath).isDirectory()) {
        delDir(curPath); //递归删除文件夹
      } else {
        fs.unlinkSync(curPath); //删除文件
      }
    });
    fs.rmdirSync(path);
  }
}
```

## * 下载文件
```js
function download(url, output, fileName) {
  return new Promise(resolve => {
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
        resolve(filePath);
      });
    });
  });
}
```