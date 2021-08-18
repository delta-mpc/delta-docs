---
description: Deltaboard is a user interface and development environment for deltanode. It can support for editing deltanode task code online.
---

# Start Deltaboard

## Start Deltaboard by docker

We recommend to deploy Deltaboard by Docker.

### Pull Docker Image

```bash
$ docker pull deltampc/deltaboard:dev
```

### Configuration

Deltaboard need to store some data locally which includes configuration file, user data and logs, etc.. Before starting deltaboard, we need to config the board first.

Firstly, make a new directory called `deltaboard` as the root directory of the board:

```text
$ mkdir deltaboard
```

Then, in the root directory, input command:

```text
$ cd deltaboard
$ docker run -it --rm -v ${PWD}:/app deltampc/deltaboard:dev init
```

运行命令后，会在根目录`deltaboard`中，新建文件夹`config, data，db`，其中，`config`文件夹用来存放节点的配置文件，data文件夹用来存放节点保存的用户数据，比如JupyterLab中的代码和数据等，db文件夹用来存放deltaboard使用的sqlite数据库文件。
This command will create three new sub directories in the root directory, called `config`, `data` and `db`. The `config` directory is for storing configuration file, the `data` directory is for storing user data such as code in JupyterLab, and the `db` directory is for storing database file for deltaboard.

### Start Docker Container

```text
$ docker run -d --name=deltaboard -v ${PWD}:/app -p 8090:8090 deltampc/deltaboard:dev
```

### **Visit Deltaboard**

Visit [http://localhost:8090](http://localhost:8090) in the browser

![](../.gitbook/assets/deltaboard_login.png)

This interface means that deltaboard has started.
