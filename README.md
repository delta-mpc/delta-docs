---
description: 开箱即用的区块链隐私计算框架
---

# Delta开发文档

## 功能介绍

Delta可以联合分散在各处的数据，进行统计计算以及机器学习，数据全程不会离开本地，实现了数据的可用不可见，可应用于金融、医疗、政府、知识产权等领域，实现联合风控、联合科研、政企数据连接等需求，充分发挥数据的价值。

Delta通过封装整合联邦学习、安全多方计算、差分隐私等最新的隐私计算技术，降低了开发门槛，使用者无需了解隐私计算技术，也可快速实现计算需求。 用户可以快速地部署Delta节点，搭建隐私计算网络，联合多方数据，完成隐私计算。

Delta保证了计算结果的可信性，在用户无法获取原始数据的前提下，通过集成的区块链和零知识证明技术验证计算结果的正确性。

### 框架特点

Delta包含了搭建一个完整的区块链隐私计算网络的全部组件，每个组件都可以使用Docker镜像快速部署。开发者可以根据需要选择需要的组件快速完成网络的搭建。

Delta对隐私计算底层技术进行了彻底的封装，编写隐私计算任务无需对底层的原理有任何了解，只需要开发者按照以前的方式完成计算任务编写，Delta就可以将其做为隐私计算任务，在多节点网络中进行执行。

Delta支持使用PyTorch编写的机器学习任务，以及使用Pandas编写的数据统计任务。以前写的PyTorch/Pandas代码几乎可以不加修改的直接在Delta网络中执行，通过隐私计算联合多个节点的数据，获得更大数据集上的模型训练结果、验证结果以及统计数据。

关于Delta框架的详细架构说明，可以参考这篇文章：

{% content-ref url="delta-architecture.md" %}
[delta-architecture.md](delta-architecture.md)
{% endcontent-ref %}

## Delta网络快速搭建

Delta网络可以使用Docker镜像快速完成搭建，详情请参考：

{% content-ref url="getting-started.md" %}
[getting-started.md](getting-started.md)
{% endcontent-ref %}

## Delta社区

\*\*Github：\*\*代码使用过程中出现的问题，可以在Github中对应的仓库提Issue，为方便其他人查询，请将Issue标题写为问题的大致描述：

{% embed url="https://github.com/delta-mpc" %}

\*\*微信交流群：\*\*请扫码加好友，并备注"Delta社区"：

![扫码加好友，并备注"Delta社区"](.gitbook/assets/9db164bd4d5d449ddb9da507085d925.png)

\*\*Slack：\*\*点击下面的邀请链接加入Slack：

{% embed url="https://join.slack.com/t/delta-mpc/shared_invite/zt-uaqm185x-52oCXcxoYvRlFwEoMUC8Tw" %}

## 相关链接

Delta官网：

{% embed url="https://deltampc.com/" %}

Github源代码仓库：

{% embed url="https://github.com/delta-mpc" %}

隐私计算技术专栏：

{% embed url="https://www.zhihu.com/column/blocktech" %}
