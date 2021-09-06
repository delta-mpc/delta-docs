# 部署智能合约

需要安装Node.js和yarn工具

## 下载源码

```text
$ git clone https://github.com/delta-mpc/delta-contracts.git
```

## 安装

运行命令

```text
$ yarn
```

## 部署

1. 删除compile目录下的所有文件
2. 设置环境变量

   ```text
   ADDRESS=<your eth address>
   PRIVATE_KEY=<your eth private key>
   WS_URL=<websocket endpoit>
   ```

   P.S.

   1. 需要保证ADDRESS变量代表的账户地址中有足够多的以太余额
   2. 如果区块链节点启动在本地，那么容器中访问宿主机的ip地址不是127.0.0.1，具体取决于操作系统
      * 对于windows和Mac系统，请使用域名host.docker.internal
      * 对于linux系统，请使用ifcongfig查看网卡docker0的ip地址（默认为172.17.0.1）

3. 运行命令

   ```text
   yarn deploy
   ```

4. 在compile目录下查看Mpc.json文件，包含合约部署信息

