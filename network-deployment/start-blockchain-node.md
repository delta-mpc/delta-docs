# 启动区块链节点

## 启动单节点测试网络

区块链网络是一个P2P的对等网络，网络中的每个节点完全相同。在一个部署了多个Delta Node的Delta网络中，多个Chain Connector也可以连接到同一个区块链节点，而不影响整个网络的功能。因此在本地进行开发测试时，为方便操作，可以只启动一个区块链节点。其功能和多节点网络完全一致。

推荐通过Delta Chain Node的Docker镜像来启动单个节点的测试网络：

### 下载镜像

```text
$ docker pull deltampc/delta-chain:dev
```

### 初始化配置

新建文件夹delta-node，作为节点启动的根目录

```text
$ mkdir delta-node
```

然后进入根目录

```text
$ cd delta-node
```

### 启动单节点

```text
$ docker run -d -p 9944:9944 -p 9933:9933 -v ${PWD}/data:/root/.local --entrypoint ./node --name delta-chain deltampc/delta-chain:dev --dev --ws-external
```

节点启动后，将在本机的9933端口启动rpc服务，9944端口启动websocket服务，并在根目录自动创建文件夹data，用来保存区块数据。

> 如果使用Windows系统，建议使用Powershell运行上述命令，否则可能无法识别命令中的`${PWD}`变量。

### 节点交互

Delta Chain基于[Substrate Frontier](https://github.com/paritytech/frontier)项目开发，节点兼容以太坊web3标准的RPC调用，因此适用于以太坊的区块链浏览器和钱包等应用，都可以直接应用于Delta Chain。

在web浏览器中打开 [Polkadot JS App](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer)，等待页面加载完成后，可以看到本地启动的节点的状态信息。

![](../.gitbook/assets/polkadot-js-app.png)

Polkadot的这个区块链浏览器完全运行于浏览器中，没有后端来存储数据。每次打开页面后，浏览器连接到区块链节点订阅区块更新，然后之后的新区块数据，会被显示出来。但是开始订阅之前的旧区块数据，是无法看到的。如果需要一个全功能的区块链浏览器，可以使用我们基于[Blockscout](https://github.com/blockscout/blockscout)开发的[Delta区块链浏览器](https://github.com/delta-mpc/delta-chain-explorer)，或者其他具有数据存储功能的以太坊浏览器：

{% page-ref page="start-blockchain-explorer.md" %}

**查看余额**

1. 在Polkadot-JS Apps页面进入：开发者--&gt;RPC calls
2. 选择功能模块：eth--&gt;getBalance
3. 输入参数address: 0xcee2b721fc2fcbb3c136effec5d555c9f9c97db1
4. 点击“提交RPC调用”

![](../.gitbook/assets/c6a000b38b07dcf523f32b1422b8c03.png)

可以看到在0xcee2b721fc2fcbb3c136effec5d555c9f9c97db1这个地址有预先设定好的以太余额

**转账**

1. 在Polkadot-JS Apps页面进入：开发者--&gt;RPC 交易
2. 选择账号：Alice
3. 选择功能模块：evm--&gt;call
4. 输入参数

   ```text
   source: 0xcee2b721fc2fcbb3c136effec5d555c9f9c97db1
   target: <Any eth address>
   input: 0x
   value: 1000000000000000000000 // 1000 Ether
   gas_limit: 4294967295
   gas_price: 1
   nonce: <empty>
   ```

5. 点击“提交交易”

![](../.gitbook/assets/de775550fb0fa25b4eb390c87681a06.png)

#### 查看节点log

```text
$ docker logs -f delta-chain
```

#### 停止 & 重启节点

```text
$ docker stop delta-chain // 停止节点
$ docker start delta-chain // 启动节点
$ docker rm delta-chain // 彻底删除容器
```

## 启动多节点网络
