# 准备节点数据

Delta Node的Data Connector组件的开发工作量较大，目前仍在开发中。

Data Connector包含一个对数据进行标准化描述的协议，Delta Task中可以直接使用协议SDK来引用数据。Data Connector支持连接多个不同的数据源，包括文件、关系型数据库、HDFS等，并支持对数据源的数据进行自动校验和清洗。Delta Node接收到计算任务后，判断自身的数据是否满足计算条件并上报给任务发起方，任务发起方根据各个节点的数据持有情况来决定采用哪种隐私计算策略。

在Data Connector上线以前，我们可以采用简单的数据文件的形式向Delta Node中放置数据。

使用Delta Node的镜像启动Delta Node时，需要绑定一个持久化存储卷，如果按照上一节中的教程启动了Delta Node，则持久化存储卷就是用户创建的`delta_node`文件夹。

