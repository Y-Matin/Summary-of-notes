### 通过controller来实现对pod的管理

- k8s推荐使用controller来管理pod，这符合k8s管理pod的习惯，便于使用k8s相关功能，比如弹性扩缩容，pod故障自动拉起等。 
- 我们也以controller管理的pod为例，简单梳理下k8s创建及调度pod流程
##### 具体过程为
- 客户端请求apiserver创建replicasets，apiserver通过认证、鉴权、准入后，会把请求相关信息持久化至etcd
- Controller-manager 管理的replicaset controller 通过list-watch机制，watch到有replicasets创建请求，通过label selector发现集群中与这个replicasets 关联的pod当前状态与期望状态不一致，则会进行调协（reconcile）向apiserver发起创建pod请求
- Scheduler 通过list-watch机制来发现未绑定的pod，并通过预选及优选策略算法，来计算出pod最终可调度的node节点，并通过apiserver将数据更新至etcd
- Kubelet 通过list-watch发现有新的pod bound到本node上，则会发起创建pod相关流程
- 