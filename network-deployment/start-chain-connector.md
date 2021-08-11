# 启动Chain Connector

Chain Connector是连接Delta Node和区块链的中间件。用于对区块链进行抽象，使得Delta Node可以支持各种不同的区块链系统。同时，Chain Connector还负责区块链系统需要的密钥托管和签名相关工作，并负责保证密钥的安全使用。

Chain Connector还可以配置为不需要连接区块链的Coordinator模式。在这种模式下，Chain Connector自身做为整个网络的中心节点，网络中所有的Delta Node都连接到Chain Connector完成任务分发和任务协调。

## 使用Coordinator模式启动Chain Connector

![Coordinator&#x6A21;&#x5F0F;&#x4E0B;&#x7684;&#x65E0;&#x533A;&#x5757;&#x94FE;Delta&#x9690;&#x79C1;&#x8BA1;&#x7B97;&#x7F51;&#x7EDC;&#x7ED3;&#x6784;](../.gitbook/assets/53635fc89ddea878178709dd8e55ba9%20%281%29.png)

### 下载镜像

目前Delta框架还处于开发阶段，后续会发布正式的release版本。在现阶段，我们可以下载开发版本的docker镜像，开发版本镜像的tag名称是`dev`：

```text
$ docker pull deltampc/delta-chain-connector:dev
```

### 初始化配置

在启动Connector之前，需要先初始化配置：

首先，新建文件夹delta\_chain\_connector，作为节点启动的根目录：

```text
$ mkdir delta_chain_connector
```

在节点根目录中，输入命令：

```text
$ cd delta_chain_connector
$ docker run -it --rm -v ${PWD}:/app deltampc/delta_chain_connector:dev init
```

运行命令后，会在根目录`delta_chain_connector`中，新建文件夹`config`，`config`文件夹用来存放节点的配置文件。

### 修改配置文件

在`config`文件夹中，会有一个预先生成的配置文件`config.yaml`:

```text
# 日志配置
log:
  # 日志级别
  level: "INFO"

# Chain Connector运行模式
mode: "coordinator"

# 数据库连接地址
db: "sqlite:///db/chain.db"

# 服务域名
host: "0.0.0.0"
# 服务端口
port: 4500

```

用户可以按需修改配置文件中的内容，完成配置之后，就可以启动Chain Connector。

### 启动Chain Connector服务

使用Docker命令启动Chain Connector，将上一步创建的文件夹绑定到Container内部的`app`文件夹。另外Chain Connector需要对外暴露端口`4500`，作为对外的API端口：

```text
$ docker run -d --name=chain_connector -v ${PWD}:/app -p 4500:4500 deltampc/delta-chain-connector:dev run
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```text
$ docker logs -f chain_connector
```

