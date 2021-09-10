# Start Delta Chain Explorer

Delta Chain Explorer is a full featured Blockchain explorer based on [Blockscout](https://github.com/blockscout/blockscout), which is compatible with Ethereum's Web3 standard. It has a backend storage to persist block data and the contracts can be registered and saved to optimize the visualization of contract invocation transactions.

## Start Delta Chain Explorer with Docker

The easiest way to start the explorer is using Delta Chain Explorer's Docker image.

### Start a Postgres Database Instance

The explorer needs a Postgres database to persist data. There're mainly two parts of data to be saved: the first is the index data extracted from blocks, which is used to display the block data on the frontend. The second is the smart contracts uploaded by users, which is used to optimize the displaying of contract invocations. On the first starting of the explorer, it will take a longer time to scan all the blocks and build the data required.

If the database is broken, or we switched to a new empty database. The index data could be automatically rebuilt just as the fresh starting of an explorer. However, the contracts uploaded by users are gone forever.

For development purpose, we could start a Postgres instance using Docker, just remember to mount to persistent volume to store the data.

Create root folder for the docker container:

```bash
$ mkdir postgres
```

 Start the container using the following command:

```bash
$ cd postgres
$ docker run -d -v ${PWD}/data:/var/lib/postgresql/data -e PGDATA=/var/lib/postgresql/data/pgdata -e POSTGRES_PASSWORD='1234qwer' postgres:alpine3.14
```

Note that we did three things in this command:

1. Mount data folder into the container to store database files.
2. Tell Postgres to store files at our persistent folder using environment variable.
3. Set the password for the default `postgres` account using environment variable.

### Pull the Image for the Delta Chain Explorer

```bash
$ docker pull deltampc/delta-chain-explorer:dev
```

### Initialization

Create the root folder for the Docker container:

```bash
$ mkdir delta-chain-explorer
```

Create an file for environment variables inside:

```bash
$ cd delta-chain-explorer
$ touch .env
```

And put the following lines in the file:

```bash
DATABASE_URL=postgresql://<db user>:<db password>@<db host>:<db port>/explorer?ssl=false
ETHEREUM_JSONRPC_VARIANT=ganache
ETHEREUM_JSONRPC_HTTP_URL=<RPC endponit url>
ETHEREUM_JSONRPC_TRACE_URL=<RPC endponit url>
ETHEREUM_JSONRPC_WS_URL=<Websocket endponit url>
COIN=DAI
```

Remember to replace the values with the real ones in your environment.

For the database configuration, type in the username, password, hostname and port. If the database is started using the instructions above, the username is `postgres` , and the password is `1234qwer`, the hostname is the IP of the machine running docker, and the port is `5432`.

Note that IP address is not `127.0.0.1`, if the OS is Windows or Mac, use `host.docker.internal`. And if the OS is Linux, find the IP address of the `docker0` ethernet interface using `ifconfig` command, which is by default `172.17.0.1`.

The endpoints of the Delta Chain Node should also be configured. The hostname is as above. The port for RPC is `9933`, and the port for WebSocket is `9944`.

### Start the Docker Container

```bash
$ docker run -d -p 4000:4000 --env-file ./.env deltampc/delta-chain-explorer:dev
```

As the container successfully started, we could visit the explorer's GUI in the browser:

```bash
http://localhost:4000
```

And here's a screenshot:

![](../.gitbook/assets/8aeda9264bfe68184d52f6baf7049e0.png)

