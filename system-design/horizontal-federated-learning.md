# 横向联邦学习

## 横向联邦学习的实现

Delta中的横向联邦学习在横向联邦任务框架下执行：

{% content-ref url="horizontal-task-framework.md" %}
[horizontal-task-framework.md](horizontal-task-framework.md)
{% endcontent-ref %}

横向联邦任务框架对横向的隐私计算任务进行了抽象，所有的横向任务都可被拆分为多个计算单元，每个计算单元包括`select`, `map`, `aggregate`, `reduce`四个步骤。

用户编写的不同的横向计算任务，在真正执行前，都会先由Delta做一次转化，将整个计算过程转化为包含这四个步骤的多个单元，然后再发送到Delta Node进行执行。

本文详细讲解横向联邦学习的任务从用户编写的计算代码，到可被Delta Node执行的多个计算单元的转化过程。

### 横向联邦学习任务的组成与流程

一个常见的机器学习任务，流程通常是这样的：

1. 定义好机器学习模型
2. 从外部加载数据集，将数据集分割为训练集与验证集
3. 对数据进行预处理
4. 对模型进行训练，更新模型的参数
5. 对模型进行验证，测试模型的性能指标
6. 如果模型的性能指标打到要求，任务完成；否则的话，跳转到第4步，进行训练

所以，我们可以把一个机器学习任务分解为以下几个部分：

1. 模型
2. 数据集，包含训练集与验证集
3. 数据预处理
4. 模型训练
5. 模型验证

除了以上几个部分外，我们还需要明确任务的输入与输出。机器学习任务的输入包含两部分， 一是数据集，二是模型初始的权重；输出的是训练好的模型，也就是训练好的模型权重。

我们回到横向联邦学习任务。在我们定义横向联邦学习任务时，也只需要定义好上述的5个部分， 就可以完成机器学习相关的定义。

参考横向联邦学习的代码示例：

{% content-ref url="../delta-task-development/hfl-task-example.md" %}
[hfl-task-example.md](../delta-task-development/hfl-task-example.md)
{% endcontent-ref %}

在横向联邦学习的示例中，除了构造函数，用户还定义了`dataset`，`make_train_dataloader`， `make_validate_dataloader`，`train`，`validate`，`state_dict`这6个方法。这6个方法与之前分解的机器学习任务 部分的对应关系如下：

1. 模型：对应`state_dict`
2. 数据集：对应`dataset`
3. 数据预处理：对应`make_train_dataloader`，`make_validate_dataloader`
4. 模型训练：对应`train`
5. 模型验证：对应`validate`

用户在定义横向联邦学习任务时，只需要定义好这6个方法，就完成了机器学习部分的定义。 那么，接下来Delta要做的就是，将这些用户定义的方法，转化为多个可被执行的计算单元。

### 转化为横向联邦任务计算单元

要生成计算单元，Delta要做的，就是将用户定义的这6个函数，对应到map与reduce操作上来。

首先，我们需要明确，一个横向联邦学习任务，有多少个Round，每个Round，又对应用户定义的哪些部分呢？ 这都是由用户在构造函数中定义的。在构造函数中，参数`max_rounds`定义了任务有多少个Round；而`merge_epoch`和`merge_iteration` 这两个参数，定义了每个Round，对应于机器学习中的多少个epoch，或者多少个iteration。

我们想一下机器学习中的概念，每更新一次模型的权重，对应于一个iteration；每遍历完一次训练集，对应于一个epoch。 整个机器学习任务，会需要很多个epoch。而在横向联邦学习任务中，模型的权重更新分为本地权重更新，和全局权重更新。 本地权重的更新，还是一个iteration进行一次；而全局权重更新，就是一个Round进行一次。所以`merge_epoch`和`merge_iteration` 这两个参数，其实就是定义好了全局权重更新的频率。

全局权重更新和本地权重更新的频率并不是一定相同的。这就带来一个问题，用户编写模型训练代码，也就是`train`方法，只对应于一个epoch的训练， 所以一个Round中，可能需要运行多次`train`方法，也可能只需要运行`train`的一部分就行了。运行多次容易做到，但是如果只需要 运行`train`的一部分，就很难办了。那么要怎么解决这个问题呢？

这其实是通过调整`train`方法的输入参数来实现的。用户在编写代码的时候认为，`train`方法的参数，就是一个完整的数据集。但是 Delta框架在运行过程中，会提前知道一个Round对应了多少个iteration，或者多少个epoch，也就知道一个Round实际上对应了多少数据， 只要对`train`方法的输入参数进行调整，使输入的数据集大小正好等于一个Round对应的数据集大小，就可以保证每个Round只对应一个`train`方法了。

接下来是`validate`。传统的机器学习中，验证的频率也是用户自行控制的，可以每隔一定的epoch验证一次，也可以每隔一定的iteration验证一次。 但是在横向联邦学习中，每个Round才会更新一次全局的模型权重，没有更新全局权重时，验证是没有意义的。所以，在横向联邦学习中，用户需要指定，每隔多少个Round进行一次验证。 在任务的构造函数中，用户可以通过`validate_interval`这个方法，来控制验证的频率：`validate_interval=1`，就表示 每个Round都进行一次validate，以此类推，默认第一个Round结束后，才开始运行第一次`validate`。通过这个参数，就可以知道 在哪一个Round中插入`validate`方法了。

剩下的方法就很简单了，`state_dict`，`dataset`，`make_train_dataloader`与`make_validate_dataloader`，分别对应的模型、数据集和预处理过的数据集， 每个Round中，它们都是不变的，所以只需要在任务开始的第一个Round中执行一次即可。

接下来，我们需要将这6个方法，对应到map与reduce操作中去。在对应的时候，需要遵守这个原则：map操作是在客户端执行的；reduce操作是在服务端执行的； 而map+reduce操作，需要聚合多个客户端的数据。从这个原则出发，我们可以得到：

1. `state_dict`：对应一个map操作，和一个reduce操作。因为在客户端上，需要提取出模型的权重进行安全聚合；在服务端上，需要全局更新模型的权重。
2. `dataset`：对应一个map操作。因为只有在客户端上，才有数据集。
3. `make_train_dataloader`与`make_validate_dataloader`：分别对应一个map操作。因为只有在客户端上，才有数据集。
4. `train`：对应一个map+reduce操作。因为一个Round训练完后，需要把本地的模型权重通过链上安全聚合系统提交，才能进行全局的更新。
5. `validate`：对应一个map+reduce操作。因为一个Round中验证完成之后，得到的只是模型在本地数据的验证结果，想要得到全局数据上的验证结果，还需要聚合多个客户端的结果。

将这些方法与map和reduce对应起来之后，构建任务就比较简单了。我们还是按照构建计算图的做法，先在图上添加`dataset`、`make_train_dataloader`与`make_validate_dataloader` 对应的操作，然后根据有多少个Round、验证的频率，依次添加`train`方法与`validate`对应的操作。然后与横向联邦统计中意义，通过计算图，构建出一个完整的隐私计算任务。

{% content-ref url="horizontal-federated-analytics.md" %}
[horizontal-federated-analytics.md](horizontal-federated-analytics.md)
{% endcontent-ref %}
