# Start Chain Connector

Chain Connector is an abstract layer connecting Delta Node and the Blockchain. Chain Connector provides a unique set of APIs to Delta Node regardless of the type of Blockchain system used. And Chain Connector handles the Blockchain transaction signing. In a strict security environment, Chain Connector supports using an isolated signer to keep the private key offline and submit the signature remotely, thus keeping the private key safe.

Chain Connector could be configured to run at "coordinator" mode. At coordinator mode, Blockchain is not required. Chain Connector itself behaves like a center node of the whole network. All the Delta Nodes in the network connect to the same Chain Connector to submit and receive Delta Tasks.

## Start Chain Connector at Coordinator mode

![Delta network structure in coordinator mode](../.gitbook/assets/image%20%281%29.png)

### Get Docker image

At this time the Delta framework is still in an early development stage. Hopefully the release version will come in the next months. And right now we could use the dev tag to get the latest image:

```text
$ docker pull deltampc/delta-chain-connector:dev
```

### Initialization

Before starting the container, some initialization steps should be performed:

Create a directory named `delta_chain_connector` for data and config persistence:

```text
$ mkdir delta_chain_connector
```

Run the following command inside the directory created above to initialize the data structure:

```text
$ cd delta_chain_connector
$ docker run -it --rm -v ${PWD}:/app deltampc/delta_chain_connector:dev init
```

After successful running of the command, a sub directory named `config` should be created.

### Configuration

