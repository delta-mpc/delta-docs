# 启动区块链浏览器

## 通过Docker镜像部署区块链浏览器

推荐通过Delta Chain Explorer的Docker镜像来进行部署

## 下载镜像

```text
$ docker pull deltampc/delta-chain-explorer:dev
```

### 初始化配置

新建文件夹delta-explorer，作为节点启动的根目录

```text
$ mkdir delta-explorer
```

然后进入根目录，创建环境变量文件

```text
$ cd delta-explorer && touch .env
```

环境变量文件示例如下：

```text
DATABASE_URL=postgresql://<db user>:<db password>@<db host>:<db port>/explorer?ssl=false
ETHEREUM_JSONRPC_VARIANT=ganache
ETHEREUM_JSONRPC_HTTP_URL=<RPC endponit url>
ETHEREUM_JSONRPC_TRACE_URL=<RPC endponit url>
ETHEREUM_JSONRPC_WS_URL=<websock endponit url>
COIN=DAI
```

P.S.

1. 需要用户提供DATABASE\_URL作为后端服务持久化数据库
2. 容器中访问宿主机的ip地址不是127.0.0.1，具体取决于操作系统
   * 对于windows和Mac系统，请使用域名host.docker.internal
   * 对于linux系统，请使用ifcongfig查看网卡docker0的ip地址（默认为172.17.0.1）

## 启动

```text
$ docker run -d -p 4000:4000 --env-file ./.env deltampc/delta-chain-explorer:dev
```

在浏览器中访问地址[http://localhost:4000](http://localhost:4000)

## 数据库

如果没有可访问的postgres数据库实例，可以在本地使用docker启动postgres数据库

```text
docker run -d -e POSTGRES_PASSWORD='1234qwer' postgres:alpine3.14
```

