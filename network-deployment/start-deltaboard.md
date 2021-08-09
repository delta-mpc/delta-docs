---
description: 使用docker 安装deltaboard 教程
---

# 启动Deltaboard

1.**安装docker** 

         **这里**[**访问docker官网安装**](https://docs.docker.com/get-docker/)\*\*\*\*

**2. 下载镜像**

         ****docker pull deltampc/deltaboard

3. 启动board 

         1. 使用命令行一键启动：

              1.1docker run -d -p 8090:8090 deltampc/deltaboard

         2  使用界面启动

              1. 打开docker desktop找到deltampc/deltaboard 镜像

              2. 点击run并配置

              ![](../.gitbook/assets/deltaboard_config.png) 

           

              3. 点击Run 启动

              4. 访问http://localhost:8090

                ![](../.gitbook/assets/deltaboard_login.png) 





\*\*\*\*

\*\*\*\*

