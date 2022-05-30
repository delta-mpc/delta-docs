# 使用Delta Node API管理任务

通常，我们使用Deltaboard，在浏览器中进行任务的管理。
但是，部署Deltaboard比较复杂。为了方便用户，我们在delta包中提供了对Delta Node API的封装，
用户可以不使用Deltaboard，直接在代码中对任务进行管理。

下面，我们以横向联邦统计为例子，看看如何在代码中对任务进行管理。

## 例子

```python
from typing import Dict

import delta.dataset
from delta import pandas as pd
from delta.delta_node import DeltaNode
from delta.task import HorizontalAnalytics


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


task = WageAvg().build()

DELTA_NODE_API = "http://127.0.0.1:6700"

delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)  # 创建任务
if delta_node.trace(task_id):  # 跟踪任务日志
# 如果不需要跟踪任务日志，也可以这么写：
# if delta_node.wait(task_id):  # 等待任务结束
    res = delta_node.get_result(task_id)  # 获取任务结果
    print(res)
else:
    print("Task error")
```

这个例子与

{% page-ref page="hfa-task-example.md" %}

中的例子相比，不同的地方在于：

```python
task_id = delta_node.create_task(task)  # 创建任务
if delta_node.trace(task_id):  # 跟踪任务日志
# 如果不需要跟踪任务日志，也可以这么写：
# if delta_node.wait(task_id):  # 等待任务结束
    res = delta_node.get_result(task_id)  # 获取任务结果
    print(res)
else:
    print("Task error")
```

在这里，我们通过`create_task`创建任务之后，可以获得一个任务ID（`task_id`）。
后续，我们可以通过这个任务ID来对任务进行管理。

如果我们需要获取任务日志，那么可以使用`trace`这个API。`trace`会持续地在`stdout`中
打印任务日志，直至任务正常结束或发生异常。`trace`会返回一个`bool`值，如果任务正常结束，
则返回值为`True`，否则为`False`。

如果我们不需要任务日志，只需要等待任务结束，那么可以使用`wait`这个API。`wait`的用法
和返回值与`trace`一致，区别只是在于，它不会打印任务日志。

如果任务正常结束，那么我们可以通过`get_result`这个API来获取任务的结果。任务结果的类型
由任务类型来决定。横向联邦学习任务，结果的类型是Dict[str, torch.Tensor]，
即delta.task.HorizontalLearning.state_dict()的返回类型；
横向联邦统计任务，结果的类型是delta.task.HorizontalAnalytics.execute()的返回类型（delta.pandas对应pandas）。

如果在任务没有结束或异常退出的情况下，调用`get_result`，会抛出异常。所以调用`get_result`前，一定要使用
`trace`或`wait`，等待任务正常结束。

如果任务发生异常，那么使用`trace`，可以在日志中看到发生的异常。

## Delta Node API 

### create_task - 创建任务

```python
delta.delta_node.DeltaNode.create_task(self, task)
```

用户可以通过该方法将任务提交到Delta Node上，创建任务。

参数：

* task: delta.core.task, 需要提交的任务，由用户编写的任务经过build生成

返回值：

* 任务ID，int类型

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
```

### trace - 跟踪任务日志

```python
delta.delta_node.DeltaNode.trace(self, task_id)
```

用户可以通过该方法，跟踪已经创建的任务的日志。该方法是阻塞方法，会持续打印任务的日志，直至任务正常结束或异常退出。

参数：

* task_id: 已经提交的任务ID。可以通过create_task方法得到

返回值：

* 任务状态，bool类型，True表示任务执行成功，False表示任务执行过程中出现异常

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
delta_node.trace(task_id)
```

命令行输出：

```
2022-05-24 03:43:06  start run task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a
2022-05-24 03:43:06  [Create Task] create task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a
...
...
...
2022-05-24 03:43:27  task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a finish
```

### wait - 等待任务结束

```python
delta.delta_node.DeltaNode.wait(self, task_id)
```

用户可以通过该方法，等待任务结束。该方法是阻塞方法，会阻塞直至任务正常结束或异常退出。

参数：

* task_id: 已经提交的任务ID。可以通过create_task方法得到

返回值：

* 任务状态，bool类型，True表示任务执行成功，False表示任务执行过程中出现异常

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
delta_node.wait(task_id)
```

### get_result - 获取任务结果

```python
delta.delta_node.DeltaNode.get_result(self, task_id)
```

用户可以通过该方法，在任务正常结束后，获取任务结果。如果任务没有结束或执行过程中出现异常，调用此方法会抛出异常。

参数：

* task_id: 已经提交的任务ID。可以通过create_task方法得到

返回值：

* 任务结果。横向联邦学习任务，结果的类型是Dict[str, torch.Tensor]，即delta.task.HorizontalLearning.state_dict()的返回类型；横向联邦统计任务，结果的类型是delta.task.HorizontalAnalytics.execute()的返回类型（delta.pandas对应pandas）。

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
if delta_node.wait(task_id):
    res = delta_node.get_result(task_id)
    print(res)
else:
    print("Task error")
```