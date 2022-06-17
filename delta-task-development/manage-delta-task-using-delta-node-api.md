# Manage Delta Tasks using the Delta Node API

Delta Tasks could be developed in an IDE on the developer's local device, and submitted to the Delta Node remotely using Delta Node API. The computation result, the execution status could also be fetched directly in the code, which makes it possible to embed the Delta PPC Tasks into a larger computation workflow and execute them automatically. In this document, we will show a complete example to write a Delta Task locally and execute it using the Delta Node API.

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

There is another document which describes the writing and executing of the same example using Deltaboard, the web interface of the Delta Network. The doc could be found here:

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
            name="wage_avg",  # The task name used for displaying purpose.
            min_clients=3,  # Minimum nodes required in each round, must be greater than 2.
            max_clients=3,  # Maximum nodes allowed in each round, must be greater or equal than min_clients.
            wait_timeout=5,  # Timeout for calculation.
            connection_timeout=5,  # Wait timeout for each step.
        )

    def dataset(self) -> Dict[str, delta.dataset.DataFrame]:
        """
        Define the data used for analytics.
        return: a dict of which key is the dataset name and value is an instance of delta.dataset.DataFrame
        """
        return {
            "wages": delta.dataset.DataFrame("wages.csv")
        }

    def execute(self, wages: pd.DataFrame):
        """
        Implementation of analytics computation logics.
        The input arguments should be the same as the keys of the returned dict of the dataset method.
        """
        return wages.mean()
```

There are basically 3 parts inside the definition of the analytics task: the task configuration, the selection of the datasets, and the computation logic.

**Task Config**

The task config is given in the `super().__init__()` method. The configs include the task name, the minimum/maximum number of nodes required, and several timeouts.

Since the nodes are not always online in a Delta Network, we must define the required number of nodes in the task config. When the task is published on the network, the nodes will decide to join the task or not depending on their own criteria. When the number of joined nodes meets the requirements defined in the task, the task will start to execute.

We have 3 enterprises in this example, so we set both min and max number to 3 here.

**Datasets**

The datasets to be computed on is defined in the `dataset` method. This method returns a dictionary with its keys as the name of the dataset, and its values as the instances of `delta.dataset.DataFrame`, which will load the dataset and convert it to a `pandas.DataFrame` to be used in the computation logic in the `execute` method of the Delta Task.

The keys in the dictionary will be passed as the arguments' name to the execute method, the values of the arguments will be the corresponding `pandas.DataFrame`.

Note that we are passing a file name to the dataset loader. To find out more about the file names and the file types supported, refer to this document:

{% content-ref url="../system-deployment/prepare-data.md" %}
[prepare-data.md](../system-deployment/prepare-data.md)
{% endcontent-ref %}

In this example, we'll feed the wages data into the task, which are stored in the `wages.csv` files on all the three nodes.

**Computation Logic**

The actual computation logic is implemented in the method `execute`. the arguments of the execute method is given exactly as what is returned in the dataset method. As for now, the type of the arguments will only be `delta.pandas.DataFrame` as this is the only supported format. which is the same as `pandas.DataFrame`. The supported Pandas APIs on `delta.pandas.DataFrame` are listed in this document:

{% content-ref url="../horizontal-federated-analytics/supported-pandas-apis.md" %}
[supported-pandas-apis.md](../horizontal-federated-analytics/supported-pandas-apis.md)
{% endcontent-ref %}

The computation of a average wage is easy in this example. Just use the `mean` method of Pandas on the imported DataFrame.

Now we have fully implemented the task. The second part is to submit the task to a Delta Node, trace its execution, and get the computation result after the execution is done:

```
task = WageAvg().build()

DELTA_NODE_API = "http://127.0.0.1:6700"

delta_node = DeltaNode(DELTA_NODE_API)

task_id = delta_node.create_task(task)  # Submit the task to the Delta Node

if delta_node.trace(task_id):  # Trace the execution of the task
# If the logs are not required, we could use wait：
# if delta_node.wait(task_id):  # Wait for the termination of the task
    res = delta_node.get_result(task_id)  # Get the computation result
    print(res)
else:
    print("Task error")
```

The invocation of `create_task` will submit the task to the specified Delta Node. The method will return a `task_id` that represents the task on the Delta Node.

If we want to monitor the logs during the task execution, we could use `trace` method. This method will keep fetching the logs from the node, and printing them to `stdout` until the termination of the task. If the task finished successfully `trace` will return `True`, otherwise `False`.

If we don't need the logs and just want to continue the next steps with the computation result, we could use `wait`. `wait` is almost the same as `trace` except that no logs will be fetched and printed.

After a successful termination of the Delta Task execution, we could use `get_result` to get the computation result. The returned data type of `get_result` depends on the definition of the task. The data type will be `Dict[str, torch.Tensor]` for federated learning tasks, which is defined in `delta.task.HorizontalLearning.state_dict()`. For analytics tasks, the data type will be the same as returned in `delta.task.HorizontalAnalytics.execute()`.

`get_result` will throw an exception if called before the termination of the task. So do use either `trace` or `wait` to wait till the termination of the task before calling `get_result`.

After the execution of the code above, we should get the following output in our console:

![](<../.gitbook/assets/image (7).png>)

As the figure is shown, the logs about the critical steps during the execution are printed. At the end of the logs, we get the final computation result.

We have finished the whole example successfully by now.

The following are the detailed explanations about the API methods:

