# 准备节点数据

Delta Node的Data Connector组件的开发工作量较大，目前仍在开发中。在Data Connector上线以前，我们可以采用简单的数据文件的形式向Delta Node中放置数据，用于测试整个网络的功能。

在之前启动Delta Node的教程中，使用Docker镜像启动Delta Node时，需要创建一个文件夹`delta_node`绑定到Docker容器。当Delta Node执行完初始化后，`delta_node`文件夹中会多出来几个子文件夹，其中一个`data`文件夹，就是用来放置数据文件的：

![](../.gitbook/assets/8b25eb6eb24504b552cae92c6fc6be1.png)

在现阶段，Delta Task支持几种固定的数据格式，比如文件夹、csv文件等。对于同一个训练任务，其需要用到的数据，必须在各个节点上以同样的格式放置。开发者在Delta Task中指定数据的文件名或者文件夹名，当这个Delta Task在网络中的其他节点上运行时，就会从这个节点的data文件夹中寻找同名文件来加载数据。

## Delta Node支持的数据格式

* `.npy`文件，保存`numpy.ndarray`。数据的第0维表示样本，即`data[0]`表示第0个样本
* `.npz`文件，压缩形式的`numpy.ndarray`。目前只支持文件中保存一个`ndarray`。数据的格式与`.npz`文件相同
* `.pt`文件，保存`torch.Tensor`。数据的第0维表示样本，即`data[0]`表示第0个样本
* `.csv`文件，保存表格，使用逗号分隔，且不包含标题栏，表格中的每一行为一个样本。使用`pandas`读取为一个`pandas.DataFrame`
* `.tsv`文件，保存表格，使用`\t`分隔，且不包含标题栏，表格中的每一行为一个样本。使用`pandas`读取为一个`pandas.DataFrame`
* `.txt`文件，保存表格，使用\t分割，且不包含标题栏，表格中的每一行为一个样本。使用`pandas`读取为一个`pandas.DataFrame`
* `.xls`或`.xlsx`文件，保存表格，且不包含标题栏，表格中的每一行为一个样本。使用`pandas`读取为一个`pandas.DataFrame`
* 图片文件，支持大部分常见的图片格式（详情参考[https://pillow.readthedocs.io/en/stable/handbook/image-file-formats.html](https://pillow.readthedocs.io/en/stable/handbook/image-file-formats.html)）。每一张图片作为一个样本。使用`PIL.Image.open`进行读取
* 单层文件夹，文件夹中只包含文件，不包含子文件夹。文件格式支持上述所有的文件格式，每一个文件作为一个样本。
* 双层文件夹，文件夹中只包含子文件夹，不包含文件；子文件夹中只包含文件，不包含文件夹。每个子文件夹对应一个类别，子文件夹名为类别名。文件格式支持上述所有的文件格式，每一个文件作为一个样本。

## Deltaboard中示例任务的数据准备

以Deltaboard中的示例计算任务为例，该示例计算任务使用MNIST数据集执行了一个手写数字识别的神经网络训练任务。如果需要运行这个任务，需要在网络中的每个Delta Node的data文件夹中，放置一个

