# Deploy Smart Contracts

Delta Contracts are written in Solidity, and can be used on any Blockchain that supports EVM.

The code inside [Delta Contracts](https://github.com/delta-mpc/delta-contracts) repository contains the smart contract codes. The smart contracts are managed using [Truffle](https://trufflesuite.com/). The helper codes and also included in this repo. Be sure to have NodeJS installed on your local machine before using this repo.

### Get the Smart Contract Codes

```bash
$ git clone --depth 1 --branch v0.5.2 https://github.com/delta-mpc/delta-contracts.git
```

### Install the Dependencies

Install truffle using NPM. Truffle could be installed globally for future use:

```bash
$ npm install -g truffle
```

### Compile and Deploy the Smart Contracts

Edit `truffle-config.js` file to tell Truffle the address of the Blockchain node:

```
module.exports = {
  networks: {
    development: {
      url: <eth url>,
      network_id: "*", // Match any network id
    },
  },
}
```

`<eth url>` is the address of the Blockchain node. The address should be prefixed with the protocol, either `http://` or `ws://`. If the Delta Chain is used here, the recommended protocol is `ws://`.

The the smart contracts could be compiled and deployed automatically using Truffle migration:

```
$ truffle migrate
```

The deployment is done when the following logs are shown:

```
...

2_deploy_identity.js
====================

   Deploying 'IdentityContract'
   ----------------------------
   > transaction hash:    0x3e15146bb32ddfa94dda6d4ae073e198ecabb661d1e41dc34c6429787cda11b0
   > Blocks: 0            Seconds: 0
   > contract address:    0xCe69c1DDCcD29a821bB4d3BdEEb3EdE9De9C7903
   > block number:        3
   > block timestamp:     1653303777
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.97278602
   > gas used:            1164499 (0x11c4d3)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.02328998 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.02328998 ETH


3_deploy_hfl.js
===============

   Deploying 'HFLContract'
   -----------------------
   > transaction hash:    0xd0d223126c28c9dc48aed6714067c30ba14e4dd2740ab3c99fd7af78aadc826a
   > Blocks: 0            Seconds: 0
   > contract address:    0x7864Bf464F9ecE0D3A95cA55e171D9060cf7336a
   > block number:        5
   > block timestamp:     1653303778
   > account:             0x3e26599f1B992b75c7A7aDF6c6EeA8d5179EeAe2
   > balance:             99.89255696
   > gas used:            3984163 (0x3ccb23)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.07968326 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.07968326 ETH

...
```

We could tell from the above logs that 2 contracts are deployed: the Identity contract and the HFL contract. The contract addresses are also given in the logs. We may save the contract addresses for future configuration.

After the compilation, we can also find the ABI files for the contracts under `build/contracts`.&#x20;

### Add the Smart Contracts to the Delta Chain Explorer

For easy debugging, we could attach the source code of the smart contracts to the deployed contracts in the Delta Chain Explorer. When we check the contract invocation messages in the explorer, we could have a better formatting of the messages.
