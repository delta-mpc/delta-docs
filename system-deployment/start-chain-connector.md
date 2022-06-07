# Start the Chain Connector

Chain Connector is the middleware sitting between the Delta Node and the Blockchain. Chain Connector provides the same set of APIs to the Delta Node regardless of the different Blockchain systems in use. Chain Connector also handles the Blockchain transaction signing. In a production  environment where the private key security is critical, a network isolated signer is supported to keep the private key beyond the reach of the online system. The signatures are generated remotely in the signer, and then submitted to the Chain Connector.

Chain Connector could be configured to run at "coordinator" mode. At coordinator mode, Blockchain is not required. Chain Connector itself behaves like a center node of the whole network. All the Delta Nodes in the network connect to this Chain Connector to submit and receive Delta Tasks.

## Start Chain Connector at the coordinator mode

![Delta network structure in coordinator mode](<../.gitbook/assets/image (2).png>)

### Get the Docker image

Use the release tag to get the latest docker image:

```
$ docker pull deltampc/delta-chain-connector:0.3.5
```

### Initialization

Create a directory named `delta_chain_connector` for data and config persistence:

```
$ mkdir delta_chain_connector
```

Run the following command inside the directory created above to initialize the data structure:

```
$ cd delta_chain_connector
$ docker run -it --rm -v ${PWD}:/app deltampc/delta_chain_connector:0.3.5 init
```

After successful running of the command, a sub directory named `config` should be created.

### Configuration

There is a generated config file `config.json` inside the config folder after initialization. To start chain connector at the coordinator mode, we must set the item `impl` to `monkey`:

```
---
# Running mode
impl: "monkey"
```

### Start the Docker container

We use the command line to start Chain Connector Docker container. The directory we created above must be mounted to the `app` folder inside the container, and `4500` port must be exposed for API connections:

```
$ docker run -d --name=chain_connector -v ${PWD}:/app -p 4500:4500 deltampc/delta-chain-connector:0.3.5 run
```

Now that the container has started, we could use the Docker logs command to check the running status of the container:

```
$ docker logs -f chain_connector
```

## Start Chain Connector at Chain mode

Starting chain connector at chain mode is the same as at coordinator mode except for the configuration.

### Configuration

To start chain connector at chain mode, we must set the item `impl` to `chain`:

```
---
# Running mode
impl: "chain"

# Blockchain configuration
chain:
  provider: ""
  nodeAddress: ""
  privateKey: ""
  
  identity:
    contractAddress: ""
  hfl:
    contractAddress: ""
```

Then we need to edit these configurations: `chain.nodeAddress`, `chain.privateKey`, `chain.provider`, `chain.identity.contractAddress` and `chain.hfl.contractAddress`.

The `chain.nodeAddress` means the wallet address compatible with Ethernet, and the `chain.privateKey` means the the private key for the wallet. You can use a wallet app you preferred , such as Metamask, to generate the address and private key.

The `chain.provider` means the URL of the chain node, and it should be a websocket endpoint. The `chain.identity.contractAddress` and `chain.hfl.contractAddress` means identity contract address and hfl contract address, respectively. You can learn how to deploy smart contracts on the chain in deploy-smart-contracts section.

{% content-ref url="system-deployment/start-chain-connector.md" %}
[start-chain-connector.md](system-deployment/start-chain-connector.md)
{% endcontent-ref %}
