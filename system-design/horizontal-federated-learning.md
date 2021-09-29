# Horizontal Federated Learning

## The Algorithm for Horizontal Federated Learning \(HFL\)

The algorithm used for HFL in Delta is from the classic paper of Google. Please read the original paper if you want to understand the details of the algorithm:

[Practical Secure Aggregation for Privacy-Preserving Machine Learning](https://eprint.iacr.org/2017/281.pdf)

Beside the full implementation of this algorithm, whose name is `FaultTolerantFedAvg` in the Delta Task framework, a simplified version is also provided as `FedAvg`, which as the name indicates, removed the fault tolerant part that dealing with computation client offline. The algorithms are interchangeable in the configuration of a Delta Task. The process we described below applies to the full fault tolerant version.

##  The Coordination Process of the HFL Task

The role of the Blockchain in the Delta Network is to enforce the settlement of the computation trading.  The Delta Task developer sends a task to the network, together with the price he is willing to pay for the computation result. Sufficient amount of money, in the form of tokens, must be locked on the Blockchain account, and authorized to be transferred by the smart contract.

The verification of the computation result is implemented with Zero-Knowledge Proofs, which ideally could be executed by the smart contract. After the computation result, together with the corresponding ZKP is submitted to the smart contract by the data holders' Delta Nodes, the Blockchain verifies the computation result using ZKP algorithm. If the verification passes, the money is transferred to the data holders' accounts as the payment for the data and the computation power.

The functions of Blockchain + ZKP in this process can be summarized as:

1. the automatic verification of computation result without seeing data.
2. the atomic exchange of computation result and payment.

which ensures the trading for the computation result can be executed successfully. No one could cheat with each other during the trading process, such as getting the result without paying.

Of course we can implement the same trading process using a middleman \(a trusted third party\) instead of the Blockchain, especially when ZKP could still be used to ensure that the middleman don't need to see the original data. However, the middleman could conspire with the data holders, providing random results to the task developers without the need of real computation and data. The Blockchain is still safer in terms of transparency by opening the execution of rules to everyone and the execution is enforced decentralized by consensus algorithms.

When dealing with the regulation and fitting the system into a traditional financial environment, the enforcement of payment could still be done offline, using the fiat money, by the middleman. The Blockchain could be "downgraded" to an audit system that provides enough evidence to prove the correctness of everything during the trading. For example, the middleman could utilize the Blockchain data to proof that he is honestly executing the rules.

At current stage, Delta doesn't have direct on-chain settlement designs but rather focuses on the fundamental execution workflow design, to have all the critical data related to the settlement recorded on-chain. Some data resides in the coordination process, such as which node joined the task, whether it has submitted the result in the end, etc. The relevant coordination is then done using Blockchain smart contracts to ensure the recorded process is immutable. Other than that, the coordination is done using second layer communication network established between Delta Nodes, which is a lot faster for task execution.

There're 5 steps to execute a HFL task: Task Creation, Candidate Selection for a Round, Private Key Exchange & Secret Sharing, Local Training and Secure Aggregation. We will use sequential graphs to explain each step in detail below:

### Task Creation

![Delta - HFL - Task Creation](../.gitbook/assets/b5d17161d77c3a1e27682db9df05923.png)

After the task developer finishes the coding of a Delta Task, he sends the task to the Delta Chain as the beginning of task execution. The original task codes are not recorded on-chain, only the crypto commitment about the task is registered for later verification of the task content.

To be precise, the content registered on-chain includes:

**The data requirements:** Described in a standard data protocol about the type of data required in the computation task.

**The data/node criteria:** Black/Whitelist of nodes, minimum number of samples required, etc.

**The price for computation/data:** The price willing to pay for each computation step, on each sample.

**The crypto commitment:** Commitment about all the declarations above, plus the computation logic, and task configurations.

The data holders run Delta Node to subscribe to the `Create Task` channel for newly created tasks. If it meets the criteria of the task, and the price is satisfying, the Delta Node will start a new subscription to the `Round Start` channel and wait for the beginning of training rounds.

Note that when decided to join a task, the Delta Node doesn't have to send the application to the Blockchain, instead just silently waits for the next event to occur. This is specifically designed for large scale HFL scenarios, where the computation clients will be mobile phones/edge devices. In these cases the computation clients can not be kept online for a long period. So we use the round as the minimum unit  for participation. The clients only need to keep online for a single round, and can go offline right after it. The next round can be successfully executed without it online.

This design is applicable in Enterprise Delta Node cases as well, where the number of nodes will be much less, with each node holding large amount of samples.

### Candidate Selection for a Round

![Delta - HFL - Candidate Selection for a Round](../.gitbook/assets/f1ec1ace994a49d0e2f49180b2cac23.png)

Round is the minimum unit for task execution. There could be a lot of rounds in a single HFL task, which lasts a super long time period. After every round of computation, we get an updated model that can be directly used.

In each round, the training requires some new randomly selected samples. There're 2 tiers of random selection in Delta, the first is the random selection of participants in each round. The second tier is the random selection of samples on each Delta Node. The first tier is especially useful in mobile phone scenario where the amount of devices is large while each device holding relative less data. The second tier is useful in Enterprise cases, in which we could even select all the nodes in every round, and leave the random selection to be applied to the large amount of data each single node holds.

To start a round, the Delta Node server initiates the contract with the sequence number of the round, and the minimum/maximum number of nodes and samples required in this round. Then the contract sends `Round Start` event to all the candidates. Now is the time for Delta Node clients to send their applications on-chain. After a relatively small period \(in case the clients go offline\), the server gets applications from the Blockchain, and sends back the randomly selected ones.

### Private Key Exchange & Secret Sharing

![Delta - HFL - Private Key Exchange &amp; Secret Sharing](../.gitbook/assets/c74879a461e392e9fee072bb1595421.png)

### 

### Local Training

![Delta - HFL - Local Training](../.gitbook/assets/76b2d1951085e78124648d4d254477e.png)

### 

### Secure Aggregation

![Delta - HFL - Secure Aggregation](../.gitbook/assets/3fe8d87364983671a0076fe414a1ea8.png)

