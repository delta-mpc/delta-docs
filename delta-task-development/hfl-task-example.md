# 一个示例任务

这是一个使用Delta框架编写的横向联邦学习的任务示例。

数据是分布在多个节点上的[MNIST数据集](http://yann.lecun.com/exdb/mnist/)，每个节点上只有其中的一部分样本。任务是训练一个卷积神经网络的模型，进行手写数字的识别。

> 本示例可以在Deltaboard中直接运行，完整的Jupyter Notebook文件也已经包含在Deltaboard中，进入Deltaboard的Playground，可以在examples文件下看到本示例文件。
>
> 在Deltaboard的在线版本中可以直接查看和运行这个示例
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. 引入需要的包

我们的计算逻辑是用torch写的。所以首先引入```numpy```和```torch```，以及一些辅助的工具，然后从```delta-task```的包中，引入Delta框架的内容，包括```DeltaNode```节点，用于调用API发送任务，用于横向联邦学习任务```HorizontalLearning```，以及用于配置安全聚合策略的```FaultTolerantFedAvg```等：

```python
from typing import Any, Dict, Iterable, List, Tuple

import logging
import numpy as np
import torch
from PIL.Image import Image
from torch.utils.data import DataLoader, Dataset

from delta.delta_node import DeltaNode
from delta.task.learning import HorizontalLearning, FaultTolerantFedAvg
import delta.dataset
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

* ***任务配置***: 我们需要在 ```super().__init__()``` 方法中对任务进行配置。 
* ***数据集***: 我们需要在```dataset```方法中定义任务所需要的数据集。 
* ***训练集的Dataloader***: 我们需要在```make_train_dataloader```方法中定义训练集的Dataloader。
* ***验证集的Dataloader***: 我们需要在```make_validate_dataloader```方法中定义验证集的Dataloader。 
* ***模型训练***: 在该方法中，定义整个模型的训练过程，包含整个前向传播和后向传播的。该方法的输入是训练集的Dataloader。
* ***模型验证***: 在该方法中，定义整个模型的验证过程。该方法的输入是验证集的Dataloader，输出是一个字典，字典的键是计算出的指标名称，值是对应的指标值。
* ***模型参数***: 我们需要在```state_dict```方法中定义所有需要训练和更新的模型参数。

```python
def transform_data(data: List[Tuple[Image, str]]):
    """
    作为dataloader的collate_fn，用于预处理函数。
    将输入的mnist图片调整大小、归一化后，变为torch.Tensor返回。
    """
    xs, ys = [], []
    for x, y in data:
        xs.append(np.array(x).reshape((1, 28, 28)))
        ys.append(int(y))

    imgs = torch.tensor(xs)
    label = torch.tensor(ys)
    imgs = imgs / 255 - 0.5
    return imgs, label


class Example(HorizontalLearning):
    def __init__(self) -> None:
        super().__init__(
            name="example",  # 任务名称，用于在Deltaboard中的展示
            max_rounds=2,  # 任务训练的总轮次，每聚合更新一次权重，代表一轮
            validate_interval=1,  # 验证的轮次间隔，1表示每完成一轮，进行一次验证
            validate_frac=0.1,  # 验证集的比例，范围(0,1)
            strategy=FaultTolerantFedAvg(  # 安全聚合的策略，可选策略目前包含 FedAvg和FaultTolerantFedAvg，都位于delta.task.learning包下
                min_clients=2,  # 算法所需的最少客户端数，至少为2
                max_clients=3,  # 算法所支持的最大客户端数，必须大雨等于min_clients
                merge_epoch=1,  # 聚合更新的间隔，merge_interval_epoch表示每多少个epoch聚合更新一次权重
                wait_timeout=30,  # 等待超时时间，用来控制一轮计算的超时时间
                connection_timeout=10  # 连接超时时间，用来控制流程中每个阶段的超时时间
            )
        )
        self.model = LeNet()
        self.loss_func = torch.nn.CrossEntropyLoss()
        self.optimizer = torch.optim.SGD(
            self.model.parameters(),
            lr=0.1,
            momentum=0.9,
            weight_decay=1e-3,
            nesterov=True,
        )

    def dataset(self) -> delta.dataset.Dataset:
        """
        定义任务所需要的数据集。
        return: 一个delta.dataset.Dataset
        """
        return delta.dataset.Dataset(dataset="mnist")

    def make_train_dataloader(self, dataset: Dataset) -> DataLoader:
        """
        定义训练集Dataloader，可以对dataset进行各种变换、预处理等操作。
        dataset: 训练集的Dataset
        return: 训练集的Dataloader
        """
        return DataLoader(dataset, batch_size=64, shuffle=True, drop_last=True, collate_fn=transform_data)  # type: ignore

    def make_validate_dataloader(self, dataset: Dataset) -> DataLoader:
        """
        定义验证集Dataloader，可以对dataset进行各种变换、预处理等操作。
        dataset: 验证集的Dataset
        return: 验证集的Dataloader
        """
        return DataLoader(dataset, batch_size=64, shuffle=False, drop_last=False, collate_fn=transform_data)  # type: ignore

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

    def validate(self, dataloader: Iterable) -> Dict[str, Any]:
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

    def state_dict(self) -> Dict[str, torch.Tensor]:
        """
        需要训练、更新的模型参数
        在聚合更新、保存结果时，只会更新、保存get_params返回的参数
        return: List[torch.Tensor]， 模型参数列表
        """
        return self.model.state_dict()
```

具体来讲，当定义横向联邦学习任务时，我们需要定义一个继承自`HorizontalLearning`的类。`HorizontalLearning`类是一个虚基类，针对横向联邦学习任务的流程，定义了一些虚函数，
需要我们来实现。

首先是构造函数`__init__`。构造函数在这里，主要是对任务任务进行一些基础的配置。这些配置项包括任务名称（```name```），任务训练的总轮数（```max_rounds```），执行验证的频率（每 ```validate_interval``` 轮执行一次验证），验证集的比例（```validate_frac```），以及安全聚合的策略（```strategy```）。在自己实现构造函数时，必须调用基类的构造函数，即`super().__init__()`。

之后是`dataset`方法。`dataset`方法定义任务所需要的数据集。该方法返回一个```delta.dataset.Dataset```实例， 其参数```dataset```代表所需数据集的名称。关于数据集格式的具体细节，请参考[这篇文章](https://docs.deltampc.com/network-deployment/prepare-data)。目前横向联邦学习任务，只支持实用一个数据集，所以`dataset`方法只能返回一个`delta.dataset.Dataset`实例

然后是两个定义DataLoader的方法，分别是定义训练集DataLoader的`make_train_dataloader`和定义验证集DataLoader的`make_validate_dataloader`。这两个方法需要实现的逻辑类似，所以放在一起说。这两个方法的输入都是一个`torch.utils.data.Dataset`实例，分别为训练集和验证集；在方法中，可以按照需要对数据集进行变换、预处理等操作。最后，我们需要返回一个```torch.utils.data.Dataloader```实例，它会作为模型训练方法的输入。

然后是训练模型的方法`train`。该方法的输入是在`make_train_dataloader`中定义的`DataLoader`；在该方法中，我们需要实现整个模型的训练过程，包括前向传播，后向传播和参数更新操作。

还有验证模型的方法`validate`。该方法的输入是在`make_validate_dataloader`中定义的Dataloader；在该方法中，可以根据需要，在验证集上，计算模型的各种性能指标；该方法最后返回一个字典，字典的键是模型的指标的名称，对应的值是指标的值。

最后，我们还需要在```state_dict```方法中定义所有需要训练和更新的模型参数，方法的返回值就是这些模型参数的列表。

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
task = Example().build()

delta_node = DeltaNode(DELTA_NODE_API)
delta_node.create_task(task)
```

### 6. 查看执行状态

点击执行后，可以从输出的日志看出，任务已经提交到了Delta Node的节点上。

接下来，可以从左侧的导航栏中，前往“任务列表”，找到刚刚提交的任务，点击进去查看具体的执行日志了。

在系统详细设计的章节中，有隐私计算任务从发送到Delta Node开始，到执行完毕的详细流程说明，在上文中执行的横向联邦学习任务，可以在下面的文章中详细了解其执行过程：

{% page-ref page="../system-design/horizontal-federated-learning.md" %}

