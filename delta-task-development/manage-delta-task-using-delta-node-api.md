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

In the code above, we define a class `WageAvg`, which inherits from a virtual base class `HorizontalAnalytics`. The class `HorizontalAnalytics` defines some virtual class method, and user must implement them.

The first method is the constructor `__init__`. In the constructor, you can configure the task. The super class constructor must be called first, in which you could configure the task name (`name`), the minimal clients task need (`min_clients`), the maximal clients task need (`max_clients`), waiting timeout (`wait_timeout`, timeout for calculation stage in a round), and connection timeout (`connection_timeout`, timeout for stages except for calculation stage in a round).

The second method is the `dataset` method. In this method, you could define the dataset for task. This method returns a dictionary, of which key is the dataset name and value is a `delta.dataset.DataFrame` instance. The parameter of `delta.dataset.DataFrame` is the dataset filename. At present, the horizontal federated analytics task only supports the `delta.dataset.DataFrame` as the dataset format.

The third and the final method is the `execute` method. You could implement the analytics logical in the execute method. The parameters of this method are corresponding to the keys of dataset dictionary returned by `dataset` method. The parameter type of `execute` method can only be `delta.pandas.DataFrame` now. The `delta.pandas.DataFrame` instance is simliar with the `pandas.DataFrame` instance, it now supports operator `+`, `-`, `*`, `/`, `//`, `%`, and methods `all`, `any`, `count`, `sum`, `mean`, `std`, `var`, `sem`.

The second part is the logic to submit the task to a Delta Node, trace its execution, and get the computation result after the execution is done:

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

