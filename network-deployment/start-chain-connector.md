# 启动Chain Connector

## 通过Docker启动Coordinator

推荐Coordinator的Docker镜像来启动Coordinator

### 下载镜像

目前Delta框架还处于开发阶段，后续会发布正式的release版本。在现阶段，我们可以下载开发版本的docker镜像，开发版本镜像的tag名称是`dev`：

```text
$ docker pull deltampc/delta-node-coordinator:dev
```

### 初始化配置

在启动Coordinator之前，需要先初始化配置：

首先，新建文件夹delta\_coordinator，作为节点启动的根目录：

```text
$ mkdir delta_coordinator
```

在节点根目录中，输入命令：

```text
$ cd delta_coordinator
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node-coordinator:dev init
```

运行命令后，会在根目录`delta_coordinator`中，新建文件夹`config`，`config`文件夹用来存放节点的配置文件。

### 配置文件

在`config`文件夹中，会有一个预先生成的配置文件`config.yaml`:

```text
# 日志配置
log:
  # 日志级别
  level: "INFO"

# 区块链节点类型
impl: "monkey"

# 数据库连接地址
db: "sqlite:///db/chain.db"

# 服务域名
host: "0.0.0.0"
# 服务端口
port: 4500

# substrate区块链配置
substrate:
  # subtrate区块链节点地址
  address: ""
```

用户可以按需修改配置文件中的内容，完成配置之后，就可以启动Coordinator。

### 启动Coordinator

使用Docker命令启动Coordinator，将上一步创建的文件夹绑定到Container内部的`app`文件夹。另外Delta Node需要对外暴露端口`4500`，作为对外的API端口：

```text
$ docker run -d --name=coordinator --rm -v ${PWD}:/app -p 4500:4500 deltampc/delta-node-coordinator:dev run
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```text
$ docker logs -f coordinator
```

