# 横向联邦统计任务

这是一个使用Delta框架编写的横向联邦统计的任务示例。

数据是两个csv文件，分布在多个节点上，每个节点上只有其中的一部分样本。任务是计算这两个csv每一列的和。

> 本示例可以在Deltaboard中直接运行，完整的Jupyter Notebook文件也已经包含在Deltaboard中，进入Deltaboard的Playground，可以在examples文件下看到本示例文件。
>
> 在Deltaboard的在线版本中可以直接查看和运行这个示例
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. 引入需要的包

我们需要```delta-task```的包中，引入Delta框架的内容，包括```DeltaNode```节点，用于调用API发送任务，以及用于横向联邦统计任务```HorizontalAnalytics```。


```python
from delta import pandas as pd
from delta.task import HorizontalAnalytics
from delta.delta_node import DeltaNode
import delta.dataset

from typing import Dict
```

### 2. 定义隐私计算任务

然后可以开始定义我们的横向统计任务了，用横向联邦统计的方式，在多节点上统计我们想要的数据。

在定义横向联邦统计任务时，有几部分内容是需要用户自己定义的：

* ***任务配置***: 我们需要在 ```super().__init__()``` 方法中对任务进行配置。 这些配置项包括任务名称（```name```），所需的最少客户端数（```min_clients```），最大客户端数（```max_clients```），等待超时时间（```wait_timeout```，用来控制一轮计算的超时时间），以及连接超时时间（```connection_timeout```，用来控制流程中每个阶段的超时时间）。
* ***数据集***: 我们需要在```dataset```方法中定义任务所需要的数据集。 该方法返回一个字典，键是数据集的名称，需要与execute方法的参数名对应；对应的值是```delta.dataset.DataFrame```实例， 其参数```dataset```代表所需数据集的名称。关于数据集格式的具体细节，请参考[这篇文章](https://docs.deltampc.com/network-deployment/prepare-data)。
* ***统计的逻辑***: 我们需要在execute方法中实现所有统计的逻辑。execute方法的输入需要与```dataset```方法的返回值对应，即一个输入形参，对应```dataset```返回的字典中的一项。目前参数类型只支持```delta.pandas.DataFrame```这一种。```delta.pandas.DataFrame```的方法类似```pandas.DataFrame```，目前支持```+,-,*,/,//,%```等操作符，以及```all, any, count, sum, mean, std, var, sem```这些方法。


```python
class WageAvg(HorizontalAnalytics):
    def __init__(self) -> None:
        super().__init__(
            name="wage_avg",  # 任务名称，用于在Deltaboard中的展示
            min_clients=2,  # 算法所需的最少客户端数，至少为2
            max_clients=3,  # 算法所支持的最大客户端数，必须大雨等于min_clients
            wait_timeout=5,  # 等待超时时间，用来控制一轮计算的超时时间
            connection_timeout=5,  # 连接超时时间，用来控制流程中每个阶段的超时时间
        )

    def dataset(self) -> Dict[str, delta.dataset.DataFrame]:
        """
        定义任务所需的数据。
        return: 字典，键是数据的名字，需要与execute方法中的参数名称对应；值是一个delta.dataset.DataFrame实例。
        """
        return {
            "wages": delta.dataset.DataFrame("wages.csv")
        }

    def execute(self, wages: pd.DataFrame):
        """
        实现具体的统计逻辑。
        输入与dataset方法的返回值对应
        """
        return wages.mean()
```

具体来讲，定义横向联邦统计任务时，我们需要定义一个继承自`HorizontalAnalytics`的类。`HorizontalAnalytics`是一个虚基类，针对横向联邦统计任务，定义了一些虚函数，需要我们来实现。


首先是构造函数`__init__`。构造函数在这里，主要是对任务任务进行一些基础的配置。这些配置项包括任务名称（```name```），所需的最少客户端数（```min_clients```），最大客户端数（```max_clients```），等待超时时间（```wait_timeout```，用来控制一轮计算的超时时间），以及连接超时时间（```connection_timeout```，用来控制流程中每个阶段的超时时间）。

之后是`dataset`方法。`dataset`方法定义任务所需要的数据集。该方法返回一个字典，键是数据集的名称，需要与execute方法的参数名对应；对应的值是```delta.dataset.DataFrame```实例， 其参数```dataset```代表所需数据集的名称。目前，横向联邦统计任务只支持DataFrame这一种数据格式。

最后是`execute`方法。我们需要在`execute`方法中实现所有统计计算的逻辑。execute方法的输入需要与```dataset```方法的返回值对应，即一个输入形参，对应```dataset```返回的字典中的一项。目前参数类型只支持```delta.pandas.DataFrame```这一种。```delta.pandas.DataFrame```的方法类似```pandas.DataFrame```，目前支持```+,-,*,/,//,%```等操作符，以及```all, any, count, sum, mean, std, var, sem```这些方法。


### 3. 指定执行任务用的Delta Node的API

定义好了任务，我们就可以开始准备在Delta Node上执行任务了。

Delta Task框架可以直接调用Delta Node API发送任务到Delta Node开始执行，只要在任务执行时指定Delta Node的API地址即可。

Deltaboard提供了对于Delta Node的API的封装，为每个用户提供了一个独立的API地址，支持多人同时使用同一个Delta Node，并且能够在Deltaboard中管理自己提交的任务。 在这里，我们使用Deltaboard提供的API来执行任务。如果用户自己搭建了Delta Node，也可以直接使用Delta Node的API。

在Deltaboard导航栏中进入“个人中心”，在Deltaboard API中，复制自己的API地址，并粘贴到下面的代码中：

```python
DELTA_NODE_API = "http://127.0.0.1:6704"
```

### 4. 执行隐私计算任务

接下来我们可以开始运行这个模型了：

```python
task = WageAvg().build()

delta_node = DeltaNode(DELTA_NODE_API)
delta_node.create_task(task)
```

### 5. 查看执行状态

点击执行后，可以从输出的日志看出，任务已经提交到了Delta Node的节点上。

接下来，可以从左侧的导航栏中，前往“任务列表”，找到刚刚提交的任务，点击进去查看具体的执行日志了。

在系统详细设计的章节中，有隐私计算任务从发送到Delta Node开始，到执行完毕的详细流程说明，在上文中执行的横向联邦学习任务，可以在下面的文章中详细了解其执行过程：

{% page-ref page="../system-design/horizontal-federated-learning.md" %}

