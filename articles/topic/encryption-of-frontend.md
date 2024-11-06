# 前端如何做加密

比如前端记住密码、功能配置化、存储用户信息等场景，都需要处理用户不可读不可改的加密数据或文件。

- 1. 文件层加密
  - a. 隐藏文件，易破解
  - b. 改个诡异的文件名，比如 app.png，难发现但易破解
  - c. 压缩成 zip 并设密码，反编译可破解
- 2. 内容层加密，内容乱码化，都可暴力破解
  - a. stringify 后转数字或 base64
  - b. crypto 相关，比如 rsa 等，其他公私钥或对称等策略
- 3. 代码层加密，配置打进 js 包
  - a. uglify 丑化 obfuscator 混淆
  - b. electron 代码转字节码，https://github.com/alex8088/electron-vite/blob/master/src/plugins/bytecode.ts 
  - c. webassembly
- 4. 运行层加密，必加
  - a. Object.freeze（传出对象方便查验）
  - b. 外发函数，不传出可查看的对象
- 5. 第三方加密
  - a. 前端网关层加密
  - b. 后端加密
  - c. 算法层通过内存地址读取
