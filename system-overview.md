# System Overview

## Network Participants

The participants of the Delta Network could be categorized into 3 classes:

#### Data Owners

Data owners privately own some original data and they don't want the data to be revealed to any other people. Provided that the data will be secure and never exposed to others, data owners receive computation tasks from others, compute them locally and send the result back.

In the context of the Delta Network, data owners deploy Delta Node locally and connect it to their private data. Delta Nodes connect to the network, receive computation tasks, execute them locally on the private dataset, and at the end send the result back, in an securely encrypted format so that no one could decrypt even the original computation result from it. The result could only be used to be mixed with the results from other Delta Nodes to get a clear text statistical value on a wider range of data.

#### Task Developers

Task developers don't possess data themselves. They have some ideas and want to perform computation tasks on other data owners' data. There're usually two types of computation they want to perform: the statistical computation and the machine learning model training. And since they can never have an eye on the original data, they want to be sure that the computation is actually executed as they programmed, and on the real data they required.

In Delta Network, task developers write the computation task codes using the Delta Task framework, send the task to the network through the API that Delta Node provides, and then receive the computation result, together with a zero knowledge proof shows that the computation really happened as required.

To send tasks to the network through a Delta Node, task developers could deploy Delta Nodes themselves, or using one of the Delta Nodes provided by the data owners.

#### Network Deployers

The network deployers are usually able to union several large data owners \(or a lot of small ones\) in a specific industry. Network deployer connects their data by deploying a Delta Network. The data could then be used by other industry participants, or the data owners themself. Since most of the data owners in other industries don't have the ability to deploy Delta Node themself, it is usually network deployer's task to help them with the deployment, the connection and adaptation between the Delta Node and the original data sources. Network deployers are also responsible for the daily maintaining of the system. 

## Network Structure

![Structure of the Delta Network](.gitbook/assets/f7a7fb858f86700fafb4bd2e627a851.png)

Delta Privacy-Preserving Computation Network is a P2P network. Each data owner deploys a node to connect to its private data, all the nodes connect to each other to form the network. All the nodes in the network are completely identical, having the same rights and functions. Nodes could join and quit the network freely with no effects on the whole network.

![Structure of each node in the Delta Network](.gitbook/assets/6757fef6c4a75a599821c5383811b7a.png)

Every node in the Delta Network consists of mainly two parts: the Blockchain node, and the computation node \(the Delta Node\). the Blockchain node and the Delta Node are connected by a Chain Connector, which is an abstract layer to support various of Blockchain systems.

The P2P network is established by all the Blockchain nodes at the beginning. Delta Node connects to the Blockchain node through the Chain Connector, registering its identity on the Blockchain, and then receives tasks from the Blockchain, executes them, and sends the results back.

The will be communications between Delta Nodes as well, thus forming a second layer network. This network is responsible for task details delivery, secret sharing used in MPC, private set intersection, etc.

Deltaboard is the user interface of Delta Node, which gives users the ability to manage the Delta Node using graphic interface. There's also a JupyterLab embedded in to support online editing and debugging of Delta Task codes. Deltaboard supports multi-user access management as well. Each user gets his own Delta Node API so that multiple users could share a single Delta Node instance.

## Delta Chain

