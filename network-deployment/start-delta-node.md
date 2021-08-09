# 启动Delta Node

## 通过Docker部署Delta Node

推荐使用Delta Node的Docker镜像来进行部署

### 下载镜像

目前Delta框架还处于开发阶段，后续会发布正式的release版本。在现阶段，我们可以下载开发版本的docker镜像，开发版本镜像的tag名称是`dev`：

```text
$ docker pull deltampc/delta-node:dev
```

### 启动节点

首先，新建文件夹delta\_node，作为节点启动的根目录。节点的数据，包括配置文件、保存的用户数据、系统运行日志等，都保存在根目录中：

```text
$ mkdir delta_node
$ cd delta_node
```

在根目录中新建文件夹config\_files，并在config\_files文件夹下新建配置文件config.yaml：

```text
$ mkdir config_files
$ cd config_files
$ touch config.yaml
```

config.yaml配置文件的配置项和说明如下：

```text
---
# 日志配置
log:  
  # 日志级别
  level: "DEBUG"
  # 是否将日志存放在数据库中
  enable_db: True

# 本地数据库地址，这里使用sqlite数据库，数据库文件保存在文件delta_node/db/delta1.db中
db: "sqlite:///db/delta1.db"

# 区块链节点连接配置
contract:
  # 区块链节点地址
  address: "http://127.0.0.1:4500"
  # 区块链节点的实现类型，这里使用monkey，即coordinator来代替区块链
  impl: "monkey"

# 本节点对外的连接地址，用于节点之间互相通信
url: "127.0.0.1:6800"

# 本节点的http服务地址，用于向deltaboard提供服务
server:
  host: "0.0.0.0"
  port: 6700

# 本节点的储存地址，用于保存任务执行的中间结果，本例中存放在delta_node/task文件夹下
storage_dir: "task"

# 本节点的数据地址，用于存放节点提供的数据，本例中存放在delta_node/data文件夹下
data_dir: "data"
```

新建数据文件夹`data`（与配置文件中的`data_dir`项对应）和任务文件夹`task`，在文件夹下放入节点提供的数据：

```text
$ cd ..
$ mkdir data
$ mkdir task
```

启动节点：

```text
$ docker run -d --name=delta_node1 --rm -v ${PWD}:/app -p 6800:6800 -p 6700:6700 deltampc/delta-node:dev
```



