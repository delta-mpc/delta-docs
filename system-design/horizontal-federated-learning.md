# Horizontal Federated Learning

## The Algorithm for Horizontal Federated Learning \(HFL\)

The algorithm used for HFL in Delta is from the classic paper of Google. Please read the original paper if you want to understand the details of the algorithm:

[Practical Secure Aggregation for Privacy-Preserving Machine Learning](https://eprint.iacr.org/2017/281.pdf)

Beyond the full implementation of this algorithm, whose name is `FaultTolerantFedAvg` inside Delta Task framework, a simplified version is also provided as `FedAvg`, which as the name indicated, removed the fault tolerant part that dealing with computation client offline. The algorithms are interchangeable in the configuration of a Delta Task.

##  The Workflow of the HFL Task



