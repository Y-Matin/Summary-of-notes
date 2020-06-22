<!-- ## Kubernet系列（4）资源清单 -->

- **k8s中所有的内容都抽象为资源，资源实例化之后，叫做对象**
### 集群资源分类
1. 名称空间级别<namespace>
    > 比如：k8s的自身的Pod都在kube-system这个名称空间下。一般的pod都默认创建在default这个空间下。
2. 集群级别
   > 比如：role， 全集群可见。
3. 元数据型
   > 通过指标进行操作，比如：hpa，根据cup使用率来进行扩容

### 名称空间级别的资源
1. 工作负载型资源（workLoad）：Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、CronJob。
2. 服务发现及负载均衡型资源（serviceDiscover LoadBalance）：Service、Ingress、....
3. 配置与存储型资源：Volume（存储卷）、CSI（容器存储接口，可以扩展各种第三方存储卷）
4. 特殊类型的存储卷：ConfigMap（当配置中心类使用的资源类型）、Secret（保存敏感数据）、DownwardAPI（把外部环境中的信息输出给容器）
#### 集群级别的资源
- Namespace、Node、Role、RoleBinding、ClusterRoleBinding
#### 元数据型资源
- HPA、PodTemplate、LimitRange

### 资源清单
- **在k8s中，一般使用yaml格式的文件来创建符合我们预期期望的pod，这样的yaml文件我们一般称为资源清单**
#### 重要的属性
|  参数名   | 字段类型  | 说明 |
|  ----  | ----  |----|
| version  | String | 这里指的是K8S 的API版本，目前基本上都是v1，可以用kubectl api-version查询
| kind  | String | 这里指的是yaml文件定义的资源类型和角色，比如：Pod
| metadata| Object | 元数据对象，固定值就写metadata
| metadata.name | String | 元数据对象的名字，这里有我们编写，比如命名Pod的名字
| metadata.namespace | String | 元数据对象的命名空间，这里由我们自身定义
| Spec | Object | 详细定义对象，固定值就写Spec
| spec.containers[] | List | 这里是Spec对象的容器列表定义，是个列表
| spec.containers[].name | String | 这里定义容器的名字
| spec.containers[].image | String | 这里定义要用到的镜像名称
| spec.containers[].imagePullPolicy | String | 定义镜像拉取策略，有Always、Never、ifNotPresent三个值可选。（1）Always：意思是每次都尝试重新拉取镜像；（2）Never：表示仅适用本地镜像；（3）ifNotPresent：如果本地有则使用本地镜像，如果本地没有就拉取在线镜像。默认值为Always
| spec.containers[].command[] | List | 指定容器启动命令，因为数组可以指定多个，不指定则使用镜像打包时使用的启动命令
| spec.containers[].args[] | List | 指定容器启动命令参数，因为数组可以指定多个。
| spec.containers[].workingDir | String | 指定容器的工作目录
| ...| ... |...
***
