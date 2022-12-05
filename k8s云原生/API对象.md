#### API对象

> API 对象是 Kubernetes集群中的管理操作单元
Kubernetes集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的API对象，支持对该功能的管理操作

##### 四大类属性
- TypeMeta：通过引入GKV 模型定义了一个对象的类型  
  - Group: kube 定义了非常多的对象， 将对象依据其功能范围归入不同分组， 比如：core，apps等，是的对这些对象的可维护性和可理解性更高
  - Kind：定义一个对象的基本类型，比如：Node，Pod，Deployment等
  - Version： 为了版本迭代，api兼容，引入的字段，内部将低版本转为高版本api调用  

- MetaData ： 数据描述；其中两个最重要的属性：Namespace和Name，分别定义了对象的Namespace归属和名字，这两个属性可以唯一标识一个对象实例
  - Label ： 给对象打上标签，一个对象可以有人已对标签，其表现形式是key/value,Label定义了对象的可识别属性，k8s api 支持以Label作为过滤条件查询对象 
  ``` yaml
     kubectl label ns default addUser=yds
     kubectl get ns -l addUser=yds
  ```
  - Annotation ： 和Label一样使用key/value类定义，但Annotation 是作为属性扩展，更多的面相系统管理员和开发人员，  
  - Finalizer: 本质上是一个资源锁，k8s在接收某对象的删除请求时，会检查finalizer是否为空，如果不为空，就只对其做逻辑删除，即只会更新对象中的metadata.deletionTimestamp字段。
  - ResourceVersion： 可以看做一种乐观锁，每个对象在任意时刻都有其ResourceVersion，极大提高系统的整体效率。当要对对象发生更改时，如果携带的version 和当前对象的version不一致，则认为操作是过时的，会被拒绝掉。和mysql中的乐观锁version基本一致

- Spec

- Status


#### 细节
##### Label
- Label是识别k8s对象的标签，以key/value的方式附加到对象上
- key 最大不能超过63字节，value可以为空，value最大值不能操作253字节的字符串
- 不提供唯一性， Pods可以使用相同的标签来标识
- 标签定义好后，其他对象可以使用Label Selector 来选择一组相同Label的对象
- Label Selector支持一下几种方式：
  - 等式，如app=nginx 和 env!=production
  - 集合，如 env in (production,qa)
  - 多个label（and 关系） 如 app=nginx,env=test
  