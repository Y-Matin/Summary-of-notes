<!-- ##### Kubernet系列（X）细节，知识点 -->

#####  kubectl api-resource 
-  查看 命令和 版本的关系
```terminal
martin@martin:~$ mkubectl api-resources 
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
.....
```
#### 端口转发
- mkubectl port-forward -n kube-system service/kubernetes-dashboard --address=0.0.0.0 10443:443
Forwarding from 0.0.0.0:10443 -> 8443
####  检查
- 启动检查 ： 周期检查服务是否可用
  - exec 退出码为0
  - http.get（）请求
  - tcp port ping访问
- 就绪检查 ： 检查一次，服务是否能对外提供服务
- 启动检查 ： 热点数据

#### 解析 pod 配置文件示例
- 使用命令 获取某个pod 的 配置详情：
> mkubectl get  pod dashboard-metrics-scraper-6b6f796c8d-jxv6z  -n kube-system  -o yaml



### dry-run 
- 使用dry-run，检查 或者得到 yaml的基本值
```
kubectl create serviceaccount ingress-manager-sa --dry-run=client -o yaml > manifests/ingress-manager-sa.yml

```

### api-resources
- 查看 资源类型和版本的关系
```
martin@yds-server:~$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node

```
### explain
- 查看某个资源下的 所有的属性字段
```
 kubectl explain pod
 kubectl explain pod.spec

```




-----
```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:  
    cni.projectcalico.org/containerID: 76a2bc2c3b6db8797545629517e730375b41512920131b5d64655d356a9460d8
    cni.projectcalico.org/podIP: 10.1.100.155/32
    cni.projectcalico.org/podIPs: 10.1.100.155/32
  creationTimestamp: "2022-07-20T15:38:13Z"
  generateName: dashboard-metrics-scraper-6b6f796c8d-
  labels:
    k8s-app: dashboard-metrics-scraper
    pod-template-hash: 6b6f796c8d
  name: dashboard-metrics-scraper-6b6f796c8d-jxv6z
  namespace: kube-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: dashboard-metrics-scraper-6b6f796c8d
    uid: f5a3f0e1-5139-4034-8252-3d2e563d6a42
  resourceVersion: "11862"
  uid: 6c3af1c4-1b0d-491e-8cf5-5393e50fd58f
spec:
  containers:
  - image: kubernetesui/metrics-scraper:v1.0.6
    imagePullPolicy: IfNotPresent                  #重新拉取镜像
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 8000
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 30
    name: dashboard-metrics-scraper
    ports:
    - containerPort: 8000
      protocol: TCP
    resources: {}
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsGroup: 2001
      runAsUser: 1001
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /tmp
      name: tmp-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-6nmmr
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: martin
  nodeSelector:
    kubernetes.io/os: linux
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always                           # 重启策略
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: kubernetes-dashboard
  serviceAccountName: kubernetes-dashboard
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: tmp-volume
  - name: kube-api-access-6nmmr
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T15:38:13Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-11-07T13:49:08Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-11-07T13:49:08Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-07-20T15:38:13Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://85802318023dff762f6da1f1cb92015a612a26208a49bc92fcf8c302bc5845dd
    image: docker.io/kubernetesui/metrics-scraper:v1.0.6
    imageID: docker.io/kubernetesui/metrics-scraper@sha256:1f977343873ed0e2efd4916a6b2f3075f310ff6fe42ee098f54fc58aa7a28ab7
    lastState:
      terminated:
        containerID: containerd://12d34664b6beaef9e64143b83867b9771a2eb482f7dac5e6d7cca4ca6fcc0737
        exitCode: 255
        finishedAt: "2022-11-07T13:48:34Z"
        reason: Unknown
        startedAt: "2022-11-07T13:14:30Z"
    name: dashboard-metrics-scraper
    ready: true
    restartCount: 5
    started: true
    state:
      running:
        startedAt: "2022-11-07T13:49:07Z"
  hostIP: 192.168.0.2
  phase: Running
  podIP: 10.1.100.155
  podIPs:
  - ip: 10.1.100.155
  qosClass: BestEffort
  startTime: "2022-07-20T15:38:13Z"

```

- restartPolicy ： 重启策略
  > Awallys: 除了正常退出外，都会重启容器  
  Never：： 任何情况下，都不会重启容器  
  OnFailure: 程序异常退出，健康检查失败时重启
- imagePullPolicy: 镜像拉取策略
  > Always:  每次都拉取最新镜像   
  IfNotPresent :  不存在才会拉取镜像  
  Never ：只使用本地镜像
