# Deploy Smart Contracts

Delta Contracts are written in Solidity, and can be used on any Blockchain that supports EVM.

The code inside [Delta Contracts](https://github.com/delta-mpc/delta-contracts) repository contains the smart contract codes, and the helper codes and tools to deploy them. The helper codes are written in NodeJS. Be sure to have Node and Yarn installed before using the codes.

### Get the Smart Contract Codes

```bash
$ git clone https://github.com/delta-mpc/delta-contracts.git
```

### Install the Dependencies

The dependencies are managed using Yarn:

```bash
$ yarn
```

### Empty the `compile` Folder for a Fresh Compiling and Deployment

The compiled smart contracts are already placed under `compile` folder, which is to be used and referenced by other tools. Before compiling the code, this folder must be cleared.

### Compile and Deploy the Smart Contracts

First the address of the Blockchain node should be specified in the config file which is located at `config/config.js`. The account that will be used to send the transaction should also be filled in:

```bash
ADDRESS=<your eth address>
PRIVATE_KEY=<your eth private key>
WS_URL=<websocket endpoit>
```

Note that the `ADDRESS` and `PRIVATE_KEY` belong to an Ethereum account, which should be created using other tools by developers right now. After the account is created, the developer must transfer some Ethers from Alice's account which we created earlier when [deploying the Delta Chain Node](start-delta-chain-node.md).

`WS_URL` is the WebSocket endpoint of the Delta Chain Node. The IP address will not be `127.0.0.1` if the Delta Chain Node is deployed locally using Docker. if the OS is Windows or Mac, use `host.docker.internal`. And if the OS is Linux, find the IP address of the `docker0` ethernet interface using `ifconfig` command, which is by default `172.17.0.1`.

Finally we can use Yarn to compile and deploy the smart contracts in one command:

```bash
$ yarn deploy
```

After the command is executed successfully, a new `Mpc.json` will show up in the compile folder, which includes the on-chain information about the smart contracts. The deployment of smart contracts is finished then.

### Add the Smart Contracts to the Delta Chain Explorer

For easy debugging, we could attach the source code of the smart contracts to the deployed ones in the Delta Chain Explorer, so that when we check the contract invocation contracts in the explorer, it will have a better visualization.

