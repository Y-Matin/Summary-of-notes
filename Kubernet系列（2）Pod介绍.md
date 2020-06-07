<!-- ## kubernet系列（2）Pod介绍 -->

### 


### 控制器类型
##### ReplicationController(RC)
> 用来确保容器引用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的Pod来替代；而如果异常多出来的容器也会自动回收。  
> 在新版本的Kubernetes中建议使用ReplicaSet来取代ReplicationController
#### ReplicaSet(RS)
> ReplicaSet跟ReplicationController没有本质的不同，只是名字不一样，并且**ReplicaSet支持集合式selector**。
#### Deployment
> 虽然ReplicaSet可以独立使用，但一般还是建议使用Deployment来自动管理ReplicaSet，这样就无需担心跟其他几只的不兼容问题，比如：ReplicaSet不支持rolling-update，但Deployment支持。
### StatefulSet 
- docker 针对的是无状态服务，无需实时存储数据，即 容器A停止后，其他容器经过一段时间运行后，在把容器A启动，这时容器A依然可以正常工作，典型的有appache tomcat；

-StatefulSet是为了解决有状态服务的问题。 有状态服务：mysql....
#### 引用场景：
- 稳定的持久化存储，及Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
- 稳定的网络标志，及Pod重新调度后，其PodName和HostName不变，基于Headless Service（即没有Cluster IP 的Service）来实现
- 有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要以及定义依次进行，（即从0到N-1，在下一个Pod运行之前的Pod必须是Running和Ready状态），基于init containers来实现
- 有序收缩，有序删除（即从N-1到0）


