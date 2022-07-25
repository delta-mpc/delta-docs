# Horizontal Federated Learning Task

This is an example of Delta Task running horizontal federated learning on multiple Delta Nodes.

The dataset is [MNIST](http://yann.lecun.com/exdb/mnist/) distributed on several nodes with each node having only part of the samples. And the task is to train a Convolutional Neural Network to identify hand-writing digits.

> This example could be executed in the Deltaboard directly, and the complete Jupyter Notebook has already been included in the Deltaboard. Just go to the playground of the Deltaboard, and find this example inside the `examples` folder.
>
> If you don't have a deployed Deltaboard to play with already, use the online version of Deltaboard here:
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. Import the Required Packages

The computation logic is written in Torch. So we must import `numpy` and `torch`, and some other helper tools. Then we need to import Delta Task framework components from Python package `delta-task` including `DeltaNode` for Delta Node API connection, the `HorizontalLearning` for defination of the horizontal learning task and the `FaultTolerantFedAvg` for the configuration of the secure aggregation:

```python
from typing import Any, Dict, Iterable, List, Tuple

import logging
import numpy as np
import torch
from PIL.Image import Image
from torch.utils.data import DataLoader, Dataset

from delta.delta_node import DeltaNode
from delta.task.learning import HorizontalLearning, FaultTolerantFedAvg
import delta.dataset
```

### 2. Define the Neural Network Model

Now let's define the CNN model, which is exactly the same as what we will do before:

```python
class LeNet(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = torch.nn.Conv2d(1, 16, 5, padding=2)
        self.pool1 = torch.nn.AvgPool2d(2, stride=2)
        self.conv2 = torch.nn.Conv2d(16, 16, 5)
        self.pool2 = torch.nn.AvgPool2d(2, stride=2)
        self.dense1 = torch.nn.Linear(400, 100)
        self.dense2 = torch.nn.Linear(100, 10)

    def forward(self, x: torch.Tensor):
        x = self.conv1(x)
        x = torch.relu(x)
        x = self.pool1(x)
        x = self.conv2(x)
        x = torch.relu(x)
        x = self.pool2(x)

        x = x.view(-1, 400)
        x = self.dense1(x)
        x = torch.relu(x)
        x = self.dense2(x)
        return x
```

### 3. Define the Horizontal Federated Learning Task

The next step is to define our horizontal federated learning task to train the above model on multiple nodes.

There're several parts in the PPC Task that need to be programmed by the developer:

* _**Task Config**_: We can make some basis task config in the `super().__init__()` method. The configurations involves task name (`name`), training rounds of task (`max_rounds`), the frequency of validation (validate per `validate_interval` round), validate dataset fraction (`validate_frac`) and the aggregate strategy (`strategy`).
* _**Dataset**_: In the `dataset` method, you can specify the dataset for task. You should return an instance of `delta.dataset.Dataset`, and the parameter `dataset` of `delta.dataset.Dataset` represents the dataset name. For detailed explanation of the dataset format, please refer to [this document](https://docs.deltampc.com/network-deployment/prepare-data).
* _**Train dataloader method**_: In the `make_train_dataloader` method, you can specify the dataloader used for training. We will pass the training dataset (an instance of `torch.utils.data.Dataset`) to this method according to the configuration in the `dataset` method, and you can transform the dataset, do some preprocess, etc. And finally you should return a `torch.utils.data.Dataloader`, and it will be used for model training.
* _**Validation dataloader method**_: In the `make_validate_dataloader` method, you can specify the dataloader used for validation. The implementation of this method is very similar with the `make_train_dataloader`, except of the passed dataset is the validation dataset.
* _**Model Training Method**_: In the `train`method, you should define the whole procedure of model training, including forward propagation and backward propagation. The input parameter of this method is the traing dataloader.
* _**Model Validation Method**_: In the `validate` method, you should define the whole procedure of model validation. The input parameter of this method is the validation dataloader, and the return value should be a `dict` of which key should be the validation metrics name, and corresponding value should be the metrics value.
* _**State dict**_: In the `state_dict` method, you can specify all the model parameters need to train and update, and the return value should be a list of these parameters.

```python
def transform_data(data: List[Tuple[Image, str]]):
    """
    Used as the collate_fn of dataloader to preprocess the data.
    Resize, normalize the input mnist image, and the return it as a torch.Tensor.
    """
    xs, ys = [], []
    for x, y in data:
        xs.append(np.array(x).reshape((1, 28, 28)))
        ys.append(int(y))

    imgs = torch.tensor(xs)
    label = torch.tensor(ys)
    imgs = imgs / 255 - 0.5
    return imgs, label


class Example(HorizontalLearning):
    def __init__(self) -> None:
        super().__init__(
            name="example",  # The task name which is used for displaying purpose.
            max_rounds=2,  # The number of total rounds of training. In every round, all the nodes calculate their own partial results, and summit them to the server.
            validate_interval=1,  # The number of rounds after which we calculate a validation score.
            validate_frac=0.1,  # The ratio of samples for validate set in the whole dataset，range in (0,1)
            strategy=FaultTolerantFedAvg(  # Strategy for secure aggregation, now available strategies are FedAvg and FaultTolerantFedAvg, in package delta.task.learning
                min_clients=2,  # Minimum nodes required in each round, must be greater than 2.
                max_clients=3,  # Maximum nodes allowed in each round, must be greater equal than min_clients.
                merge_epoch=1,  # The number of epochs to run before aggregation is performed.
                merge_iteration=0, # The number of iterations to run before aggregation is performed. One of this and the above number must be 0.
                wait_timeout=30,  # Timeout for calculation.
                connection_timeout=10  # Wait timeout for each step.
            )
        )
        self.model = LeNet()
        self.loss_func = torch.nn.CrossEntropyLoss()
        self.optimizer = torch.optim.SGD(
            self.model.parameters(),
            lr=0.1,
            momentum=0.9,
            weight_decay=1e-3,
            nesterov=True,
        )

    def dataset(self) -> delta.dataset.Dataset:
        """
        Define the dataset for task.
        return: an instance of delta.dataset.Dataset
        """
        return delta.dataset.Dataset(dataset="mnist")

    def make_train_dataloader(self, dataset: Dataset) -> DataLoader:
        """
        Define the training dataloader. You can transform the dataset, do some preprocess to the dataset.
        dataset: training dataset
        return: training dataloader
        """
        return DataLoader(dataset, batch_size=64, shuffle=True, drop_last=True, collate_fn=transform_data)  # type: ignore

    def make_validate_dataloader(self, dataset: Dataset) -> DataLoader:
        """
        Define the validation dataloader. You can transform the dataset, do some preprocess to the dataset.
        dataset: validation dataset
        return: validation dataloader
        """
        return DataLoader(dataset, batch_size=64, shuffle=False, drop_last=False, collate_fn=transform_data)  # type: ignore

    def train(self, dataloader: Iterable):
        """
        The training step defination.
        dataloader: the dataloader corresponding to the dataset.
        return: None
        """
        for batch in dataloader:
            x, y = batch
            y_pred = self.model(x)
            loss = self.loss_func(y_pred, y)
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()

    def validate(self, dataloader: Iterable) -> Dict[str, Any]:
        """
        Validation method.
        To calculate validation scores on each node after several training steps.
        The result will also go through the secure aggregation before sending back to server.
        dataloader: the dataloader corresponding to the dataset.
        return: Dict[str, float], A dictionary with each key (str) corresponds to a score's name and the value (float) to the score's value.
        """
        total_loss = 0
        count = 0
        ys = []
        y_s = []
        for batch in dataloader:
            x, y = batch
            y_pred = self.model(x)
            loss = self.loss_func(y_pred, y)
            total_loss += loss.item()
            count += 1

            y_ = torch.argmax(y_pred, dim=1)
            y_s.extend(y_.tolist())
            ys.extend(y.tolist())
        avg_loss = total_loss / count
        tp = len([1 for i in range(len(ys)) if ys[i] == y_s[i]])
        precision = tp / len(ys)

        return {"loss": avg_loss, "precision": precision}

    def state_dict(self) -> Dict[str, torch.Tensor]:
        """
        The params that need to train and update.
        Only the params returned by this function will be updated and saved during aggregation.
        return: List[torch.Tensor]， The list of model params.
        """
        return self.model.state_dict()
```

In the code above, we define a class `Example`, which inherits from a virtual base class `HorizontalLearning`. The class `HorizontalLearning` defines some virtual class method, and user must implement them.

The first method is the constructor `__init__`. The super class' constructor must be called first, in which we could configure the task name (`name`), the number of total rounds to be executed (`max_rounds`), the validation interval (validate per `validate_interval` round), the fraction of validation dataset (`validate_frac`), and the strategy for secure aggregation (`strategy`).

The second method is `dataset`. In `dataset` method, you could define the dataset for task. This method returns a `delta.dataset.Dataset` instance of which parameter is the dataset filename. For detailed explanation of the dataset format, please refer to [this document](https://docs.deltampc.com/network-deployment/prepare-data). At present, the horizontal learning task type only supports one dataset per task, so the `dataset` method can only return one `delta.dataset.Dataset` instance.

Then there are two methods `make_train_dataloader` and `make_validate_dataloader` which defines the train dataloader and validation dataloader, respectively. The parameter of both method is a `torch.utils.data.Dataset` instance, represents train dataset and validation dataset, respectively. In these two methods, you could transform and process the dataset, and finally return a `torch.utils.data.DataLoader` instance which will be passed to `train` and `validate` method.

The next is `train` method which defines the training process. In this method, you should define the whole train process, include the forward propagation, backward propagation and weight updating. The parameter of `train` method is the dataloader defined in the `make_train_dataloader` method.

And then is the `validate` method which defines the vaidation process. In this method, you could test you model's performance on the validation dataset and calculate some metrics. This method should return a dictionary of key is the metrics name and value is the metrics value. The parameter of `validate` method is the dataloader defined in the `make_validate_dataloader` method.

Finally, you should define the parameters for training and updating in the `state_dict` method. The return value of this method is a list of model parameters.

### 4. Set the API Address of the Delta Node

After defining the task details, we're ready to run the task on the Delta Nodes.

Delta Task framework could send the task to Delta Node directly, as long as the Delta Node API address is specified.

Here we use the Delta Node API provided by Deltaboard. Deltaboard provides a separate API address for each of its users, the tasks submitted using the API could be listed inside Deltaboard. The developer could also use API from Delta Node directly.

Click "Profiles" on the sidebar of Deltaboard, copy the API Address in Deltaboard API section, and paste it here:

```python
DELTA_NODE_API = "http://127.0.0.1:6704"
```

### 5. Run the PPC Task

Finally we can start the task:

```python
task = Example().build()
delta_node = DeltaNode(DELTA_NODE_API)
delta_node.create_task(task)
```

### 6. Check the Running Status

After hitting the run button, some logs are printed showing that the task is submitted to the Delta Node successfully.

To see the details of the execution, go to "My Tasks" on the sidebar of Deltaboard, the task should be listed. Click the item to view the execution logs.

In the System Design sections of this document, there're more detailed explanations about the task execution process. For the execution process of the above horizontal federated learning task, please read the following document:

{% content-ref url="../system-design/horizontal-federated-learning.md" %}
[horizontal-federated-learning.md](../system-design/horizontal-federated-learning.md)
{% endcontent-ref %}
