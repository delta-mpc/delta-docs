# 常见问题解答

## 怎样在其他区块链上运行Delta

Delta在架构设计时做了区块链系统的分层，所以理论上可以很容易的使用在其他区块链系统上，比如Fabric、BCOS等。

具体来说，实现一个新的区块链系统的连接需要做两部分工作：

1. 实现Delta的链上智能合约
2. 在`delta-chain-connector`中实现调用区块链的方法

### 实现Delta的链上智能合约

目前Delta已经实现了Solidity的智能合约，如果目标区块链系统使用了EVM做为智能合约层，可以直接使用写好的Solidity合约。如果是Fabric等需要自己写的智能合约，就需要参照Delta的智能合约，实现一套功能和接口完全一样的。Delta的Solidity智能合约放置在下面的Github仓库中：

{% embed url="https://github.com/delta-mpc/delta-contracts" %}

Delta的Solidity合约使用了[Truffle](https://trufflesuite.com/docs/truffle/)的开发框架，目录结构也是一个标准的Truffle项目结构。合约代码放置于`/contracts`目录下，使用方法可参考Truffle的官方教程。如果要自己实现的话，需要把`/contracts`目录下的合约都实现一遍。

### 实现调用区块链的方法

Delta系统中针对区块链的调用进行了封装和解耦。对区块链的调用全部封装在Chain Connector中。Delta Node使用同一套API调用Chain Connector，在Chain Connector中再根据区块链的不同，使用不同的调用方法。

Chain Connector中，调用区块链的方法也是统一的抽象方法。只要针对目标区块链系统，实现具体的调用逻辑就可以了。在Chain Connector的架构中也考虑了对更多区块链的兼容，可以很方便的加入其他区块链的实现。

Chain Connector的代码放置在下面的仓库中：

{% embed url="https://github.com/delta-mpc/delta-chain-connector" %}

在`/src/impl`目录中放置的就是对不同区块链的实现。目前有两个实现，`monkey`和`chain`。`monkey`是无区块链模式的实现，Chain Connector自己做为中心节点，模拟区块链的行为。`chain`是以太坊的实现。如果需要增加新的区块链系统，只要在`/src/impl`中新建一个文件夹，比如`/src/impl/fabric`，然后参考以太坊`/src/impl/chain`的实现方式，实现对应的调用方法就可以了。

调用方法的抽象`Impl`放置在[service.ts](https://github.com/delta-mpc/delta-chain-connector/blob/main/src/impl/service.ts)文件中。需要实现其中的全部方法。

然后使用修改过的Chain Connector启动整个系统，在Chain Connetor的配置文件中指定使用自己新增加的区块链就可以了。

## Docker容器无法启动

如果`docker-compose up`或`docker-compose up -d`有容器无法启动， 输入命令`docker-compose ps`查看具体的容器状态，如果容器状态显示`Exit (1)`，则表明发生了异常。 常见的情况是容器`node1`、`node2`或`node3`启动失败。

这里以`node1`发生异常为例，输入命令`docker-compose logs -f node1`，查看node1的日志。 发生的异常不同，处理方式也不同。

### service.XXX.depends\_on contains an invalid type

![](.gitbook/assets/node\_err3.png)

类似上图的问题，是由docker和docker-compose版本过低引起的。需要将docker版本升级至19.03.0+，docker-compose版本升级至1.29.2+。

### node XXX has already joined

![](.gitbook/assets/node\_err1.png)

如果遇到`node XXX has already joined`这种异常，表明上一次关闭节点时，节点没有正常退出。

以`no-blockchain`为例子。在容器`connector`运行正常的情况下，在`no-blockchain`文件夹下，依次输入命令：

```
$ docker run --rm -v ${PWD}/delta-node1:/app --network no-blockchain_default deltampc/delta-node:0.5.3 leave
$ docker run --rm -v ${PWD}/delta-node2:/app --network no-blockchain_default deltampc/delta-node:0.5.3 leave
```

然后输入命令：

```
$ docker-compose restart
```

即可重新启动。

在`with-blockchain`文件夹下，命令是类似的。需要将命令中的`no-blockchain_default`改为`with-blockchain_default`。

### PermissionError: \[Errno 13] Permission denied: 'XXX'

![](.gitbook/assets/node\_err2.png)

如果遇到`PermissionError: [Errno 13] Permission denied: 'XXX'`这种异常，表明发生了权限问题，docker容器无法挂载volume。 这种问题一般是由于用户是root用户导致的。在非root用户的情况下，一般不会出现如下问题。

如果不能切换到非root用户，可以修改`docker-compose.yml`文件，在所有容器的配置中添加`privileged: true`。 例如：

```
  node1-init:
    image: deltampc/delta-node:0.5.3
    container_name: node1-init
    volumes:
      - "./delta-node1:/app"
    command: "init"
    privileged: true  # 新增此行
```

之后即可正常启动。

####
