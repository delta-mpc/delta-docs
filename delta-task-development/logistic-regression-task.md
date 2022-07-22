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
        定义任务所需的数据。
        输出: 字典，键是数据的名字，需要与preprocess方法中的参数名称对应.
        """
        return {
            "data": delta.dataset.DataFrame("spector.csv"),
        }

    def preprocess(self, data: pandas.DataFrame):
        """
        预处理函数，处理数据集，将其分为特征(x)与标签(y)返回。
        输入：与dataset方法的返回值对应
        输出：特征(x)与标签(y)
        """
        names = data.columns

        y_name = names[3]
        y = data[y_name].copy()  # type: ignore
        x = data.drop([y_name], axis=1)
        return x, y
    
    def options(self):
        """
        可选方法，对逻辑回归任务的训练进行配置。
        输出：字典，逻辑回归任务的训练配置选项。
        """
        return {
            "maxiter": 35,  # 训练最大迭代次数，默认值为35
            "method": "newton",  # 训练方法，目前选项只有"newton"
            "start_params": None,  # 逻辑回归的初始权重，默认值为None。当为None时，会将权重初始化为全0
            "ord": np.inf,  # newton法相关系数。梯度范数的阶
            "tol": 1e-8,  # newton法相关系数。停止训练的容忍度
            "ridge_factor": 1e-10,  # newton法相关系数。岭回归的系数
        }
```
