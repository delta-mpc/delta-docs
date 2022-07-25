# Horizontal Federated Analytics Task

This is an example of running horizontal federated analytics Delta Task on multiple Delta Nodes.

The task we're about to write is to compute the average wage of all the employees distributed in 3 different enterprises. Each enterprise has a Delta Node deployed, and each Delta Node has access to a `wages.csv` file containing all the wages of the employees of the corresponding enterprise. Obviously `wages.csv` is sensitive to all the enterprises and should always be kept private for each of them. That's where federated analytics is required to compute the overall average wage without revealing to others about the internal wages of each enterprise.

> This example could be executed in the Deltaboard directly, and the complete Jupyter Notebook has already been included in the Deltaboard. Just go to the playground of the Deltaboard, and find this example inside the `examples` folder.
>
> If you don't have a deployed Deltaboard to play with already, use the online version of Deltaboard here:
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. Import the Required Packages

We need to import Delta Task framework components from Python package `delta-task` including `DeltaNode` for Delta Node API connection, and the `HorizontalAnalytics` for defination of the horizontal analytics task:

```python
from delta import pandas as pd
from delta.task import HorizontalAnalytics
from delta.delta_node import DeltaNode
import delta.dataset

from typing import Dict
```

Note that we are replacing the original `Pandas` library with the one imported from `Delta`. By doing so, the code that performs the computation using Pandas keeps the same, however, its actual execution is replaced underneath with privacy-preserving computation in the Delta Network.

### 2. Define the Horizontal Federated Analytics Task

The next step is to define the Delta Task:

```python
class WageAvg(HorizontalAnalytics):
    def __init__(self) -> None:
        super().__init__(
            name="wage_avg",  # The task name which is used for displaying purpose.
            min_clients=3,  # Minimum nodes required in each round, must be greater than 2.
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

### 3. Set the API Address of the Delta Node

After defining the task details, we're ready to run the task on the Delta Network.

`delta-task` library could send the task to the Delta Node directly, as long as the Delta Node API address is specified.

Here we use the Delta Node API provided by the Deltaboard. Deltaboard provides a separate API address for each of its users, the tasks submitted via the API could be listed inside the Deltaboard. The developer could also use API from the Delta Node directly.

Click "Profiles" on the sidebar of the Deltaboard, copy the API Address in the Deltaboard API section, and paste it here:

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

To see the task execution details, go to "Task List" on the sidebar of the Deltaboard, the task should be listed. Click the item to view the execution logs:

![](<../.gitbook/assets/image (8).png>)

We can tell from the logs that during the execution of this task, one round of On-chain Secure Aggregation is performed to get the result. We could also click on the links to view the Blockchain transaction details in the Blockchain explorer. For a complete explanation about the On-chain Secure Aggregation, refer to this document:

{% content-ref url="../system-design/on-chain-secure-aggregation.md" %}
[on-chain-secure-aggregation.md](../system-design/on-chain-secure-aggregation.md)
{% endcontent-ref %}

After the termination of the task. We could click on the download button to get the computation result. The content of the file is a serialized version of what is returned in the `execute` function of the task. The file could be read directly using Python:

![](<../.gitbook/assets/image (7).png>)

The result shows that the average wage of all the three enterprises is 4906. Beyond this number, nothing could be found about the wage of a single employee, or the average wage of a single enterprise.

To understand more about the techniques underneath, start from here:

{% content-ref url="../system-design/horizontal-federated-analytics.md" %}
[horizontal-federated-analytics.md](../system-design/horizontal-federated-analytics.md)
{% endcontent-ref %}
