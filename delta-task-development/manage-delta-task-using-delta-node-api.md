# Manage Delta Tasks using the Delta Node API

Delta Tasks could be developed in an IDE on the developer's local device, and submitted to the Delta Node using Delta Node API. The computation result, the execution status could also be fetched directly in the code, which makes it possible to embed the Delta PPC Tasks into a larger computation workflow and execute them automatically. In this document, we will show a complete example to write a Delta Task locally and execute it using the Delta Node API.

The task we're about to write is a federated analytics task that computes the average wage of all the employees distributed in 3 different enterprises. Each enterprise has a Delta Node deployed, and each Delta Node has access to a `wages.csv` file containing all the wages of the employees of the corresponding enterprise. Obviously `wages.csv` is sensitive to all the enterprises and should always be kept private for each of them. That's where federated analytics is required to compute the overall average wage without revealing to others about the internal wages of each enterprise.

The Python library `delta-task` implements the relevant methods to call Delta Node APIs remotely. We must install the library in the local Python environment before running the task:

```
$ pip install delta-task
```

The methods involved in managing Delta Tasks in `delta-task` library are listed below:

* `create_task`: submit the Delta Task to Delta Node.
* `trace`: keep fetching the logs from Delta Node and printing them to the console during the execution of a Delta Task.
* `wait`: block the current code execution till the Delta Task finishes running on the Delta Node.
* `get_result`: get the computation result of a Delta Task from the Delta Node.

There is another document describing the writing and executing of the same example using Deltaboard, the web interface of Delta Network. The doc could be found here:

{% content-ref url="horizontal-federated-analytics-task.md" %}
[horizontal-federated-analytics-task.md](horizontal-federated-analytics-task.md)
{% endcontent-ref %}

Now let's start to write the code in our local IDE. The first part is the definition of the Delta Task:

```
from typing import Dict
import delta.dataset
from delta import pandas as pd
from delta.delta_node import DeltaNode
from delta.task import HorizontalAnalytics


class WageAvg(HorizontalAnalytics):
    def __init__(self) -> None:
        super().__init__(
            name="wage_avg",  # 任务名称，用于在Deltaboard中的展示
            min_clients=3,  # 算法所需的最少客户端数，至少为2
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

