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



