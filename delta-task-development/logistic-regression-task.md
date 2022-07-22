# Logistic Regression Task

This article describes an example of a Delta Logistic Regression Task.

The dataset is the [Spector dataset](https://www.statsmodels.org/stable/datasets/generated/spector.html) which is the experimental data on the effectiveness of the personalized system of instruction (PSI) program. The data is stored in a CSV file including 4 columns:

* GPA: whether or not a student's grade improved
* TCUE: test score on economics test
* PSI: participation in program
* Grade: grade point average

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

```python
class SpectorLogitTask(LogitTask):
    def __init__(
        self,
    ) -> None:
        super().__init__(
            name="spector_logit",  # The task name. Only for displaying purpose.
            min_clients=2,  # The min number of the clients required. Must be larger than 2.
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
