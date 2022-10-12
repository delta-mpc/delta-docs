# 启动Deltaboard

## 通过Docker镜像启动Deltaboard

推荐使用Delta ZK的Docker镜像来进行部署

### 下载镜像

```
$ docker pull deltampc/delta-zk:0.8.0
```


### 启动节点服务

```
$ docker run -d --name=delta_zk -v ${PWD}:/app -p 3400:3400 deltampc/delta-zk:0.8.0
```

通过Docker的log命令查看Container的执行状态，确认节点已经正常启动：

```
$ docker logs -f delta_zk
```

至此Delta ZK启动完成。Delta ZK是用于生成零知识证明的服务，如果需要在任务中开启零知识证明，则在启动Delta Node前，
必须启动Delta ZK。启动Delta Node请参考：

{% content-ref url="start-delta-node.md" %}
[start-delta-node.md](start-delta-node.md)
{% endcontent-ref %}
