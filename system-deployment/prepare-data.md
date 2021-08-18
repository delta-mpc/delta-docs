# Prepare Data

There's a huge amount of development work for Data Connector. And it is not ready. Before we can actually put Data Connector into a working state, developers can already put simple data files inside Delta Node to have an end-to-end test running of Delta network.

At the previous document on starting Delta Node, we have created a folder `delta_node` on the host machine to store the container data. After initialization of the container, there is a sub folder named `data` is created inside `delta_node` folder, which is used to store the data files:

![](../.gitbook/assets/image%20%281%29.png)

At this moment, Delta Node supports several types of data structure inside `data` folder, such as csv files and sub folder. For a given Delta Task, when it starts to run on a group of Delta Nodes, it will search for the same file/folder on each of the nodes using the same file name which is coded into the task. So the data files must be put in the exactly same file name and structure on every node.

To ease the first running of the example Delta Task, Delta Node has a built-in command to download the MNIST dataset and randomly delete 2/3 of the samples to simulate the data separation environment of privacy-preserving computation. To download the MNIST dataset, run the following command:

```text
$ docker run -it --rm -v ${PWD}:/app deltampc/delta-node:dev get-mnist
```

## Supported Data Structure

Every single file or sub-folder inside `data` folder represents a whole dataset.

### Dataset represented by a single file

All the samples of the dataset are stored in a same file. Delta Node reads the file and then pass each sample to the `preprocess` function one-by-one before running the computation logic. Supported file formats are listed below:

| File suffix | Content format | Input argument type of `preprocess` function |
| :--- | :--- | :--- |
| `npy` | `numpy.ndarray` 第0维为样本的维度，即`data[0],data[1]...` 表示第0个、第1个...样本 | `numpy.ndarray` 包含单个样本的数据 |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

### Dataset represented by a sub folder



