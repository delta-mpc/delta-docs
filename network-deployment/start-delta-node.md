# 启动Delta Node

## 通过Docker镜像启动Delta Node

推荐使用Delta Node的Docker镜像来进行部署

### 下载镜像

目前Delta框架还处于开发阶段，后续会发布正式的release版本。在现阶段，我们可以下载开发版本的docker镜像，开发版本镜像的tag名称是`dev`：

```text
$ docker pull deltampc/delta-node:dev
```

### 初始化配置

delta-node节点保存的数据包括配置文件、保存的用户数据、系统运行日志等，在启动节点服务之前，需要先初始化配置：

首先，新建文件夹delta\_node，作为节点启动的根目录：

```text
$ mkdir delta_node
```

在节点根目录中，输入命令：

```text
$ cd delta_node
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node:dev init
```

运行命令后，会在根目录`delta_node`中，新建文件夹`config,task,data`，其中，`config`文件夹用来存放节点的配置文件，`task`文件夹用来存放节点任务执行的中间结果，`data`用来存放节点提供的数据。

### 配置文件

在`config`文件夹中，会有一个预先生成的配置文件`config.yaml`:

```text
---
# 日志配置
log:  
  # 日志级别
  level: "INFO"
  # 日志输出的目录
  dir: "logs"

# 本地数据库地址，这里使用sqlite数据库，数据库文件保存在文件delta_node/db/delta.db中
db: "sqlite:///db/delta.db"

# 区块链节点连接配置
chain_connector:
  # 区块链节点地址，必填
  address: ""

# 本节点对外的公开地址，将会公开到区块链上，供其他Delta Node节点连接
node_address:
  # 节点的公网地址，必填
  host: ""
  # 节点公开地址的端口
  port: 6800

# 本节点的http服务监听地址，用于Deltaboard的连接以及Delta Task的注册等
api_port: 6700

# 本节点的储存地址，用于保存任务执行的中间结果，本例中存放在delta_node/task文件夹下
task_dir: "task"

# 本节点的数据地址，用于存放节点提供的数据，本例中存放在delta_node/data文件夹下
data_dir: "data"
```

用户必须填写配置文件中的必填项（区块链节点地址、节点公网地址），完成配置之后，即可启动`Delta Node` 节点。

### 连接区块链节点

在config.yaml文件中需要配置Delta Node连接到我们在上一节中配置好的区块链节点，以完成组网以及任务获取、提交的功能。在config文件的contract章节填入配置好的区块链节点IP地址和端口号即可。如果还没有完成区块链节点的搭建，可参考：

{% page-ref page="start-blockchain-node.md" %}

Delta提供了另一种简化的组网方式，在不需要构建P2P网络，也不需要计算可信性保障时，可使用Delta Node Coordinator替代区块链完成组网。此时，需要配置Delta Node连接到Coordinator服务。详情可参考：

{% page-ref page="start-chain-connector.md" %}

### 启动节点服务

使用Docker命令行创建并启动节点的container，将上一步创建的文件夹绑定到Container内部的`app`文件夹。另外Delta Node需要对外暴露两个端口`6700`和`6800`，用于对外的API以及节点间通信：

```text
$ docker run -d --name=delta_node_1 --rm -v ${PWD}:/app -p 6800:6800 -p 6700:6700 deltampc/delta-node:dev run
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```text
$ docker logs -f delta_node_1
```

Delta Node也会将log同时输出到本地数据文件夹中的log目录，可以在log目录中查看节点运行日志。

至此Delta Node启动完成，下一步我们将启动Deltaboard的服务，用于对Delta Node的可视化管理，以及计算任务的在线编辑和运行：

{% page-ref page="start-deltaboard.md" %}
