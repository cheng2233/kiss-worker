# KISS-WORKER

适用于 [KISS-Translator](https://github.com/fishjar/kiss-translator) 的简单的数据同步服务。

有两种部署方式可供选择：

## Cloudflare Workers 部署（推荐）

本仓库已改用 npm，避免 Cloudflare 使用 Yarn 4 安装旧锁文件时出现
`YN0028: The lockfile would have been modified`。

### 通过 Cloudflare 控制台连接 Git 仓库

1. 在 `wrangler.toml` 中确认 KV 的 `id` 是你创建的命名空间 ID。
2. Workers & Pages 中导入本 Git 仓库。
3. 构建命令填写 `npm run build`，部署命令填写 `npm run deploy`。
4. 在 Worker 的 Settings / Variables and Secrets 中添加加密变量
   `AUTH_VALUE`，值为 KISS Translator 使用的同步密码。
5. 部署后访问 Worker 地址，看到 `{"service":"kiss-worker","status":"ok"}`
   即表示服务已运行。

也可以在本地部署：

```sh
npm install
npm run secret
npm run deploy
```

> `AUTH_VALUE` 必须设置为 Secret，不要直接写入源码或 `wrangler.toml`。

## 原 Cloudflare Workers 部署说明

### 前提条件

- [Cloudflare](https://www.cloudflare.com/) 帐号
- 部署时本地安装 `git` + `nodejs`
- 一个域名（可选）

### 部署步骤

1、登录 Cloudflare 管理面板，进入路径 `dashboard > select Workers & Pages > KV`。创建一个命名空间，名称随意。创建完成后将获得一个`命名空间 ID`。

2、克隆项目，修改 `wrangler.toml` 文件，将前面步骤获取到的`命名空间 ID` 替换到`id`的位置。

```toml
# wrangler.toml
kv_namespaces = [
    { binding = "KV", id = "replace you id here!!!" }
]
```

3、依次执行下面的命令，执行完成会要求设定自己的密码。首次部署时可能需要连接到 Cloudflare 授权。

```sh
yarn install
yarn deploy
```

4、（可选）登录 Cloudflare 管理面板，进入路径 `dashboard > select Workers & Pages > kiss-worker`，点击 `触发器`选项卡，再点击`添加自定义域`添加访问的域名。

## `docker` 部署方式

### 前提条件

- 自有服务器
- `docker`相关知识

### 部署步骤

1、克隆项目，并修改 `docker-compose.yml` 文件，将`APP_KEY`后面的字符修改为你自己的密码。

```yml
version: "3.1"
services:
  kiss-worker:
    image: fishjar/kiss-worker
    environment:
      PORT: 8080
      APP_KEY: 123456 # 修改这里的密码
      APP_DATAPATH: data
    ports:
      - 8080:8080
    volumes:
      - ./data:/app/data
```

2、执行以下命令启动

```sh
docker-compose up -d
```
