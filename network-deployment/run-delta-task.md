# 执行计算任务

在Delta隐私计算网络搭建完成后，我们就可以开始编写并执行一个Delta Task计算任务了。

Delta Task的开发可以在任意支持Python语言的IDE中进行。为了方便使用，Deltaboard中已经集成了JupyterHub，也就是支持多用户同时使用的JupyterLab，是在数据分析领域非常广泛使用的IDE，可以直接在线编辑代码，并实时显示执行结果。

Deltaboard的JupyterLab中放置了一个已经写好的Delta Task，启动Deltaboard的界面，进入JupyterLab的Tab，打开HelloWorld.ipynb文件，可以看到这个示例Delta Task的代码：

示例代码是一个通过神经网络进行手写数字识别的模型训练。在运行代码之前，需要确认Delta Node中已经放置了示例代码需要的公开数据集[MNIST](http://yann.lecun.com/exdb/mnist)，具体的放置方法可以参考文档的对应章节：

{% page-ref page="prepare-data.md" %}

准备好数据后，在Delta Task代码中将数据集的名称修改为节点上放置的文件名后，可以点击JupyterLab中的运行按钮，开始Delta Task的执行，并查看执行结果。开发者也可以尝试自行修改Delta Task的代码，并运行查看结果：

从执行Log可以看出，这个任务以横向联邦学习的方式，被分发给了网络中的全部节点并完成了多轮的训练。每轮训练中，各个节点完成本地计算后，上报了经过安全聚合处理的梯度向量，而我们的发起方Delta Node收到全部节点的统计结果后，求和得到了最终的训练模型。至此我们已经完成了第一个Delta Task的编写和全网运行。

在开始真正编写自己的计算任务以前，开发者还需要对Delta Task框架有更深入的了解。可以从详细了解这个示例任务开始：

{% page-ref page="../delta-task-development/delta-task-example.md" %}

在本文档的计算任务开发章节，也有更多的关于Delta Task框架的解释说明。

