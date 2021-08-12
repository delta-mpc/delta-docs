---
description: Deltaboard 是deltanode的用户界面和开发环境。支持在线编写deltanode代码。
---

# 启动Deltaboard

## 安装Docker

docker是一个基于linux container技术的虚拟执行环境。启动deltaboard需要先安装docker,可以 ****[**访问docker官网安装Docker Desktop**](https://docs.docker.com/get-docker/)\*\*\*\*

## 下载镜像

```text
$ docker pull deltampc/deltaboard:dev
```

## **初始化配置**

```text
$docker run --rm -d -v ${PWD}:/app/app_config deltampc/deltaboard:dev init
```

执行完后将会在  当前路径下生成config.yaml文件

```text
db:
  connection: ''
  driver: ''
web_port: '8090'
```

## 命令行执行



```text
docker run --rm -d -p 8090:8090 -v ${PWD}:/app/app_config dashboard_in_all
```



## **打开Deltaboard**

浏览器访问 http://localhost:8090

![](../.gitbook/assets/deltaboard_login.png)

## 使用mysql并持久化jupyterhub data

默认情况下deltaboard 会使用自带的sqlite作为数据存储。用户也可以根据自己的情况配置mysql数据库

启动并配置mysql 

修改之前的config.yaml

```text
db:
  connection: 'mysql_user:my_sqlpassword@(mysql_host:mysql_port)/my_sql_database'
  driver: 'mysql'
web_port: '8090'
```

重新使用命令行运行docker

```text
docker run -d -p 8090:8090 -v ${PWD}:/app/app_config dashboard_in_all
```

使用 -v ${PWD}:/app/app\_config 将jupyter的用户数据和镜像的配置文件config.yaml映射到本地的文件系统

启动镜像后用户的ipynb文件将会保存当前路径的notebook\_dir目录下