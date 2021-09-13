# An Example Delta Task

This is an example of Delta Task running horizontal federated learning on multiple Delta Nodes.

The dataset is [MNIST](http://yann.lecun.com/exdb/mnist/) distributed on several nodes with each node having only part of the samples. And the task is to train a Convolutional Neural Network to identify hand-writing digits.

> This example could be executed in Deltaboard directly, and the complete Jupyter Notebook has already been included in Deltaboard. Just go to the playground of Deltaboard, and find this example inside the `examples` folder.
>
> If you don't have a deployed Deltaboard to play with already, use the online version of Deltaboard here:
>
> [https://board.deltampc.com](https://board.deltampc.com)

### 1. Import the Required Packages

The computation logic is written in Torch. So we must import `numpy` and `torch`, and some helper tools. Then we need to import Delta Task framework components from Python package `delta-task` including `DeltaNode` for Delta Node API connection and the `HorizontalTask` that we'll be running in this example:

```python
from typing import Dict, Iterable, List, Tuple, Any, Union

import numpy as np
import torch

from delta import DeltaNode
from delta.task import HorizontalTask
from delta.algorithm.horizontal import FedAvg
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

There're several parts in the PPC Task that need to be programed by the developer himself:

* _**Model Training Method:**_ Including what loss function and optimizer are used, and how to perform training steps.
* _**Data Pre-process Method:**_ Before performing training step, there's a function `preprocess` could be used to transform the training data. For the detailed explanation of the arguments, please refer to [this document](https://docs.deltampc.com/network-deployment/prepare-data).
* _**Model Validation Method:**_ How to calculate precision score on each iteration.
* _**Horizontal Federated Learning Config:**_ The minimum/maximum number of nodes required to start an iteration, number of max steps, etc.

```python
class ExampleTask(HorizontalTask):
    def __init__(self):
        super().__init__(
            name="example", # The task name which is used for displaying purpose.
            dataset="mnist", # The file/folder name of the dataset used. The file/folder should be placed under the data folder of all the Delta Nodes.
            max_rounds=2,  # The number of total rounds of training. In every round, all the nodes calculate their own partial results, and summit them to the server.
            validate_interval=1,  # The number of rounds after which we calculate a validation score.
            validate_frac=0.1,  # The ratio of samples for validate set in the whole dataset，range in (0,1)
        )
        
        # Pass in the NN model we just defined
        self.model = LeNet()
        
        # Define the loss function
        self.loss_func = torch.nn.CrossEntropyLoss()
        
        # Define the optimizer
        self.optimizer = torch.optim.SGD(
            self.model.parameters(),
            lr=0.1,
            momentum=0.9,
            weight_decay=1e-3,
            nesterov=True,
        )

    def preprocess(self, x, y=None):
        """
        The data pre-processing method.
        After data loading, every sample is passed through this method to be transformed.
        For the detailed explanation of the input arguments, please refer to https://docs.deltampc.com/network-deployment/prepare-data
        x: a sample from the dataset, the type depends on the data provided.
        y: the label of the sample, None if no label is attached to the sample.
        return: the data and label after processing, the type should be torch.Tensor or np.ndarray
        """
        x /= 255.0
        x *= 2
        x -= 1
        x = x.reshape((1, 28, 28))
        return torch.from_numpy(x), torch.tensor(int(y), dtype=torch.long)

    def train(self, dataloader: Iterable):
        """
        The training step definition.
        dataloader: the dataloader cooresponding to the dataset.
        return: None
        """
        for batch in dataloader:
            x, y = batch
            y_pred = self.model(x)
            loss = self.loss_func(y_pred, y)
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()

    def validate(self, dataloader: Iterable) -> Dict[str, float]:
        """
        Validation method.
        To calculate validation scores on each node after several training steps.
        The result will also go through the secure aggregation before sending back to server.
        dataloader: the dataloader cooresponding to the dataset.
        return: Dict[str, float]，A dictionary with each key (str) corresponds to a score's name and the value (float) to the score's value.
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

    def get_params(self) -> List[torch.Tensor]:
        """
        The params that need to be trained.
        Only the params returned by this function will be updated and saved during aggregation.
        return: List[torch.Tensor]， The list of model params.
        """
        return list(self.model.parameters())

    def algorithm(self):
        """
        Algorithm used to perform result aggregation. All the candidates are included in the package delta.algorithm.horizontal
        """
        return FedAvg(
            merge_interval_epoch=0,  # The number of epochs to run before aggregation is performed.
            merge_interval_iter=20,  # The number of iterations to run before aggregation is performed. One of this and the above number must be 0.
            wait_timeout=10,  # Timeout when waiting for node participating.
            connection_timeout=10,  # Connection timeout in each communication in the aggregation algorithm.
            min_clients=2,  # Minimum nodes required in each round.
            max_clients=2,  # Maximum nodes allowed in each round.
        )

    def dataloader_config(
        self,
    ) -> Union[Dict[str, Any], Tuple[Dict[str, Any], Dict[str, Any]]]:
        """
        the config for dataloaders of training and validating，
        each config is a dictionary corresponding to the dataloader config of PyTorch.
        The details are in https://pytorch.org/docs/stable/data.html
        return: One or two Dict[str, Any]. When returning one dict, it is used for both training and validating dataloader.
        """
        train_config = {"batch_size": 64, "shuffle": True, "drop_last": True}
        val_config = {"batch_size": 64, "shuffle": False, "drop_last": False}
        return train_config, val_config

```

We defined a class for the `DeltaTask`,which inherits from `HorizontalTask`, which is a virtual base class. Some methods must be implemented by the developer before it can be executed on Delta Node.

The first method we need to implement is the constructor `__init__`. The parent's constructor must be called first, in which we could configure the task name, the dataset to be used, the number of total rounds to be executed, the validation interval, and the fraction of samples to be used as the validation set.

After the invocation of the parent's constructor, we could define the model's parameters, such as the loss function and optimizer to be used. 

Then there're 4 more methods to be implemented, as listed below:

* `preprocess`：Data preprocessing before the training step.
* `train`：The training step.
* `validate`：Definition of the validation metrics.
* `get_params`：The parameters to be trained in the training step.

And there're 2 optional methods to be overloaded:

* `algorithm`：The secure aggregation algorithm to be used.
* `dataloader_config`：The `dataloader` config for the training set and the validation set.

The  `algorithm` method is used to define the secure aggregation algorithm. The algorithm candidates to be chosen from resides in the `delta.algorithm.horizontal` package.  2 options are provided for now: `FedAvg` and `FaultTolerantFedAvg`. If `algorithm` method is not overloaded, `FedAvg` is chosen by default.

The other method `dataloader_config` is used to configure the `dataloader` for the data of the training set and the validation set. The configurable items are the same as `PyTorch`, which could be found in its [official document](https://pytorch.org/docs/stable/data.html). The default config is to use the same configuration for both the training set and the validation set `(shuffle:True, batch_size:64, drop_last:True)`.

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
task = ExampleTask()

delta_node = DeltaNode(DELTA_NODE_API)
delta_node.create_task(task)
```

### 6. Check the Running Status

After hitting the run button, some logs are printed showing that the task is submitted to the Delta Node successfully.

To see the details of the execution, go to "My Tasks" on the sidebar of Deltaboard, the task should be listed. Click the item to view the execution logs.

