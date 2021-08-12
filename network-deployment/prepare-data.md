# 准备节点数据

Delta Node的Data Connector组件的开发工作量较大，目前仍在开发中。

Data Connector包含一个对数据进行标准化描述的协议，Delta Task中可以直接使用协议SDK来引用数据。Data Connector支持连接多个不同的数据源，包括文件、关系型数据库、HDFS等，并支持对数据源的数据进行自动校验和清洗。Delta Node接收到计算任务后，判断自身的数据是否满足计算条件并上报给任务发起方，任务发起方根据各个节点的数据持有情况来决定采用哪种隐私计算策略。

在Data Connector上线以前，我们可以采用简单的数据文件的形式向Delta Node中放置数据，用于测试整个网络的功能。

使用Delta Node的Docker镜像启动Delta Node时，需要创建一个文件夹绑定到Docker容器，如果按照上一节中的教程启动了Delta Node，则这个文件夹就是`delta_node`。当Delta Node执行完初始化后，`delta_node`文件夹中会多出来几个子文件夹，其中一个`data`文件夹，就是用来放置数据文件的：

![](../.gitbook/assets/8b25eb6eb24504b552cae92c6fc6be1.png)

在现阶段，Delta Task支持几种固定的数据格式，比如文件夹、csv文件等。对于同一个训练任务，其需要用到的数据，必须在各个节点上以同样的格式放置。开发者在Delta Task中指定数据的文件名或者文件夹名，当这个Delta Task在网络中的其他节点上运行时，就会从这个节点的data文件夹中寻找同名文件来加载数据。

### Delta Node支持的数据格式

### Deltaboard中示例任务的数据准备

以Deltaboard中的示例计算任务为例，该示例计算任务使用MNIST数据集执行了一个手写数字识别的神经网络训练任务。如果需要运行这个任务，需要在网络中的每个Delta Node的data文件夹中，放置一个





