# Start the Delta Node

## Start the Delta Node using Docker Image

### Pull the Docker Image

Use the latest release tag to pull the docker image:

```
$ docker pull deltampc/delta-node:0.5.3
```

### Initialization

Delta Node stores data locally including configuration file, temporary files generated during task computation, logs, etc. We must create the folder before starting the container.

Create a directory named `delta_node` as the root directory:

```
$ mkdir delta_node
```

Use the built-in command inside Delta Node to create the folder structure, and generate the sample config file. Run the following command inside the root directory we just created:

```
$ cd delta_node
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node:0.5.3 init
```

This command will create three sub directories in the root directory: `config`, `task` and `data`. The `config` directory is for the configuration file, the `task` directory is for temporary computation results, and the `data` directory is used to store private datasets.

### Edit the Configuration File

After initialization, there will be a pre-generated configuration file `config.yaml` in the config directory. Before actually starting the container, some items must be adjusted according to our environment.

`chain_connector.host` must be set to the address of the Chain Connector we just started in the previous section.

`node_address.host` is used by other Delta Nodes to communicate with our node. This address will be publicly registered on the Blockchain. Set this address to a publicly accessible address.

The following is an example of the configuration file:

```
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

### Start the Docker Container

When starting the container, the root directory we created in last step must be mounted to the `app` directory in the docker container. In addition, port `6700` and `6800` must be exposed from the container for the API service and the communication between Delta Nodes.

Using the following command to start the container:

```
$ docker run -d --name=delta_node_1 -v ${PWD}:/app -p 6800:6800 -p 6700:6700 deltampc/delta-node:0.5.3
```

We can use `docker logs` command to watch logs of the node to ensure the node is running properly:

```
$ docker logs -f delta_node_1
```

Delta Node will also write logs to `log` under the root directory. We could find detailed logs of the node in the `log` directory.

The Delta Node is started. Before running computation tasks, we should prepare some datasets for the computation tasks:

{% content-ref url="prepare-data.md" %}
[prepare-data.md](prepare-data.md)
{% endcontent-ref %}

And now we're finally ready to execute some computation tasks. We could start a Deltaboard for a graphic user interface to write tasks and manage the network:

{% content-ref url="start-deltaboard.md" %}
[start-deltaboard.md](start-deltaboard.md)
{% endcontent-ref %}

Or we can use a local IDE to write tasks, and send them to the Delta Node using Delta Node API:

{% content-ref url="../delta-task-development/manage-delta-task-using-delta-node-api.md" %}
[manage-delta-task-using-delta-node-api.md](../delta-task-development/manage-delta-task-using-delta-node-api.md)
{% endcontent-ref %}
