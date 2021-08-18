# Start Delta Node

## Start Delta Node by Docker

We recommend to deploy Delta Node by Docker.

### Pull Docker Image

At this time the Delta framework is still in an early development stage. Hopefully the release version will come in the next months. And right now we could use the dev tag to get the latest image:

```text
$ docker pull deltampc/delta-node:dev
```

### Configuration

Delta node need to store some data locally which includes configuration file, user data and logs, etc.. Before starting delta node, we need to config the node first.

Firstly, Create a directory named `delta_node` as the root directory:

```text
$ mkdir delta_node
```

Run the following command inside the directory created above:

```text
$ cd delta_node
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node:dev init
```

This command will create three new sub directories in the root directory, called `config`, `task` and `data`. The `config` directory is for storing configuration file, the `task` directory is for storing result and temporary data of task, and the `data` directory is for storing data provided by the node for task.

### Edit Configuration File

There is a pre-generated configuration file `config.yaml` in the config directory. Before first start, we must edit some configuration item in the configuration file. The first is the `chain_connector.host` which means the address of the Chain Connector described in last section. And the second is the `node_address.host` which means the address of the node itself, and delta node will use this address to register itself in the block chain and other nodes will connect to this address for communication. For example, we can set `node_address.host` to the public ip address of the machine:

```text
---
# chain connector configuration
chain_connector:
  # chain connector address, required
  host: "192.168.1.20"

# address of delta node, will be published in chain and other nodes will connect
# to the address
node_address:
  # public address of the node
  host: "202.120.24.5"
  # port of the node
  port: 6800
```

After completing edition of the configuration file, we can start delta node now.

### Start Docker Container

We use docker to start delta node service, and we need to mount the root directory created in last step to the `app` directory in the docker container. In addition, delta node docker container needs to expose two ports, `6700` and `6800`, for providing API service and communication between nodes.

```text
$ docker run -d --name=delta_node_1 -v ${PWD}:/app -p 6800:6800 -p 6700:6700 deltampc/delta-node:dev
```

We can use `docker logs` command to watch logs of the node to ensure the node is running properly:

```text
$ docker logs -f delta_node_1
```

Delta Node will also write logs to the local directory `log`, and we can watch logs in the `log` directory.

Now the delta node is started, in the next step, we will start the Deltaboard which can manage the node, edit and run the task in GUI:

{% page-ref page="start-deltaboard.md" %}

