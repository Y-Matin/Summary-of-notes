## kubernet系列（2）组件介绍
<!-- TOC -->

- [kubernet系列（2）组件介绍](#kubernet系列2组件介绍)
  - [Pod](#pod)
  - [控制器类型](#控制器类型)
      - [ReplicationController(RC)](#replicationcontrollerrc)
    - [ReplicaSet(RS)](#replicasetrs)
    - [Deployment](#deployment)
  - [StatefulSet<状态集>](#statefulset状态集)
    - [引用场景：](#引用场景)
  - [DaemonSet<守护进程集>](#daemonset守护进程集)
    - [典型用法：](#典型用法)
  - [Job，Con Job](#jobcon-job)
  - [Pod的生命周期](#pod的生命周期)
    - [Initc](#initc)
    - [探针](#探针)

<!-- /TOC -->

### Pod
- Pod是K8s中最小的管理单元
- Pod中可以包含多个cantainer（容器）
- 多个相关联的Pod


### 控制器类型
##### ReplicationController(RC)
> 用来确保容器引用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的Pod来替代；而如果异常多出来的容器也会自动回收。  
> 在新版本的Kubernetes中建议使用ReplicaSet来取代ReplicationController
#### ReplicaSet(RS)
> ReplicaSet跟ReplicationController没有本质的不同，只是名字不一样，并且**ReplicaSet支持集合式selector**。
#### Deployment
> 虽然ReplicaSet可以独立使用，但一般还是建议使用Deployment来自动管理ReplicaSet，这样就无需担心跟其他几只的不兼容问题，比如：ReplicaSet不支持rolling-update，但Deployment支持。
### StatefulSet<状态集> 
- docker 针对的是无状态服务，无需实时存储数据，即 容器A停止后，其他容器经过一段时间运行后，在把容器A启动，这时容器A依然可以正常工作，典型的有appache tomcat；

-StatefulSet是为了解决有状态服务的问题。 有状态服务：mysql....
#### 引用场景：
- 稳定的持久化存储，及Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
- 稳定的网络标志，及Pod重新调度后，其PodName和HostName不变，基于Headless Service（即没有Cluster IP 的Service）来实现
- 有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要以及定义依次进行，（即从0到N-1，在下一个Pod运行之前的Pod必须是Running和Ready状态），基于init containers来实现
- 有序收缩，有序删除（即从N-1到0）
### DaemonSet<守护进程集>
> DaemonSet 确保全部（或者一些）Node上运行一个Pod的副本。当有Node加入集群时，也会为他们新增一个Pod。当有Node从集群移除是，这些Pod也会被回收。删除DaemonSet将会删除它创建的所有Pod
#### 典型用法：
- 运行集群存储daemon，例如在每个Node上运行gluserd、ceph。
- 在每个Node上运行日志收集daemon，例如flentd、logstash。
- 在每个Node上运行监控daemon,例如Prometheus Node Exporter。

### Job，Con Job
- Job负责批处理任务，即仅执行一次的任务，他保证批处理任务的一个或多个Pod成功结束
- Cron Job 管理基于时间的Job，比如：定时任务。 

### Pod的生命周期
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/pod生命周期.png)
####  Initc
- Pod能够拥有多个容器，应用运行在容器里面，但是他也可能有一个或多个先于容器启动的Init容器
Init容器与普通容器非常像，除了一下两点：
  1. Init容器总是运行到成功为止
  2. 每个Init容器都必须在下一个Init容器之前成功完成
- 如果Pod的Init容器失败，Kubernetes会不断的重启该Pod，知道Init容器成功为止。然而 如果***Pod对应的restartPolicy为Never***，他不会重新启动

#### 探针
> 探针是有kubelet对容器执行的定期诊断，要执行诊断，kubelet调用有容器实现的Handler。有三种类型的处理程序：
  1. ExecAction：在容器内执行指定命令。如果命令退出时，返回码为0则认为诊断成功。
  2. ECPSocketAction：对指定端口的IP地址进行TCP检查，如果端口打开，则诊断被认为是成功的。
  3. HTTPGetAction：对指定的端口和路径上的容器IP地址执行HTTP GET请求，如果响应的转台吗大于等于200，且小于400，则诊断被认为是成功的。
> 每次诊断都将获得以下三种结果之一：
  1. 成功：容器通过率诊断。
  2. 失败：容器未通过诊断。
  3. 未知：诊断失败，因此不会采取任何行动。

