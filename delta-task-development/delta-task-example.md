# 一个示例任务

这是一个使用Delta框架编写的横向联邦学习的任务示例。

数据是分布在多个节点上的[MNIST数据集](http://yann.lecun.com/exdb/mnist/)，每个节点上只有其中的一部分样本。任务是训练一个卷积神经网络的模型，进行手写数字的识别。

> 本示例可以在Deltaboard中直接运行，完整的Jupyter Notebook文件也已经包含在Deltaboard中，进入Deltaboard的Playground，可以在examples文件下看到本示例文件。
>
> 在Deltaboard的在线版本中可以直接查看和运行这个示例
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. 引入需要的包

我们的计算逻辑是用torch写的。所以首先引入`numpy`和`torch`，以及一些辅助的工具，然后从`delta-task`的包中，引入Delta框架的内容，包括`DeltaNode`节点，用于调用API发送任务，以及我们本示例中要执行的横向联邦学习任务`HorizontalTask`等等：

```python
from typing import Dict, Iterable, List, Tuple, Any, Union

import numpy as np
import torch

from delta import DeltaNode
from delta.task import HorizontalTask
from delta.algorithm.horizontal import FedAvg
```

### 2. 定义神经网络模型

接下来我们来定义神经网络模型，这里和传统的神经网络模型定义完全一样：

```python
class LeNet(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = torch.nn.Conv2d(1, 16, 5, padding=2)
        self.pool1 = torch.nn.AvgPool2d(2, stride=2)
        self.conv2 = torch.nn.Conv2d(16, 16, 5)
        self.pool2 = torch.nn.AvgPool2d(2, stride=2)
        self.dense1 = torch.nn.Linear(400, 100)
        self.dense2 = torch.nn.Linear(100, 10)

    def forward(self, x: torch.Tensor):
        x = self.conv1(x)
        x = torch.relu(x)
        x = self.pool1(x)
        x = self.conv2(x)
        x = torch.relu(x)
        x = self.pool2(x)

        x = x.view(-1, 400)
        x = self.dense1(x)
        x = torch.relu(x)
        x = self.dense2(x)
        return x
```

### 3. 定义隐私计算任务

然后可以开始定义我们的横向联邦任务了，用横向联邦学习的方式，在多节点上训练上面定义的神经网络模型

在定义横向联邦学习任务时，有几部分内容是需要用户自己定义的：

* _**模型训练方法**_：包括损失函数、优化器，以及训练步骤的定义
* _**数据预处理方法**_：在执行训练步骤以前，对于加载的每个样本数据进行预处理的方法，具体的参数说明，可以参考[这篇文档](https://docs.deltampc.com/network-deployment/prepare-data)
* _**模型验证方法**_：在每个节点上通过验证样本集，计算模型精确度的方法
* _**横向联邦配置**_：每轮训练需要多少个节点，如何在节点上划分验证样本集合等等

```python
class ExampleTask(HorizontalTask):
    def __init__(self):
        super().__init__(
            name="example", # 任务名称，用于在Deltaboard中的展示
            dataset="mnist", # 任务用到的数据集的文件名，对应于Delta Node的data文件夹下的一个文件/文件夹
            max_rounds=2,  # 任务训练的总轮次，每聚合更新一次权重，代表一轮
            validate_interval=1,  # 验证的轮次间隔，1表示每完成一轮，进行一次验证
            validate_frac=0.1,  # 验证集的比例，范围(0,1)
        )
        
        # 传入刚刚定义的神经网络模型
        self.model = LeNet()
        
        # 模型训练时用到的损失函数
        self.loss_func = torch.nn.CrossEntropyLoss()
        
        # 模型训练时的优化器
        self.optimizer = torch.optim.SGD(
            self.model.parameters(),
            lr=0.1,
            momentum=0.9,
            weight_decay=1e-3,
            nesterov=True,
        )

    def preprocess(self, x, y=None):
        """
        数据预处理方法，会在数据加载时，对每一个样本进行预处理。
        具体的参数说明，可以参考https://docs.deltampc.com/network-deployment/prepare-data
        x: 原始数据集中的一个样本，类型与指定的数据集相关
        y: 数据对应的标签，如果数据集中不包含标签，则为None
        return: 预处理完的数据和标签（如果存在），类型需要为torch.Tensor或np.ndarray
        """
        x /= 255.0
        x *= 2
        x -= 1
        x = x.reshape((1, 28, 28))
        return torch.from_numpy(x), torch.tensor(int(y), dtype=torch.long)

    def train(self, dataloader: Iterable):
        """
        训练步骤
        dataloader: 训练数据集对应的dataloader
        return: None
        """
        for batch in dataloader:
            x, y = batch
            y_pred = self.model(x)
            loss = self.loss_func(y_pred, y)
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()

    def validate(self, dataloader: Iterable) -> Dict[str, float]:
        """
        验证步骤，输出验证的指标值
        dataloader: 验证集对应的dataloader
        return: Dict[str, float]，一个字典，键为指标的名称（str），值为对应的指标值（float）
        """
        total_loss = 0
        count = 0
        ys = []
        y_s = []
        for batch in dataloader:
            x, y = batch
            y_pred = self.model(x)
            loss = self.loss_func(y_pred, y)
            total_loss += loss.item()
            count += 1

            y_ = torch.argmax(y_pred, dim=1)
            y_s.extend(y_.tolist())
            ys.extend(y.tolist())
        avg_loss = total_loss / count
        tp = len([1 for i in range(len(ys)) if ys[i] == y_s[i]])
        precision = tp / len(ys)

        return {"loss": avg_loss, "precision": precision}

    def get_params(self) -> List[torch.Tensor]:
        """
        需要训练的模型参数
        在聚合更新、保存结果时，只会更新、保存get_params返回的参数
        return: List[torch.Tensor]， 模型参数列表
        """
        return list(self.model.parameters())

    def algorithm(self):
        """
        聚合更新算法的配置，可选算法包含在delta.algorithm.horizontal包中
        """
        return FedAvg(
            merge_interval_epoch=0,  # 聚合更新的间隔，merge_interval_epoch表示每多少个epoch聚合更新一次权重
            merge_interval_iter=20,  # 聚合更新的间隔，merge_interval_iter表示每多少个iteration聚合更新一次，merge_interval_epoch与merge_interval_iter互斥，必须有一个为0
            wait_timeout=10,  # 等待超时时间，用来控制等待各个客户端加入的超时时间
            connection_timeout=10,  # 连接超时时间，用来控制聚合算法中，每轮通信的超时时间
            min_clients=2,  # 算法所需的最少客户端数，至少为2
            max_clients=2,  # 算法所支持的最大客户端数，必须大雨等于min_clients
        )

    def dataloader_config(
        self,
    ) -> Union[Dict[str, Any], Tuple[Dict[str, Any], Dict[str, Any]]]:
        """
        训练集dataloader和验证集dataloader的配置，
        每个配置为一个字典，对应pytorch中dataloader的配置
        详情参见 https://pytorch.org/docs/stable/data.html
        return: 一个或两个Dict[str, Any]，返回一个时，同时配置训练集和验证集的dataloader，返回两个时，分别对应训练集和验证集
        """
        train_config = {"batch_size": 64, "shuffle": True, "drop_last": True}
        val_config = {"batch_size": 64, "shuffle": False, "drop_last": False}
        return train_config, val_config

```

### 4. 指定执行任务用的Delta Node的API

定义好了任务，我们就可以开始准备在Delta Node上执行任务了。

Delta Task框架可以直接调用Delta Node API发送任务到Delta Node开始执行，只要在任务执行时指定Delta Node的API地址即可。

Deltaboard提供了对于Delta Node的API的封装，为每个用户提供了一个独立的API地址，支持多人同时使用同一个Delta Node，并且能够在Deltaboard中管理自己提交的任务。 在这里，我们使用Deltaboard提供的API来执行任务。如果用户自己搭建了Delta Node，也可以直接使用Delta Node的API。

在Deltaboard导航栏中进入“个人中心”，在Deltaboard API中，复制自己的API地址，并粘贴到下面的代码中：

```python
DELTA_NODE_API = "http://127.0.0.1:6704"
```

### 5. 执行隐私计算任务

接下来我们可以开始运行这个模型了：

```python
if __name__ == "__main__":
    task = ExampleTask()

    delta_node = DeltaNode(DELTA_NODE_API)
    delta_node.create_task(task)
```

### 6. 查看执行状态

点击执行后，可以从输出的日志看出，任务已经提交到了Delta Node的节点上。

接下来，可以从左侧的导航栏中，前往“任务列表”，找到刚刚提交的任务，点击进去查看具体的执行日志了。

