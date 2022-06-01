# 执行计算任务

在Delta隐私计算网络搭建完成后，我们就可以开始编写并执行一个Delta Task计算任务了。

Delta Task的开发可以在任意支持Python语言的IDE中进行。为了方便使用，Deltaboard中已经集成了JupyterHub，也就是支持多用户同时使用的JupyterLab，是在数据分析领域非常广泛使用的IDE，可以直接在线编辑代码，并实时显示执行结果。

Deltaboard的JupyterLab中已经放置了横向联邦学习和横向联邦统计任务的示例。启动Deltaboard的界面，进入JupyterLab的Tab，打开`examples`文件夹，可以看到4个示例文件

![](../.gitbook/assets/playground.png)

其中，`en-horizontal-analytics-task.ipynb`和`zh-horizontal-analytics-task.ipynb`对应横向联邦统计任务，`en-horizontal-learning-task.ipynb`和`zh-horizontal-learning-task.ipynb`对应横向联邦学习任务。

下面我们以横向联邦学习任务为例子，演示如何在Deltaboard中执行计算任务。横向联邦学习示例的任务是一个通过神经网络进行手写数字识别的模型训练。需要用到手写数字的公开数据集[MNIST](http://yann.lecun.com/exdb/mnist)，Delta Node的Docker镜像提供了下载MNIST数据集的功能，并且仅随机保留1/3的样本数据以模拟多节点拥有不同数据的隐私计算场景，详情可参考：

{% content-ref url="prepare-data.md" %}
[prepare-data.md](prepare-data.md)
{% endcontent-ref %}

在准备好MIST数据后，可以开始运行Deltaboard中的示例代码。

点击JupyterLab中的运行按钮，即可运行示例中指定的代码块。根据示例文件中的提示修改代码，并依次运行示例文件中的代码块，
就可以成功地创建任务。如果任务创建成功，最后一个代码块下方会显示任务的ID

![](../.gitbook/assets/submit.png)

然后在左侧的Tab栏中选择"任务列表"，可以看到刚刚提交的任务，已经进入了执行中的状态：

![](../.gitbook/assets/task-list.png)

这里的任务列表，如果是非管理员的普通用户登录，只能看到自己提交的计算任务。如果是管理员登录的话，可以看到全部Delta Node上的任务，包括所有用户通过本Delta Node提交的任务，以及其他Delta Node提交，在本Delta Node完成本地计算的任务。

点击任务条目，进入任务详情页，可以看到任务执行的详细Log：

![](../.gitbook/assets/task-detail.png)

从执行Log可以看出，这个任务以横向联邦学习的方式，被分发给了网络中的全部节点并完成了多轮的训练。每轮训练中，各个节点完成本地计算后，上报了经过安全聚合层处理的梯度向量，安全聚合层的处理，保证了从这个上报的数据中，无法反推出本节点计算出的原始梯度向量值，只有当所有节点上报的数据进行相加后，才能得到明文的全部梯度平均值。我们的发起方Delta Node收到全部节点的统计结果后，求和得到了最终的训练模型。至此我们已经完成了第一个Delta Task的编写和全网运行。

在开始真正编写自己的计算任务以前，开发者还需要对Delta Task框架有更深入的了解。如果想要了解横向联邦学习，可以参考这个示例任务：

{% content-ref url="../delta-task-development/hfl-task-example.md" %}
[hfl-task-example.md](../delta-task-development/hfl-task-example.md)
{% endcontent-ref %}

如果想要了解横向联邦统计，可以参考这个示例任务：

{% content-ref url="../delta-task-development/hfa-task-example.md" %}
[hfa-task-example.md](../delta-task-development/hfa-task-example.md)
{% endcontent-ref %}

如果不想使用Deltaboard来编写、管理任务，可以参考下面的示例，直接通过python代码来编写、管理任务

{% content-ref url="../delta-task-development/manage-task-with-delta-node-api.md" %}
[manage-task-with-delta-node-api.md](../delta-task-development/manage-task-with-delta-node-api.md)
{% endcontent-ref %}



在本文档的计算任务开发章节，也有更多的关于Delta Task框架的解释说明。
