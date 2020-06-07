<!-- ## Kubernet 系列   -->
### 简介
- Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。
- Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用（就像canary deployments）
### 架构图
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/K8s架构图.png)
### 重要概念
> 在这张系统架构图中，我们把服务分为运行在工作节点上的服务和组成集群级别控制板的服务。

> Kubernetes节点有运行应用容器必备的服务，而这些都是受Master的控制。

> 每次个节点上当然都要运行Docker。Docker来负责所有具体的映像下载和容器运行。


####  Kubernetes主要由以下几个核心组件组成：
- **etcd**：保存了整个集群的状态；键值对数据库，存储k8s集群所有重要信息（持久化）
- **apiserver**：提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- **controller manager**：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；维持副本期望数目。
- **scheduler**：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；选择合适的节点进行分配任务。
- **kubelet**：直接与容器引擎交付，实现容器的生命周期管理，同时也负责Volume（CVI）和网络（CNI）的管理；
- **Container runtime**：负责镜像管理以及Pod和容器的真正运行（CRI）；
- **kube-proxy**：负责写入规则至IPTABLES、IPVS，负责为Service提供cluster内部的服务发现和负载均衡；

**官方提供的**

#### 除了核心组件，还有一些推荐的Add-ons：
- **kube-dns**：负责为整个集群提供DNS服务
- **Ingress Controller**：为服务提供外网入口（官方只能实现四层代理，Ingress可以实现七层代理）
- **Heapster**：提供资源监控
- **Dashboard**：提供GUI，给k8s提供一个B/S结构访问体系
- **Federation**：提供一个可以块集群中心的多k8s统一管理的功能
- **Prometheus**：普罗米修斯 提供k8s集群的监控能力
- **Fluentd-elasticsearch**：提供集群日志采集、存储与查询
  **高可用集群副本数据最好是>=3 （奇数个）**

  ![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/k8s-master.png)
  ![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/k8s-node.png)