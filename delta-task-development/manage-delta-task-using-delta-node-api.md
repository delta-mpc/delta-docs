# Manage Delta Tasks using the Delta Node API

Delta Tasks could be developed in an IDE on the developer's local device, and submitted to the Delta Node using Delta Node API. The computation result, the execution status could also be fetched directly in the code, which makes it possible to embed the Delta PPC Tasks into a larger computation workflow and execute them automatically. In this document, we will show a complete example to write a Delta Task locally and execute it using the Delta Node API.

The task we're about to write is a federated analytics task that computes the average wage of all the employees distributed in 3 different enterprises. Each enterprise has a Delta Node deployed, and each Delta Node has access to a `wages.csv` file containing all the wages of the employees of the corresponding enterprise. Obviously `wages.csv` is sensitive to all the enterprises and should be always kept private for each of them. That's where federated analytics is required to compute the overall average wage without revealing the details of the wages data of each enterprise to others.

The Python library `delta-task` implements the relevant methods to call Delta Node APIs remotely. We must install the library in the local Python environment before running the task:

```
$ pip install delta-task
```
