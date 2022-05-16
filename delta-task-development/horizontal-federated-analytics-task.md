# Horizontal Federated Analytics Task

This is an example of running horizontal federated analytics Delta Task on multiple Delta Nodes.

The data is a csv file distributed on several nodes, and the file content is the wage of  all employee in the company.
And the task is to calculate the average wage of these companies.

> This example could be executed in Deltaboard directly, and the complete Jupyter Notebook has already been included in Deltaboard. Just go to the playground of Deltaboard, and find this example inside the `examples` folder.
>
> If you don't have a deployed Deltaboard to play with already, use the online version of Deltaboard here:
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. Import the Required Packages

We need to import Delta Task framework components from Python package ```delta-task``` including ```DeltaNode``` for Delta Node API connection, and the ```HorizontalAnalytics``` for defination of the horizontal analytics task:

```python
from delta import pandas as pd
from delta.task import HorizontalAnalytics
from delta.delta_node import DeltaNode
import delta.dataset

from typing import Dict
```

### 2. Define the Horizontal Federated Analytics Task

The next step is to define our horizontal analytics learning task to analyze the data on multiple nodes.

There're several parts in the PPC Task that need to be programmed by the developer:

* ***Task Config***: We can make some basis task config in the ```super().__init__()``` method. The configurations involves task name (```name```), minimum client count(```min_clients```), maximum client count(```max_clients```),waiting timeout for calculation (```wait_timeout```)，and connection timeout for each step in the procedure(```connection_timeout```).
* ***Dataset***: In the ```dataset``` method, you can specify the dataset for task. The return value is a dict of which key should be the name of dataset and value should be an instance of ```delta.dataset.DataFrame```; the key of dict should be corresponding to the parameters of the execute method. For detailed explanation of the dataset format, please refer to [this document](https://docs.deltampc.com/network-deployment/prepare-data).
* ***Analytics implementation***: We need to implement the analytics process in the execute method. The input parameters should be the same with the keys of returned dict of ```dataset``` method. Now the type of all parameter can only be ```delta.pandas.DataFrame```. ```delta.pandas.DataFrame``` is similar with ```pandas.DataFrame```,  now it support operator ```+,-,*,/,//,%```, and method```all, any, count, sum, mean, std, var, sem```.

```python
class WageAvg(HorizontalAnalytics):
    def __init__(self) -> None:
        super().__init__(
            name="wage_avg",  # The task name which is used for displaying purpose.
            min_clients=2,  # Minimum nodes required in each round, must be greater than 2.
            max_clients=3,  # Maximum nodes allowed in each round, must be greater equal than min_clients.
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
        Implementation of analytics.
        input should be the same with keys of returned dict of dataset method.
        """
        return wages.mean()
```

In the code above, we define a class `WageAvg`, which inherits from a virtual base class `HorizontalAnalytics`. The class `HorizontalAnalytics` defines some virtual class method, and user must implement them.

The first method is the constructor `__init__`. In the constructor, you can configure the task. The super class constructor must be called first, in which you could configure the task name (`name`), the minimal clients task need (`min_clients`), the maximal clients task need (`max_clients`), waiting timeout (`wait_timeout`, timeout for calculation stage in a round), and connection timeout (`connection_timeout`, timeout for stages except for calculation stage in a round).

The second method is the `dataset` method. In this method, you could define the dataset for task. This method returns a dictionary, of which key is the dataset name and value is a `delta.dataset.DataFrame` instance. The parameter of `delta.dataset.DataFrame` is the dataset filename. At present, the horizontal federated analytics task only supports the `delta.dataset.DataFrame` as the dataset format.

The third and the final method is the `execute` method. You could implement the analytics logical in the execute method. The parameters of this method are corresponding to the keys of dataset dictionary returned by `dataset` method. The parameter type of `execute` method can only be `delta.pandas.DataFrame` now. The `delta.pandas.DataFrame` instance is simliar with the `pandas.DataFrame` instance, it now supports operator `+`, `-`, `*`, `/`, `//`, `%`, and methods `all`, `any`, `count`, `sum`, `mean`, `std`, `var`, `sem`.

### 3. Set the API Address of the Delta Node

After defining the task details, we're ready to run the task on the Delta Nodes.

Delta Task framework could send the task to Delta Node directly, as long as the Delta Node API address is specified.

Here we use the Delta Node API provided by Deltaboard. Deltaboard provides a separate API address for each of its users, the tasks submitted via the API could be listed inside Deltaboard. The developer could also use API from Delta Node directly.

Click "Profiles" on the sidebar of Deltaboard, copy the API Address in Deltaboard API section, and paste it here:

```python
DELTA_NODE_API = "http://127.0.0.1:6704"
```

### 4. Run the PPC Task

Finally we can start the task:

```python
task = WageAvg().build()
delta_node = DeltaNode(DELTA_NODE_API)
delta_node.create_task(task)
```

### 5. Check the Running Status

After clicking the run button, some logs will be print out showing the task is submitted to the Delta Node successfully.

To see the task execution details, go to "My Tasks" on the sidebar of Deltaboard, the task should be listed.
Click the item to view the execution logs.
