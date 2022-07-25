# Logistic Regression Task

This article describes an example of the Delta Logistic Regression Task.

The dataset is the [Spector dataset](https://www.statsmodels.org/stable/datasets/generated/spector.html) which is the experimental data on the effectiveness of the personalized system of instruction (PSI) program. The data is stored in a CSV file including 4 columns:

* GPA: grade point average
* TCUE: test score on economics test
* PSI: participation in program
* Grade: whether or not a student's grade improved

And our task is to train a logistic regression model to predict whether the student's grade will be improved given his TCUE, PSI and Grade data.

> This example could be executed in the Deltaboard directly, and the complete Jupyter Notebook has already been included in the Deltaboard. Just go to the playground of the Deltaboard, and find this example inside the `examples` folder.
>
> If you don't have a deployed Deltaboard to play with already, use the online version of Deltaboard here:
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. Import the Required Packages

```python
import numpy as np
import pandas
import delta.dataset
from delta import DeltaNode
from delta.statsmodel import LogitTask
```

The logistics regression task in Delta must be inherited from the `LogitTask` from `delta.statsmodel`. After importing the above packages, we are ready to write the task content:

### 2. Define the Logistic Regression Model

```python
class SpectorLogitTask(LogitTask):
    def __init__(
        self,
    ) -> None:
        super().__init__(
            name="spector_logit",  # The task name. Only for displaying purpose.
            min_clients=3,  # The min number of the clients required. Must be larger than 2.
            max_clients=3,  # The max number of the clients required. Must be larger than min_clients.
            wait_timeout=5,  # The max time to wait for the clients to join a round.
            connection_timeout=5,  # The max time to wait for network connections.
        )

    def dataset(self):
        """
        Define the data used for LR.
        return: a dict with keys as the dataset names and values as the instances of delta.dataset.DataFrame
        """
        return {
            "data": delta.dataset.DataFrame("spector.csv"),
        }

    def preprocess(self, data: pandas.DataFrame):
        """
        Dataset preprocessing.
        Returns the features(x) and labels(y) separately for the next step.
        input：the arguements are the same as the outputs of the dataset method above.
        return：features(x) and labels(y)
        """
        names = data.columns

        y_name = names[3]
        y = data[y_name].copy()  # type: ignore
        x = data.drop([y_name], axis=1)
        return x, y
    
    def options(self):
        """
        (Optional)
        The options of the logistic regression
        return: the option names and values
        """
        return {
            "maxiter": 35,  # The max number of iterations. Defaults to 35.
            "method": "newton",  # The optimization method. The only option is "newton" for now.
            "start_params": None,  # The initial weights. Defaults to None(all zero).
            "ord": np.inf,  # The newton method option. The order of the gradient norm.
            "tol": 1e-8,  # The newton method option. The tolerance of the error in the params for the convergence.
            "ridge_factor": 1e-10,  # The newton method option. The factor of the ridge regression.
        }
```

There are four parts in the definition of the logistic regression task:

**Task Config**

The task config is given in the `super().__init__()` method. The configs include the task name, the minimum/maximum number of nodes required, and several timeouts.

Since the nodes are not always online in a Delta Network, we must define the required number of nodes in the task config. When the task is published on the network, the nodes will decide to join the task or not depending on their own criteria. When the number of joined nodes meets the requirements defined in the task, the task will start to execute.

We have divided the dataset into 3 parts and put them on 3 different nodes.  So we set both min and max number to 3 here.

**Datasets**

The datasets to be computed on is defined in the `dataset` method. This method returns a dictionary with its keys as the name of the dataset, and its values as the instances of `delta.dataset.DataFrame`, which will load the dataset and convert it to a `pandas.DataFrame` to be used in the computation logic in the `execute` method of the Delta Task.

The keys in the dictionary will be passed as the arguments' name to the execute method, the values of the arguments will be the corresponding `pandas.DataFrame`.

Note that we are passing a file name to the dataset loader. To find out more about the file names and the file types supported, refer to this document:

{% content-ref url="../system-deployment/prepare-data.md" %}
[prepare-data.md](../system-deployment/prepare-data.md)
{% endcontent-ref %}

In this example, we will use the spector`.csv` file which is stored on all the three nodes.

**Data Preprocess**

In the `preprocess` method, we should do the preprocessing of the data returned from the `dataset` method. The arguments of the `preprocess` method is the same as what is returned from the `dataset` method. The names of the arguments are the keys of the dictionary, and the values of the arguments are the instances of `pandas.DataFrame`.

We must return the features(`x`) and the labels(`y`) separately from the `preprocess` method. The data type of `x` and `y` could be either `pandas.DataFrame` or `numpy.ndarray`. And `y` must be an 1-dimensional vector.

In this example, we will use the first 3 columns of the `spector.csv` as the features, and the last column `Grade` as the label.

**Logistic Regression Options**

The options for logistics regression are configured in the `options` method. This method returns a dictionary with its keys as the option names and the values as the config values. The available options are all listed in the above example with detailed explanations in the comments.

The `options` method is optional. If not provided, the default values will be used for all the options.

### 3. Set the API Address of the Delta Node

After defining the task details, we're ready to run the task on the Delta Network.

`delta-task` library could send the task to the Delta Node directly, as long as the Delta Node API address is specified.

We will use the Delta Node API provided by the Deltaboard. Deltaboard provides a separate API address for each of its users, the tasks submitted via the API could be listed inside the Deltaboard. The developer could also use the API provided by the Delta Node directly.

Click "Profiles" on the sidebar of the Deltaboard, copy the API Address in the Deltaboard API section, and paste it here:

```python
DELTA_NODE_API = "http://127.0.0.1:6704"
```

