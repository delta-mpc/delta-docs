# Run the First Delta Task

We're finally ready to run the first Delta Task!

Delta Task could running in any environment that has Python runtime integrated. For fast testing and developing of the Delta Tasks. Deltaboard has [JupyterHub](https://jupyter.org/hub) embedded in, which is basically a multi-user version of the [JupyterLab](https://jupyter.org), which is a very popular IDE among the data scientists (not among the developers though, lol.). JupyterLab supports writing and running the code directly online in the browser, and supporting Markdown explanation blocks displayed alongside the codes.

We have put some example Delta Tasks inside the JupyterLab already, including both learning tasks and statistical tasks. Just go to the playground tab of the Deltaboard and go to the "examples" folder, you will find the examples in different languages:

![](<../.gitbook/assets/image (5).png>)

We will use `en-horizontal-learning-task.ipynb` as the example in this document. This is an example of a horizontal federated learning task that trains a neural network model to recognize handwriting digits on the public dataset [MNIST](http://yann.lecun.com/exdb/mnist/).&#x20;

Delta Node has a built-in command to prepare MNIST data, go to the following document for details if you have not:

{% content-ref url="prepare-data.md" %}
[prepare-data.md](prepare-data.md)
{% endcontent-ref %}

After the data is prepared, we could start to run the example codes. Just follow the instructions in the file, modify the codes as needed, and run the code blocks one-by-one. If everything goes right, the task will be submitted to the Delta Network. And the task ID will be given in the logs displayed below the code block.

![](<../.gitbook/assets/image (6).png>)

Now go to the "Task List" tab in the Deltaboard, we can find the task we just submitted. The running status of the task is also given:

![](<../.gitbook/assets/image (4).png>)

The task has already started running. Click on the item, we can see the detailed running logs:

![](<../.gitbook/assets/image (7).png>)

As the logs indicated, the task has been sent to multiple nodes in the network. Each node has performed several rounds of local training. The training result of each node has been masked using on-chain secure aggregation before sending back to the task initiator. The task initiator collected all the masked partial results, and summed them up to get the final result on all the data.

Now we have successfully finished our first privacy-preserving computation task. To know more about Delta Task and write your own real computation tasks, start here by knowing more about the example code:

{% content-ref url="../delta-task-development/horizontal-federated-learning-task.md" %}
[horizontal-federated-learning-task.md](../delta-task-development/horizontal-federated-learning-task.md)
{% endcontent-ref %}

An example task of the horizontal federated analytics is also given in this document:

{% content-ref url="../delta-task-development/horizontal-federated-analytics-task.md" %}
[horizontal-federated-analytics-task.md](../delta-task-development/horizontal-federated-analytics-task.md)
{% endcontent-ref %}

Delta Task could also be written in a local IDE, and submitted to the Delta Node using Delta Node API. In this scenario, the Deltaboard is no longer required. The details could be find in this document:

{% content-ref url="../delta-task-development/manage-delta-task-using-delta-node-api.md" %}
[manage-delta-task-using-delta-node-api.md](../delta-task-development/manage-delta-task-using-delta-node-api.md)
{% endcontent-ref %}
