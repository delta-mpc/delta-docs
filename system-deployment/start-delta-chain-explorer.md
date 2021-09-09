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



