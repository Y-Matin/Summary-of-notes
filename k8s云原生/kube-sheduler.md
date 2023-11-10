
### kube-sheduler
- 调度分为两个阶段，predicatea和priority
- predicate：过滤不符合条件的节点
- priority： 优先级排序，选择优先级最高的节点

#### prpedicate  条件过滤



#### priority  优先级排序

#### 资源需求
- cpu
  - requests
    - kubenetes调度pod时，会判断当前节点正在运行的cpu request的总和，再加上当前调度pod的cpu request，计算其是否超过节点的cpu可分配资源
    - limits
      - 配置cgroup以限制资源上限
- 内存
  - requests
    - 判断节点剩余内存是否满足pod的内存请求量，以确定是否可以将pod调度到该节点
  - limits
    - 配置cgroup以限制资源上限

#### initContainer的资源需求
- 当kube-sheduler调度带有多个init容器的pod时，只计算cpu.request最大的init容器，而不是计算所有init容器之和
- 因为init容器时按顺序执行的，并且执行完成立即退出，所以申请最多资源的init容器即可满足所有init容器的资源需求
- kube-sheduler在计算该节点被占用的资源时，init容器的资源依然会被纳入计算，因为init容器在特定情况下可能会被再次执行，比如由于更改镜像而引起的Sandbox重建


#### 把Pod调度到Node上的方式
- 通过 nodeSelector、nodeAffinity、podAffinity以及Taints和tolerations等来将pod调度到需要的Node上
- 或者通过nodeName参数，将Pod调度到指定的node节点上
- 通过nodeSelector来指定node




```yaml
```


### 亲和性
- 

- 

