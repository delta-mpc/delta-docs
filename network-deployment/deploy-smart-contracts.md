# 部署智能合约

## 通过Docker镜像部署智能合约到Delta Chain

推荐通过Delta Contacts的Docker镜像来进行部署

## 下载镜像

```
$ docker pull deltampc/delta-contracts:dev
```

### 初始化配置

新建文件夹delta-contracts，作为节点启动的根目录

```
$ mkdir delta-contracts
```

然后进入根目录，创建环境变量文件

```
$ cd delta-contracts && touch .env
```

环境变量文件示例如下：

```
ADDRESS=<your eth address>
PRIVATE_KEY=<your eth private key>
WS_URL=<websocket endpoit>
```

P.S. 

1. 需要保证ADDRESS变量代表的账户地址中有足够多的以太余额
2. 如果区块链节点启动在本地，那么容器中访问宿主机的ip地址不是127.0.0.1，具体取决于操作系统
   * 对于windows和Mac系统，请使用域名host.docker.internal
   * 对于linux系统，请使用ifcongfig查看网卡docker0的ip地址（默认为172.17.0.1）



## 运行容器

```
$ docker run --rm -v ${PWD}/compile:/app/compile --env-file ./.env deltampc/delta-contracts:dev deploy
```

运行成功后，会在根目录创建文件compile/Mpc.json，包含部署后的合约信息。

