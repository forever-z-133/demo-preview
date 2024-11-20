# 前端如何做加密

比如前端记住密码、功能配置化、存储用户信息等场景，都需要处理用户不可读不可改的加密数据或文件。

- 1. 文件层加密
  - a. 隐藏文件，易破解
  - b. 改个诡异的文件名，比如 app.png，难发现但易破解
  - c. 压缩成 zip 并设密码，反编译可破解
- 2. 内容层加密，都可暴力破解
  - a. 先 stringify，然后 base64、移位、字节代替等
  - b. aes、rsa 等，若为网页端还需对私钥进一步加密
- 3. 代码层加密，配置打进 js 包
  - a. uglify 丑化 obfuscator 混淆
  - b. electron 代码转字节码，https://github.com/alex8088/electron-vite/blob/master/src/plugins/bytecode.ts 
  - c. webassembly，旧项目难以改造
- 4. 运行层加密，必加
  - a. Object.freeze
  - b. 外发函数，不传出可查看的对象
- 5. 第三方加密
  - a. 前端网关层或后端存私钥，提供加解密接口
  - c. 算法层通过内存地址读取
