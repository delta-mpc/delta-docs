---
description: Deltaboard 是deltanode的用户界面和开发环境。支持在线编写deltanode代码。
---

# 启动Deltaboard

## 通过Docker镜像启动Deltaboard

推荐使用Deltaboard的Docker镜像来进行部署

### 下载镜像

```text
$ docker pull deltampc/deltaboard:dev
```

### 初始化配置

deltaboard节点保存的数据包括配置文件、保存的用户数据、系统运行日志等，在启动节点服务之前，需要先初始化配置：

首先，新建文件夹deltaboard，作为节点启动的根目录：

```text
$ mkdir deltaboard
```

在节点根目录中，输入命令：

```text
$ cd deltaboard
$ docker run -it --rm -v ${PWD}:/app deltampc/deltaboard:dev init
```

运行命令后，会在根目录`deltaboard`中，新建文件夹`config, data`，其中，`config`文件夹用来存放节点的配置文件，data文件夹用来存放节点保存的用户数据，比如JupyterLab中的代码和数据等。

### 启动节点服务

```text
$ docker run -d --name=deltaboard -v ${PWD}:/app -p 8090:8090 deltampc/deltaboard:dev
```

### **访问Deltaboard**

浏览器访问 [http://localhost:8090](http://localhost:8090)

![](../.gitbook/assets/deltaboard_login.png)

看到这个界面，说明Deltaboard已经启动完成。

