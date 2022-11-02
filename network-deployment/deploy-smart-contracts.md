# 部署智能合约

Delta提供了Solidity语言编写的合约，可以部署在支持EVM虚拟机的区块链上。

智能合约的编译和部署需要通过Node.js包truffle来进行。在开始部署以前，需要确认本地安装了Node.js。

### 下载Delta Contracts源码

Delta Contracts代码库中包含了合约源代码，以及部署合约需要的自动化工具：

```text
$ git clone --depth 1 --branch v0.8.1 https://github.com/delta-mpc/delta-contracts.git
```

### 安装依赖

使用`npm`来安装源代码依赖的其他代码库：

```
$ npm install -g truffle
```

### 编译及部署智能合约

编辑`truffle-config.js`文件，设置区块链节点的地址：

```
module.exports = {
  networks: {
    development: {
      url: <eth url>,
      network_id: "*", // Match any network id
    },
  },
}
```

其中，`<eth url>`是区块链节点的服务地址，注意需要地址需要加上协议头，即"http://:"或"ws://:"。 如果使用Delta Chain Node进行部署，协议使用"ws://"。

然后运行下面的命令，编译并部署`contracts`目录下的智能合约。其中，IdentityContract合约是用来进行节点身份管理的合约，HFLContract合约是用来进行横向联邦学习的合约，
DataHub合约是用来进行节点数据管理的合约，HLR合约是用来进行横向逻辑回归的合约，PlonkVerifier3合约是用来验证输入长度为3的零知识证明合约：

```bash
$ truffle migrate
```

看到如下输出，则表明部署成功：

```bash
...

2_deploy_identity.js
====================

   Deploying 'IdentityContract'
   ----------------------------
   > transaction hash:    0x3e15146bb32ddfa94dda6d4ae073e198ecabb661d1e41dc34c6429787cda11b0
   > Blocks: 0            Seconds: 0
   > contract address:    0xCe69c1DDCcD29a821bB4d3BdEEb3EdE9De9C7903
   > block number:        3
   > block timestamp:     1665558190
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.97278602
   > gas used:            1164499 (0x11c4d3)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.02328998 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.02328998 ETH


3_deploy_hfl.js
===============

   Deploying 'HFLContract'
   -----------------------
   > transaction hash:    0xd0d223126c28c9dc48aed6714067c30ba14e4dd2740ab3c99fd7af78aadc826a
   > Blocks: 0            Seconds: 0
   > contract address:    0x7864Bf464F9ecE0D3A95cA55e171D9060cf7336a
   > block number:        5
   > block timestamp:     1665558190
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.89255696
   > gas used:            3984163 (0x3ccb23)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.07968326 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.07968326 ETH


4_deploy_verifier3.js
=====================

   Deploying 'PlonkVerifier3'
   --------------------------
   > transaction hash:    0xb99668bf525e660442e2683852522850b2db614871e8a62e4281a241327cd309
   > Blocks: 0            Seconds: 0
   > contract address:    0x075200EBa6317bE48ED73c04082088fab26f5D02
   > block number:        7
   > block timestamp:     1665558191
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.86093038
   > gas used:            1554039 (0x17b677)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.03108078 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.03108078 ETH


5_deploy_datahub.js
===================

   Deploying 'DataHub'
   -------------------
   > transaction hash:    0x88dbb128fdefc619ae9825fbe59c4ee90f7d591fdbacadb5e5cfa10975c58d2a
   > Blocks: 0            Seconds: 0
   > contract address:    0x02c1d56eB5C04Cb026cd98Cf38F75A9942EED39B
   > block number:        9
   > block timestamp:     1665558191
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.85419864
   > gas used:            309297 (0x4b831)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00618594 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00618594 ETH


6_deploy_hlr.js
===============

   Deploying 'HLR'
   ---------------
   > transaction hash:    0x98abca445a9b25833f70ab377d4cf35cdf5791b8ca02f59735503d895dfb03fb
   > Blocks: 0            Seconds: 0
   > contract address:    0x19eE33EC0eAC58205f1797303461dEa88BFF93E9
   > block number:        11
   > block timestamp:     1665558192
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.74765404
   > gas used:            5299940 (0x50dee4)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.1059988 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:           0.1059988 ETH

...
```

从上面的输出中，我们可以得到我们部署的智能合约的地址。例如，IdentityContract的合约地址为0xCe69c1DDCcD29a821bB4d3BdEEb3EdE9De9C7903，HFLContract的合约地址为0x7864Bf464F9ecE0D3A95cA55e171D9060cf7336a。

运行完成后，在`build/contracts`目录下可以看到一系列json文件，包括`IdentityContract.json`、`HFLContract.json`、`DataHub.json`、`HLR.json`和`PlonkVerifier3.json`。它们是上述智能合约对应的abi文件。至此智能合约已经部署完成。

### 在区块链浏览器中添加合约信息

部署完智能合约后，为了方便在区块链浏览器中查看合约调用的数据，可以将合约信息添加到区块链浏览器中。
