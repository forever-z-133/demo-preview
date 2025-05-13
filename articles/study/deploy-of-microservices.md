# 记一次单仓库微服务部署

## 前言

现在蛮喜欢 `monorepo` 这样的仓库管理方式，单仓库中管理所有。我的仓库结构如下：

```
- build（打包相关）
  - Dockerfile
  - nginx.conf
- packages
  - desktop（桌面应用端代码）
    - package.json
  - eslint-plugin（eslint 相关拓展，比如自定义代码格式等）
    - package.json
  - front-api（微服务，比如带密钥的请求、服务端渲染等）
    - package.json
  - unit-test（单元测试）
    - package.json
  - vite-plugin（vite 相关扩展，比如新建时生成代码、自定义分包等）
    - package.json
- src（网页端代码）
  - App.vue
  - main.ts
- index.html
- package.json
- vite.config.ts
```

以往部署只有 `/src` 网页端这一个端口，现在想加个 `koa` 前端微服务即 `/front-api` 将会有多个端口。

本文将探索如何在单仓库中管理代码和构建 `docker` 容器部署。

## 多个镜像方案

最简单的做法，就是将 `/front-api` 单独打个 `docker` 镜像。

```dockerfile
# 网页端 Dockerfile-web
FROM nginx
RUN cp -r build/nginx.conf ./dist
WORKDIR /usr/share/nginx/html
COPY ./dist .
RUN mv -f nginx.conf /etc/nginx
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```

```dockerfile
# 微服务 Dockerfile-node
FROM node:20.18.0-alpine
WORKDIR /usr/share/nginx/html
COPY ./dist .
CMD ["node", "server.cjs;"]
```

两个 `Dockerfile`，`docker` 相关操作如 `build`/`push`/`pull`/`run` 也是两次。

好处当然也有，就是两者部署也变得独立，可以单独仅部署微服务而不影响网页端。但前端微服务向来比较简单快速，这点好处还是不够诱人。

社区中倒是有个 `docker compose` 来简化 `docker` 操作，让 **部分** 命令仅需一次。

```bash
# docker 命令
apt-get install docker-compose-plugin
docker compose up # 构建并启动容器
docker compose build # 构建镜像，但后置启动就得 pull 和 run 多个镜像了

# podman 命令
brew install podman-compose # https://github.com/containers/podman-compose
podman-compose build
```

具体实现是需要 `docker-compose.yml` 文件。

```yml
# docker-compose.yml
version: '3'

services:
  node:
    build: ./packages/front-api # Dockerfile 所在目录
    restart: unless-stopped

  web:
    build: ./
    restart: unless-stopped
    depends_on:
      - node
```

当然，你也可以按 `compose.yml` 语法把 `Dockerfile` 的 `FROM nginx` 改为 `docker-compose.yml` 的 `image: nginx` 等，  
就不用建 `Dockerfile` 文件了，进一步减少配置所需关注的文件。

## 单个镜像方案

构建出来多个镜像还是管理起来太麻烦，可能有的部署平台定制程度不高还只能 `pull` 一个镜像那就哦豁了。

因此，也可以使用 `supervisord` 管理多进程，仅构建单个镜像。

```ini
# supervisord.conf
[supervisord]
nodaemon=true

[program:node]
command=node server.cjs
autostart=true
autorestart=true
environment=PORT=4002

[program:web]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true
```

```dockerfile
# 总 Dockerfile
FROM node:20.18.0-alpine

RUN apk add nginx supervisor
RUN mkdir -p /etc/supervisor/conf.d
COPY build/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /usr/share/nginx/html
COPY ./dist .
RUN mv -f nginx.conf /etc/nginx
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

当然你也可以采用其他管理多进程的方案，比如 `pm2` 等。

按单个镜像的话，改造难度和影响面都不大，与旧部署流程能基本保持一致。

此外，如果每次都安装 `supervisor` 觉得耗时或有风险，你也可以部署一个基础镜像后基于它再构建业务。

```dockerfile
# 基础镜像 Dockerfile.base
FROM node:20.18.0-alpine
RUN apk add nginx supervisor
RUN mkdir -p /etc/supervisor/conf.d
COPY build/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
```
```bash
# 构建基础镜像
docker build -t my-node-nginx-base -f Dockerfile.base .
docker tag my-node-nginx-base your-registry/my-node-nginx-base:1.0
docker push your-registry/my-node-nginx-base:1.0
```
```dockerfile
# 总 Dockerfile
FROM your-registry/my-node-nginx-base:1.0
```

## 端口问题

若微服务端口为 4002，想复用网页端域名，那端口还得暴露和代理一下。

```ini
# supervisord.conf
[program:node]
environment=PORT=4002
```

```nginx
# nginx.conf
location / {
  if ($request_filename ~* .*\.(?:htm|html)$) {
    add_header Cache-Control "private, must-revalidate, no-cache, no-store, max-age=0";
  }
  try_files $uri $uri/ /index.html;
  index index.html;
}

location /front-api/ {
  proxy_pass http://localhost:4002/;
}
```
