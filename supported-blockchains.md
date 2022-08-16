# 区块链支持

### 以太坊兼容（EVM Compatible）区块链

Delta目前的智能合约使用Solidity编写，在Chain Connector中实现了以太坊API的调用，理论上可以在所有兼容以太坊的区块链上运行。但是由于Chain Connector目前使用了以太坊的`pub/sub`接口来订阅以太坊事件，因此需要目标区块链系统支持`pub/sub`事件订阅。下表中列出了经社区测试的Delta在各个以太坊兼容区块链上的适用情况：

| 区块链                                       | 是否兼容 | 情况说明              | 测试者                        |
| ----------------------------------------- | ---- | ----------------- | -------------------------- |
| [Conflux/树图](https://confluxnetwork.org/) | 否    | 缺少`pub/sub`事件订阅支持 | Jupyter(s2068649@ed.ac.uk) |
| [FISCO BCOS](http://fisco-bcos.org/)      | 未知   | 未测试               |                            |
|                                           |      |                   |                            |

如果是Delta兼容的区块链系统，可以直接使用现有的Chain Connector进行连接。只需要在配置文件中指定`impl`为`ethereum`，并且按照区块链系统的要求设置具体的配置项即可。

如果需要在不兼容的以太坊区块链上运行，需要开发者自己在Chain Connector中实现对应的支持，比如使用`json-rpc`替代掉`pub/sub`方法，即可运行在没有`pub/sub`接口的区块链上。

### 其他区块链

其他区块链系统的支持，需要开发者自己进行开发。Delta在架构设计时做了区块链系统的分层，所以理论上可以很容易的使用在其他区块链系统上，比如Fabric等。

具体来说，实现一个新的区块链系统的连接需要做两部分工作：

1. 实现Delta的链上智能合约
2. 在`delta-chain-connector`中实现调用区块链的方法

#### 实现Delta的链上智能合约

目前Delta已经实现了Solidity的智能合约，如果目标区块链系统使用了EVM做为智能合约层，可以直接使用写好的Solidity合约。如果是Fabric等需要自己写的智能合约，就需要参照Delta的智能合约，实现一套功能和接口完全一样的。Delta的Solidity智能合约放置在下面的Github仓库中：

{% embed url="https://github.com/delta-mpc/delta-contracts" %}

Delta的Solidity合约使用了[Truffle](https://trufflesuite.com/docs/truffle/)的开发框架，目录结构也是一个标准的Truffle项目结构。合约代码放置于`/contracts`目录下，使用方法可参考Truffle的官方教程。如果要自己实现的话，需要把`/contracts`目录下的合约都实现一遍。

#### 实现调用区块链的方法

Delta系统中针对区块链的调用进行了封装和解耦。对区块链的调用全部封装在Chain Connector中。Delta Node使用同一套API调用Chain Connector，在Chain Connector中再根据区块链的不同，使用不同的调用方法。

Chain Connector中，调用区块链的方法也是统一的抽象方法。只要针对目标区块链系统，实现具体的调用逻辑就可以了。在Chain Connector的架构中也考虑了对更多区块链的兼容，可以很方便的加入其他区块链的实现。

Chain Connector的代码放置在下面的仓库中：

{% embed url="https://github.com/delta-mpc/delta-chain-connector" %}

在`/src/impl`目录中放置的就是对不同区块链的实现。目前有两个实现，`coordinator`和`ethereum`。`coordinator`是无区块链模式的实现，Chain Connector自己做为中心节点，模拟区块链的行为。`ethereum`是以太坊的实现。如果需要增加新的区块链系统，只要在`/src/impl`中新建一个文件夹，比如`/src/impl/fabric`，然后参考以太坊`/src/impl/ethereum`的实现方式，实现对应的调用方法就可以了。

调用方法的抽象`Impl`放置在[service.ts](https://github.com/delta-mpc/delta-chain-connector/blob/main/src/impl/service.ts)文件中。需要实现其中的全部方法。

然后使用修改过的Chain Connector启动整个系统，在Chain Connetor的配置文件中指定使用自己新增加的区块链就可以了。
