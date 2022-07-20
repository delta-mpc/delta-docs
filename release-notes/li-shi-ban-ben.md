# 历史版本

### v0.3.5

修复了Deltaboard远程部署时也一直访问localhost后端api server的bug

{% embed url="https://github.com/delta-mpc/deltaboard/issues/51" %}

修复了任务发生错误时不显示错误信息的bug

{% embed url="https://github.com/delta-mpc/delta-node/issues/19" %}

修复了任务列表没有倒序显示的bug

{% embed url="https://github.com/delta-mpc/deltaboard/issues/53" %}

修复了Delta-all-in-one中，no-blockchain无法启动的bug

{% embed url="https://github.com/delta-mpc/delta-all-in-one/issues/4" %}

修复了长时间空闲后，Delta-node节点无法接收事件的bug

{% embed url="https://github.com/delta-mpc/delta-node/issues/21" %}

### v0.3.0

此版本增加了横向联邦学习任务的链上协调过程，任务执行中涉及到结算的步骤都使用智能合约实现。详细的原理介绍可以参考这篇知乎文章：

{% embed url="https://zhuanlan.zhihu.com/p/414433931" %}

具体的使用方法参考这篇教程，可以使用兼容EVM的区块链部署，也可以使用Delta Chain进行部署：

{% embed url="https://zhuanlan.zhihu.com/p/459412742" %}
