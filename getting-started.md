# 快速搭建指南

Delta隐私计算网络由多个组件构成，可根据需要进行选择和组合。在开始搭建前，建议先阅读系统架构说明文档，以对Delta的整个框架结构有一个初步的了解：

{% page-ref page="delta-architecture.md" %}

## 最小网络搭建（无区块链）

最小化的Delta隐私计算网络，需要搭建一个Chain Connector（运行于Coordinator模式），两个Delta Node，一个Deltaboard，如下图所示：

![&#x6700;&#x5C0F;Delta&#x9690;&#x79C1;&#x8BA1;&#x7B97;&#x7F51;&#x7EDC;&#xFF08;&#x65E0;&#x533A;&#x5757;&#x94FE;&#xFF09;](.gitbook/assets/88fbc43f76d794b066889a7cac4d4f4.png)

### 使用All-in-One镜像启动整个网络

Delta提供了一个docker-compose文件，用于一次启动整个网络。

### 使用各个组件的Docker镜像搭建

1.参照启动Chain Connector的教程启动Chain Connector，并配置为Coordinator模式：

{% page-ref page="network-deployment/start-chain-connector.md" %}

2.分别启动两个Delta Node，都连接到上面配置的Chain Connector：

{% page-ref page="network-deployment/start-delta-node.md" %}

3.在Delta Node中各自放置一些测试用的数据

{% page-ref page="network-deployment/prepare-data.md" %}

4.启动Deltaboard，连接到上面配置的其中一个Delta Node：

{% page-ref page="network-deployment/start-deltaboard.md" %}

5.至此最小化的Delta隐私计算网络已经搭建完成，接下来可以开始编写一个隐私计算任务试试看了：

{% page-ref page="network-deployment/run-delta-task.md" %}

## 完整网络搭建（有区块链）

