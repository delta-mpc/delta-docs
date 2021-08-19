# Delta

![Delta Architecture](./assets/arch.png)

Out-of-the-Box Blockchain-powered MPC network

[Homepage](https://deltampc.com) | [Documentation](https://docs.deltampc.com/v/en)

## Overview

Delta performs statistical and machine learning computations jointly on data possessed by a group of data holders while keeping the data private for each of them. Delta is useful in a lot of cases such as risk assessment model training using private data from different banks, or health condition prediction for patients using data from different hospitals.

By encapsulating modern privacy-preserving computation techniques inside, Delta doesn't require users to have any knowledge about privacy-preserving computation. Developers just write computation task using PyTorch and send it to the Delta network. And the rest parts are taken care of by Delta.

Delta network determines the task type by checking the distribution status of the data required by the task, and transforms the task into horizontal/vertical federated learning, or federated analytics task and executes it on the network.

The original data is never accessible to the developers. Delta integrates Blockchain and Zero Knowledge Proof so that the developer could verify that the computation is actually performed as designed on the required data.

## Getting Started

Delta network is composed of several components, and all of them could be easily deployed with Docker. Before choosing the required components and starting the network, take a look at the [System Overview](https://docs.deltampc.com/v/en/system-overview) and get a basic understanding about the network components.

To get your hands dirty and start to play with the network, check the [Getting Started](https://docs.deltampc.com/v/en/getting-started) guide for a quick deployment of the network.

## Join the Community

If you have any questions using the source code, submit issues on [Github](https://github.com/delta-mpc).

To get connected to other developers or want to know more about privacy-preserving computation techniques, join Delta's Slack:

[Join Delta on Slack](https://join.slack.com/t/delta-mpc/shared\_invite/zt-uaqm185x-52oCXcxoYvRlFwEoMUC8Tw)


