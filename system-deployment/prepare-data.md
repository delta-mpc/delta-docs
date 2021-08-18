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

| File suffix | File content format | Input argument type of `preprocess` function |
| :--- | :--- | :--- |
| `npy` | `numpy.ndarray` the first axis represents the sample, which means `data[0]` is the first sample | `numpy.ndarray` the data of a single sample |
| `npz` | `numpy.ndarray` the first axis represents the sample, which means `data[0]` is the first sample | `numpy.ndarray` the data of a single sample |
| `pt` | `torch.Tensor`the first axis represents the sample | `torch.Tensor` the data of a single sample |
| `csv` | `,`separated table, no title row, each row represents a sample | `pandas.DataFrame`the data of a single sample |
| `tsv` | `\t`separated table, no title row, each row represents a sample | `pandas.DataFrame`the data of a single sample |
| `txt` | `\t`separated table, no title row, each row represents a sample | `pandas.DataFrame`the data of a single sample |
| `xls/xlsx` | Excel table, no title row, each row represents a sample | `pandas.DataFrame`the data of a single sample |

### Dataset represented by a sub folder

If a sub folder is placed under `data` folder, the whole sub folder represents a single dataset, thus we call the sub folder the "dataset folder". The content of the dataset folder could be:

#### Sub folders under the dataset folder

In this scenario, every sub folder under the dataset folder represents a class of the samples, we call it the "class folder". the folder's name is recognized as the class name, and all the files under the class folder are the samples of the same class. Each file represents a **single** sample.

#### Files under the dataset folder

Each file under the dataset folder represents the data of a **single** sample.

The file under both the dataset folder and the class folder contains the data for a **single** sample. Supported file formats are the same as the table listed above, except the content format inside the file should be the corresponding format for a single sample\(remove the sample axis\).

Beside the formats supported above, the files put under a folder, both dataset folder and class folder, could also be images. Most of the [common image formats](https://pillow.readthedocs.io/en/stable/handbook/image-file-formats.html) are supported. When the file is an image, Delta Node loads it using `PIL.Image.open` and pass the result to `preprocess`function.

