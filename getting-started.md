# Getting Started

Delta network is composed of several components. Based on the functions required, components could be chosen and organized to form different network types. Before the deployment, it is better to take a look at the system architecture overview to have a basic understanding of the network structure and components:

{% page-ref page="system-overview.md" %}

## Minimum Network without Blockchain

A minimum Delta network without Blockchain requires a Chain Connector running in coordinator mode, 2 Delta Nodes, and a Deltaboard connected to one of the Delta Nodes, as shown in the image below:

![](.gitbook/assets/image%20%283%29.png)

### Start the network using All-in-One Docker image

Delta has an All-in-One startup scripts using docker-compose to start the whole network at once.

1.Clone the Github repo:

```text
$ git clone --depth 1 --branch v0.3.0 https://github.com/delta-mpc/delta-all-in-one.git
```

2.Go to the config folder for no-blockchain network:

```text
$ cd delta-all-in-one/no-blockchain
```

3.Start all the container using docker-composer:

```text
$ docker-compose up -d
```

After the downloading of all the Docker images, the service should be started normally. The network now is fully up and running. We can go to Deltaboard to run our first Delta Task:

{% page-ref page="system-deployment/run-delta-task.md" %}

### Start the network using Docker images of the components

1.Start the Chain Connector and configure it to run in coordinator mode:

{% page-ref page="system-deployment/start-chain-connector.md" %}

2.Start 2 instances of Delta Node, and connect them to the Chain Connector:

{% page-ref page="system-deployment/start-delta-node.md" %}

3.Put some test data into each of the Delta Nodes:

{% page-ref page="system-deployment/prepare-data.md" %}

4.Start Deltaboard, and connect it to one of the Delta Nodes:

{% page-ref page="system-deployment/start-deltaboard.md" %}

5.Now the network is fully up and running, let's try to run a computation task:

{% page-ref page="system-deployment/run-delta-task.md" %}

## Minimum Network with Blockchain

A minimum network with Blockchain consists of 3 parties deploying the same set of network components: Delta Chain Node with Delta Contracts deployed, Chain Connector running in Blockchain mode,  Delta Node, and the GUI Deltaboard, the structure is as following:

![](.gitbook/assets/delta-with-multi-chain.png)

The network could be further minimized if started by a single developer. We need only one instance of Deltaboard to control the network. And the Blockchain node could be shared among the 3 Chain Connectors:

![](.gitbook/assets/delta-with-single-chain.png)

### Start the network using All-in-One Docker image

1. Clone the Github repo:

```text
$ git clone --depth 1 --branch v0.3.0 https://github.com/delta-mpc/delta-all-in-one.git
```

2. Go to the folder for blockchain network:

```text
$ cd delta-all-in-one/with-deltachain
```

3. Edit config files

In the folder for blockchain network, we can see some sub directories:

```bash
$ ls
connector1  connector2  connector3  deltaboard  delta-node1  delta-node2  delta-node3  docker-compose.yml
```

The directories `connector1`, `connector2` and `connector3` store config files for delta-chain-connector.
The config file looks like:

```json
$ cat connector1/config/config.json
{
  "log": {
    "level": "info"
  },
  "impl": "chain",
  "host": "0.0.0.0",
  "port": 4500,
  "monkey": {
    "db": {
      "type": "sqlite",
      "url": "db/chain.db"
    }
  },
  "chain": {
    "nodeAddress": "",
    "privateKey": "",
    "provider": "wss://apus.chain.deltampc.com",
    "gasPrice": 1,
    "gasLimit": 4294967294,
    "chainParam": {
      "chainId": 42,
      "name": "delta"
    },

    "identity": {
      "contractAddress": "0x43A6feb218F0a1Bc3Ad9d9045ee6528349572E42"
    },
    "hfl": {
      "contractAddress": "0x3830C82700B050dA87F1D1A60104Fb667227B686"
    }
  }
}
```

In the config file, `chain.provider`, `chain.identity.contractAddress` and `chain.hfl.contractAddress` represent the blockchain node address, the address of Identity Contract and HFL Contract which support for the delta framework, respectively. For convenience, we have created a blockchain node for our delta-chain, and deployed the Identity Contract and HFL Contract on it. You can keep these config items as they are, unless you want to deploy the contracts on another chain compatible with EVM.

In the config file, `chain.nodeAddress` and `chain.privateKey` represent a wallet address compatible with Ethernet and its private key, respectively.
You can a wallet you prefer to, such as Metamask, to generate the wallet address and private key.

Since we need to start three parties, we need to generate three different wallet address and private key, and then set them to `chain.nodeAddress` and `chain.privateKey` in config files in `connector1`, `connector2` and `connector3`. As a wallet need to have some tokens to send transactions to blockchain, you need to query for some tokens for you three wallet. You can join in our [slack](https://join.slack.com/t/delta-mpc/shared\_invite/zt-uaqm185x-52oCXcxoYvRlFwEoMUC8Tw), and query for tokens in the `delta-chain-faucet` channel.

4. Start all the container using docker-composer:

After config files have been edited, you can start the network by:

```text
$ docker-compose up -d
```

After the downloading of all the Docker images, the service should be started normally. The network now is fully up and running. We can go to Deltaboard to run our first Delta Task:

{% page-ref page="system-deployment/run-delta-task.md" %}

### Start the network using Docker images of the components

1.Start the Delta Chain Node. We can start only one node running in test mode, and make both Chain Connectors connect to the same node. Or we can start 2 nodes to run a more complete network:

{% page-ref page="system-deployment/start-delta-chain-node.md" %}

2.To check the Blockchain data more intuitively, we could start a Blockchain explorer. This step is optional:

{% page-ref page="system-deployment/start-delta-chain-explorer.md" %}

3.Deploy the Delta Smart Contracts on Blockchain:

{% page-ref page="system-deployment/deploy-smart-contracts.md" %}

4.Start the Chain Connectors in Blockchain mode:

{% page-ref page="system-deployment/start-chain-connector.md" %}

Then, we could following the steps in the previous section of running no-blockchain network, from step 2 to step 5, to start Delta Node and prepare the data, and finally run a Delta Task.

