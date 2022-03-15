# 部署智能合约

Delta提供了Solidity语言编写的合约，可以部署在支持EVM虚拟机的区块链上。

智能合约的编译和部署需要通过Nodejs的工具进行。在开始部署以前，需要确认本地安装了Node.js和yarn。

### 下载Delta Contracts源码

Delta Contracts代码库中包含了合约源代码，以及部署合约需要的自动化工具：

```text
$ git clone https://github.com/delta-mpc/delta-contracts.git
```

### 安装依赖

使用`yarn`来安装源代码依赖的其他代码库：

```text
$ yarn
```

### 清空compile文件夹

在`compile`目录下已经包含了编译好的智能合约，用于其他项目引用。在重新编译部署前，需要将此文件夹下的内容清空。

### 编译及部署智能合约

编辑`config/config.js`文件，或者通过环境变量的方式，设置部署智能合约的账号，以及区块链节点的地址：

```text
ADDRESS=<your eth address>
PRIVATE_KEY=<your eth private key>
WS_URL=<websocket endpoit>
```

其中，前两项是部署合约需要用到的以太坊账号和对应的私钥。如果按照前面启动区块链节点章节的配置，启动了单节点的测试网络，这里需要用户自行生成一个新的以太坊地址和私钥，然后从测试网络中默认的Alice账号中转账一些ETH到这个账号中，供其部署智能合约使用。

`WS_URL`是区块链节点的Websocket服务地址，如果区块链节点启动在本地，那么容器中访问宿主机的ip地址不是127.0.0.1，具体取决于操作系统：

* 对于windows和Mac系统，请使用域名host.docker.internal
* 对于linux系统，请使用ifcongfig查看网卡docker0的ip地址（默认为172.17.0.1）

然后先运行下面的命令，命令会编译并部署IdentityContract合约：

```bash
$ yarn deploy IdentityContract
```

IdentityContract是用来进行节点身份管理的合约。
部署完IdentityContract合约后，我们可以看到如下输出，其中包含了IdentityContract的合约地址：

```
IdentityContract 合约已成功部署，地址为: 0x2ec88204a540f506D661f8281d6f7C29e9A949ef
```

（合约地址不是固定的，以实际部署时的输出为准）。

然后我们部署HFLContract：

```bash
yarn deploy HFLContract 0x2ec88204a540f506D661f8281d6f7C29e9A949ef
```

HFLContract是用来进行横向联邦学习的合约。需要注意的是，在部署HFLContract时，我们需要添加IdentityContract的合约地址，作为HFLContract的构造参数。
部署成功后，我们可以看到如下输出：

```
HFLContract 合约已成功部署，地址为: 0x97F109BE334Db4300B20C927838707241Bb46676
```

运行完成后，在`compile`目录下可以看到`IdentityContract.json`文件和`HFLContract.json`，分别为IdentityContract和HFLContract的abi文件。至此智能合约已经部署完成。

### 在区块链浏览器中添加合约信息

部署完智能合约后，为了方便在区块链浏览器中查看合约调用的数据，可以将合约信息添加到区块链浏览器中。


