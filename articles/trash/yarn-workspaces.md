# 用 yarn 的 workspaces 体验多项目管理 

```
npm i -g yarn
yarn -v   // 1.22.17
```

*据说 yarn 2.x 的 workspaces 会有一些不一样，暂不讨论*

## 基础配置

`workspaces-root` 项目中的 `package.json` 需要以下配置：

```json
{
  "private": true,
  "workspaces": [
    "packages/*",
    "components/Editor",
  ],
}
```

后续这些文件夹下的依赖都将归于 `workspaces-root` 的 `node_modules`，只有少量版本差异的包会存在于子仓库中。

```
// 安装在根目录
yarn add lodash -W

// 安装在子目录
yarn workspace one add lodash
```

如果想运行一些脚本同理，如 `yarn workspace one run build`。

## 实际开发

#### 全局配置

比如 EsLint 或者 Prettier 等配置，理论上应该有个全局配置

#### 公共内容引用

想引入 `components/Editor`，子目录中其实没有，在根目录上，需处理此连接关系。

#### 跨工作区引用

#### 子项目版本问题

假如子项目是要发布 npm 线上包的，则大概率需要独立 git 进行版本管理，但都是独立 git 那恐怕又不需要 workspaces 管理了。
