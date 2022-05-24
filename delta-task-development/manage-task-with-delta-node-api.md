# 使用Delta Node API管理任务

通常，我们使用Deltaboard，在浏览器中进行任务的管理。
但是，部署Deltaboard比较复杂。为了方便用户，我们在delta包中提供了对Delta Node API的封装，
用户可以直接使用代码来管理任务。

下面，我们对这些API进行介绍。

## create_task - 创建任务

```python
delta.delta_node.DeltaNode.create_task(self, task)
```

用户可以通过该方法将任务提交到Delta Node上，创建任务。

参数：

* task: delta.core.task, 需要提交的任务，由用户编写的任务经过build生成

返回值：

* 任务ID，int类型

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
```

## trace - 跟踪任务日志

```python
delta.delta_node.DeltaNode.trace(self, task_id)
```

用户可以通过该方法，跟踪已经创建的任务的日志。该方法是阻塞方法，会持续打印任务的日志，直至任务正常结束或异常退出。

参数：

* task_id: 已经提交的任务ID。可以通过create_task方法得到

返回值：

* 任务状态，bool类型，True表示任务执行成功，False表示任务执行过程中出现异常

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
delta_node.trace(task_id)
```

命令行输出：

```
2022-05-24 03:43:06  start run task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a
2022-05-24 03:43:06  [Create Task] create task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a
...
...
...
2022-05-24 03:43:27  task 0xcea03535685face8c12034c585876062915cbf01810c711c3cbb7e424f56d24a finish
```

## wait - 等待任务结束

```python
delta.delta_node.DeltaNode.wait(self, task_id)
```

用户可以通过该方法，等待任务结束。该方法是阻塞方法，会阻塞直至任务正常结束或异常退出。

参数：

* task_id: 已经提交的任务ID。可以通过create_task方法得到

返回值：

* 任务状态，bool类型，True表示任务执行成功，False表示任务执行过程中出现异常

例子：

```python
# Example is a horizontal learning task (subclass of HorizontalLearning) or a horizontal analytics task (subclass of HorizontalAnalytics)
task = Example().build()
DELTA_NODE_API = "http://127.0.0.1:6700"
delta_node = DeltaNode(DELTA_NODE_API)
task_id = delta_node.create_task(task)
delta_node.wait(task_id)
```