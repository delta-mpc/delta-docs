# Start Chain Connector

Chain Connector is an abstract layer connecting Delta Node and the Blockchain. Chain Connector provides a unique set of APIs to Delta Node regardless of the type of Blockchain system used. And Chain Connector handles the Blockchain transaction signing. In a strict security environment, Chain Connector supports using an isolated signer to keep the private key offline and submit the signature remotely, thus keeping the private key safe.

Chain Connector could be configured to run at "coordinator" mode. At coordinator mode, Blockchain is not required. Chain Connector itself behaves like a center node of the whole network. All the Delta Nodes in the network connect to the same Chain Connector to submit and receive Delta Tasks.

## Start Chain Connector at Coordinator mode

![Delta network structure in coordinator mode](../.gitbook/assets/image%20%282%29.png)

### Get Docker image

At this time the Delta framework is still in an early development stage. Hopefully the release version will come in the next months. And right now we could use the dev tag to get the latest image:

```text
$ docker pull deltampc/delta-chain-connector:0.3.0
```

### Initialization

Create a directory named `delta_chain_connector` for data and config persistence:

```text
$ mkdir delta_chain_connector
```

Run the following command inside the directory created above to initialize the data structure:

```text
$ cd delta_chain_connector
$ docker run -it --rm -v ${PWD}:/app deltampc/delta_chain_connector:0.3.0 init
```

After successful running of the command, a sub directory named `config` should be created.

### Configuration

There is a generated config file `config.json` inside the config folder after initialization. To start chain connector at coordinator mode, we must set the item `impl` to `monkey`, means the coordinaotr mode:

```text
---
# Running mode
impl: "monkey"
```

### Start Docker container

We use the command line to start Chain Connector Docker container. The directory we created above must be mounted to the `app` folder inside the container, and `4500` port must be exposed for API connections:

```text
$ docker run -d --name=chain_connector -v ${PWD}:/app -p 4500:4500 deltampc/delta-chain-connector:0.3.0 run
```

Now that the container has started, we could use the Docker logs command to check the running status of the container:

```text
$ docker logs -f chain_connector
```

## Start Chain Connector at Chain mode

Starting chain connector at chain mode is the same as at coordinator mode except for the configuration.

### Configuration

To start chain connector at chain mode, we must set the item `impl` to `chain`, means the chain mode:

```text
---
# Running mode
impl: "chain"
```

Then we need to edit these configurations: `chain.nodeAddress`, `chain.privateKey`, `chain.provider`, `chain.identity.contractAddress` and `chain.hfl.contractAddress`.

The `chain.nodeAddress` means the wallet address compatible with Ethernet, and the `chain.privateKey` means the the private key for the wallet.
You can use a wallet app you preferred , such as Metamask, to generate the address and private key.

The `chain.provider` means the URL of the chain node, and it should be a websocket endpoint. The `chain.identity.contractAddress` and `chain.hfl.contractAddress` means identity contract address and hfl contract address, respectively. You can learn how to deploy smart contracts on the chain in deploy-smart-contracts section.

{% page-ref page="system-deployment/start-chain-connector.md" %}
