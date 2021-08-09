---
description: 使用docker 安装deltaboard 教程
---

# 启动Deltaboard

**安装docker** 

         ****[**访问docker官网安装**](https://docs.docker.com/get-docker/)\*\*\*\*

**下载镜像**

```text
$ docker pull deltampc/deltaboard
```

**界面启动board** 

      1. 打开docker desktop找到deltampc/deltaboard 镜像

      2. 点击run并配置

              ![](../.gitbook/assets/deltaboard_config.png) 

      3. 点击Run 启动

      4. 访问http://localhost:8090

                ![](../.gitbook/assets/deltaboard_login.png) 

**命令行启动**

```text
$ docker run -d -p 8090:8090 deltampc/deltaboard
```



\*\*\*\*

