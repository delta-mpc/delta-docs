# Start the Chain Connector

Chain Connector is the middleware sitting between the Delta Node and the Blockchain. Chain Connector provides the same set of APIs to the Delta Node regardless of the different Blockchain systems in use. Chain Connector also handles the Blockchain transaction signing. In a production  environment where the private key security is critical, a network isolated signer is supported to keep the private key beyond the reach of the online system. The signatures are generated remotely in the signer, and then submitted to the Chain Connector.

Chain Connector could be configured to run at "coordinator" mode. At coordinator mode, Blockchain is not required. Chain Connector itself behaves like a center node of the whole network. All the Delta Nodes in the network connect to this Chain Connector to submit and receive Delta Tasks.

## Coordinator mode (No Blockchain)

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

There is a generated config file `config.json` inside the config folder after initialization. To start the Chain Connector in the coordinator mode, we must set the item `impl` to `monkey`:

```
{
  "log": {
    "level": "debug"
  },
  "impl": "monkey",
  "host": "0.0.0.0",
  "port": 4500,
  "monkey": {
    "db": {
      "type": "sqlite",
      "url": "db/chain.db"
    }
  }
}
```

The Chain Connector in the coordinator mode uses a database to store the "on-chain" data. The database must also be configured. SQLite could be used for simplicity.

### Start the Docker container

We use the command line to start Chain Connector Docker container. The directory we created above must be mounted to the `app` folder inside the container, and `4500` port must be exposed for API connections:

```
$ docker run -d --name=chain_connector -v ${PWD}:/app -p 4500:4500 deltampc/delta-chain-connector:0.3.5 run
```

Now that the container has started, we could use the Docker logs command to check the running status of the container:

```
$ docker logs -f chain_connector
```

## Blockchain mode

Starting the Chain Connector in the chain mode is the same as coordinator mode except for the configuration.

### Configuration

To start the chain connector in the chain mode, we must set the item `impl` to `chain`:

```
{
  "log": {
    "level": "debug"
  },
  "impl": "chain",
  "host": "0.0.0.0",
  "port": 4500,
  "chain": {
    "nodeAddress": "0x6578aDabE867C4F7b2Ce4c59aBEAbDC754fBb990",
    "privateKey": "0xf0f239a0cc63b338e4633cec4aaa3b705a4531d45ef0cbcc7ba0a4b993a952f2",
    "provider": "ws://127.0.0.1:8545",
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
    }
  }
}
```

Then we need to further specify the details about the Blockchain:



The `chain.nodeAddress` means the wallet address compatible with Ethernet, and the `chain.privateKey` means the the private key for the wallet. You can use a wallet app you preferred , such as Metamask, to generate the address and private key.

The `chain.provider` means the URL of the chain node, and it should be a websocket endpoint. The `chain.identity.contractAddress` and `chain.hfl.contractAddress` means identity contract address and hfl contract address, respectively. You can learn how to deploy smart contracts on the chain in deploy-smart-contracts section.

{% content-ref url="system-deployment/start-chain-connector.md" %}
[start-chain-connector.md](system-deployment/start-chain-connector.md)
{% endcontent-ref %}
