# 启动Delta Node

## 通过Docker镜像启动Delta Node

推荐使用Delta Node的Docker镜像来进行部署

### 下载镜像

从Docker Hub拉取最新的release版本镜像：

```
$ docker pull deltampc/delta-node:0.8.3
```

### 初始化配置

delta-node节点保存的数据包括配置文件、保存的用户数据、系统运行日志等，在启动节点服务之前，需要先初始化配置：

首先，新建文件夹delta\_node，作为节点启动的根目录：

```
$ mkdir delta_node
```

在节点根目录中，输入命令：

```
$ cd delta_node
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node:0.8.3 init
```

运行命令后，会在根目录`delta_node`中，新建文件夹`config,task,data`，其中，`config`文件夹用来存放节点的配置文件，`task`文件夹用来存放节点任务执行的中间结果，`data`用来存放节点提供的数据。

### 修改配置文件

在`config`文件夹中，会有一个预先生成的配置文件`config.yaml`。在首次启动时，必须修改的项目一个是Chain Connector的地址，修改为在上一节中配置好的Chain Connector的IP地址，另一个是需要配置Delta Node自身的IP地址，用于在链上进行本节点的身份注册，使得其他节点能够和本节点通讯。需要找到本节点的公网IP地址，填入配置文件中：

```
---
# 区块链节点连接配置
chain_connector:
  # 区块链连接器的地址
  host: "127.0.0.1"
  # 区块链连接器的端口
  port: 4500

# 本节点对外的公开地址，将会公开到区块链上，供其他Delta Node节点连接
node:
  # 本节点的名称
  name: "node1"
  # 本节点的对外公开地址
  url: "127.0.0.1:6700"

# 零知识证明服务连接配置
zk:
  # 零知识证明服务地址
  host: ""
  # 零知识证明服务端口
  port: 3400

# 本节点的http服务监听地址，用于Deltaboard的连接以及Delta Task的注册等
api_port: 6700
```

完成配置之后，即可启动`Delta Node` 节点。

### 启动节点服务

使用Docker命令行创建并启动节点的container，将上一步创建的文件夹绑定到Container内部的`app`文件夹。另外Delta Node需要对外暴露端口`6700`，用于对外的API以及节点间通信：

```
$ docker run -d --name=delta_node_1 -v ${PWD}:/app -p 6700:6700 deltampc/delta-node:0.8.3
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```
$ docker logs -f delta_node_1
```

Delta Node也会将log同时输出到本地数据文件夹中的log目录，可以在log目录中查看节点运行日志。

至此Delta Node启动完成，在执行Delta计算任务之前，还需要在Delta Node的data文件夹中放置一些数据，供Delta计算任务调用：

{% content-ref url="prepare-data.md" %}
[prepare-data.md](prepare-data.md)
{% endcontent-ref %}

如果需要在任务中开启零知识证明，则需要在执行任务前，额外启动Delta ZK服务，用于生成零知识证明。启动Delta ZK请参考：

{% content-ref url="start-delta-zk.md" %}
[start-delta-zk.md](start-delta-zk.md)
{% endcontent-ref %}


数据准备好后，我们就可以真正开始执行计算任务了。可以部署Deltaboard，在web界面中可视化管理Delta网络，以及在线编辑和运行Delta任务：

{% content-ref url="start-deltaboard.md" %}
[start-deltaboard.md](start-deltaboard.md)
{% endcontent-ref %}

如果不需要GUI的话，也可以直接在本地的IDE中编写Delta Task，通过delta-task库直接调用Delta Node API完成任务发送和执行：

{% content-ref url="../delta-task-development/manage-task-with-delta-node-api.md" %}
[manage-task-with-delta-node-api.md](../delta-task-development/manage-task-with-delta-node-api.md)
{% endcontent-ref %}
