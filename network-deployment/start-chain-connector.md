# 启动Chain Connector

Chain Connector是连接Delta Node和区块链的中间件。用于对区块链进行抽象，使得Delta Node可以支持各种不同的区块链系统。同时，Chain Connector还负责区块链系统需要的密钥托管和签名相关工作，并负责保证密钥的安全使用。

Chain Connector还可以配置为不需要连接区块链的Coordinator模式。在这种模式下，Chain Connector自身做为整个网络的中心节点，网络中所有的Delta Node都连接到Chain Connector完成任务分发和任务协调。

## 使用Coordinator模式启动Chain Connector

![Coordinator模式下的无区块链Delta隐私计算网络结构](<../.gitbook/assets/53635fc89ddea878178709dd8e55ba9 (2) (2) (3) (1) (4) (4) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### 下载镜像

```
$ docker pull deltampc/delta-chain-connector:0.8.3
```

### 初始化配置

在启动Connector之前，需要先初始化配置：

首先，新建文件夹delta\_chain\_connector，作为节点启动的根目录：

```
$ mkdir delta_chain_connector
```

在节点根目录中，输入命令：

```
$ cd delta_chain_connector
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-chain-connector:0.8.3 init
```

运行命令后，会在根目录`delta_chain_connector`中，新建文件夹`config`，`config`文件夹用来存放节点的配置文件。

### 修改配置文件

在`config`文件夹中，会有一个预先生成的配置文件`config.json`:。在首次启动前，我们必须要配置的项目包括Chain Connector的运行模式。如果运行模式配置为Blockchain，那么还需要配置区块链节点的IP地址。

在这里我们将impl设置为`coordinator`，即为coordinator模式：

```
{
  "log": {
    "level": "debug"
  },
  "impl": "coordinator",
  "host": "0.0.0.0",
  "port": 4500,
  "coordinator": {
    "db": {
      "type": "sqlite",
      "url": "db/chain.db"
    }
  }
}
```

coordinator模式的Chain Connector，还需要配置一个数据库，用于存储"链上"数据。可以直接使用SQLite，指定一个数据库文件存储地址即可。

完成配置之后，就可以启动Chain Connector。

### 启动Chain Connector服务

使用Docker命令启动Chain Connector，将上一步创建的文件夹绑定到Container内部的`app`文件夹。另外Chain Connector需要对外暴露端口`4500`，作为对外的API端口：

```
$ docker run -d --name=chain_connector -v ${PWD}:/app -p 4500:4500 deltampc/delta-chain-connector:0.8.3 run
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```
$ docker logs -f chain_connector
```

## 使用Chain模式启动Chain Connector

使用Chain模式启动Chain Connector 与使用Coordinator 模式启动差别不大，唯一需要修改的就是配置文件。

### 修改配置文件

要以Chain模式启动Chain Connector，我们需要将impl改为chain：

```
{
  "log": {
    "level": "debug"
  },
  "impl": "chain",
  "host": "0.0.0.0",
  "port": 4500,
  "ethereum": {
    "nodeAddress": "0x6578aDabE867C4F7b2Ce4c59aBEAbDC754fBb990",
    "privateKey": "0xf0f239a0cc63b338e4633cec4aaa3b705a4531d45ef0cbcc7ba0a4b993a952f2",
    "provider": "ws://localhost:8545",
    "gasPrice": 20000000000,
    "gasLimit": 6721975,
    "chainParam": {
      "chainId": 1337,
      "name": ""
    },

    "identity": {
      "contractAddress": "0xCe69c1DDCcD29a821bB4d3BdEEb3EdE9De9C7903"
    },
    "hfl": {
      "contractAddress": "0x7864Bf464F9ecE0D3A95cA55e171D9060cf7336a"
    },
    "datahub": {
      "contractAddress": "0x02c1d56eB5C04Cb026cd98Cf38F75A9942EED39B"
    },
    "hlr": {
      "contractAddress": "0x19eE33EC0eAC58205f1797303461dEa88BFF93E9",
      "verifiers": {
        "3": "0x075200EBa6317bE48ED73c04082088fab26f5D02"
      }
    }
  }
}
```

接下来，我们需要修改`ethereum.nodeAddress`、`ethereum.privateKey`、`ethereum.provider`、`ethereum.identity`、`ethereum.hfl`、`ethereum.datahub`以及`ethereum.hlr`这几项配置。

其中，`ethereum.nodeAddress`和`ethereum.privateKey`用来配置用户的区块链钱包地址和私钥。用户可以使用任意与以太坊兼容的钱包（比如[Metamask](https://metamask.io/)），来生成钱包地址和私钥。

`chain.provider`用来配置区块链节点的地址，这里需要使用WebSocket的链接地址。

`ethereum.identity`、`ethereum.hfl`、`ethereum.datahub`和`ethereum.hlr`分别对应IdentityContract、HFLContract、DataHub和HLR这些智能合约的配置。 这些配置项目下的`contractAddress`项，代表对应的智能合约地址。在`ethereum.hlr.verifiers`项下，用来配置不同输入长度所对应的零知识证明验证合约的地址，例如`ethereum.hlr.verifiers。3`，就对应于智能合约`PlonkVerifier3`的地址。

在部署智能合约章节可以了解如何部署智能合约并获得合约地址。

{% content-ref url="deploy-smart-contracts.md" %}
[deploy-smart-contracts.md](deploy-smart-contracts.md)
{% endcontent-ref %}

### 启动Chain Connector服务

使用Chain模式启动Chain Connector服务的方式与使用Coordinator模式的方式完全一致。
