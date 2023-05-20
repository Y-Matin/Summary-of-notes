DaltaFIFO

- Store的类型
  - cache：实现Store，利用threadSafeMap存放数据
  - UndeltaStore：实现Store，利用Cache存放数据，数据变更时通过PushFunc发送当前完整状态
  - FIFO： 实现Queue（包含Store），利用自己内部的items数据结构存放数据
  -  DeltaFIFO： 
  -  Heap：实现Store，利用data数据结构，存放数据，实现堆数据结构，用于优先队列
  -  ExpirationCache：实现Store，利用threadSafeMap存放数据
  

- indexer ： 是 Store的消费者
  - 

- ShareInformer的作用 ：可以添加事件处理方法
  - 缓存我们关注的资源对象的最新状态数据
    - eg:创建Indexer/Clientset（通过listerwatcher）/deltaFIFO/Conotroller（包含Reflector的创建） 
  - 根据资源对象的变化事件来通知我们注册的事件处理方法 
    - eg:创建ShardProcessor/注册事件处理方法 

- WorkQueue 
  - 通用队列
  - 延迟队列
  - 限速队列 默认使用


annotation：
  ingress/http: true




控制器有两个主要组件：Informer/SharedInformer 和 Workqueue。 Informer/SharedInformer监视Kubernetes对象当前状态的变化，并将事件发送到Workqueue，然后由Workers pop 事件进行处理。

### infromer
Informer 是 client-go 中的核心工具包，已经被 kubernetes 中众多组件所使用。所谓 Informer，其实就是一个带有本地缓存和索引机制的、可以注册 EventHandler 的 client
本地缓存被称为 Store，索引被称为 Index。使用 informer 的目的是为了减轻 apiserver 数据交互的压力而抽象出来的一个 cache 层, 客户端对 apiserver 数据的 "读取" 和 "监听" 操作都通过本地 informer 进行。
Informer 的主要功能：

    同步数据到本地缓存

    根据对应的事件类型，触发事先注册好的 ResourceEventHandle

```
Informer 在初始化时，Reflector 会先 List API 获得所有的 Pod
Reflect 拿到全部 Pod 后，会将全部 Pod 放到 Store 中
如果有人调用 Lister 的 List/Get 方法获取 Pod， 那么 Lister 会直接从 Store 中拿数据
Informer 初始化完成之后，Reflector 开始 Watch Pod，监听 Pod 相关 的所有事件;如果此时 pod_1 被删除，那么 Reflector 会监听到这个事件
Reflector 将 pod_1 被删除 的这个事件发送到 DeltaFIFO
DeltaFIFO 首先会将这个事件存储在自己的数据结构中(实际上是一个 queue)，然后会直接操作 Store 中的数据，删除 Store 中的 pod_1
DeltaFIFO 再 Pop 这个事件到 Controller 中
Controller 收到这个事件，会触发 Processor 的回调函数
LocalStore 会周期性地把所有的 Pod 信息重新放到 DeltaFIFO 中
```

#### 1.Reflector
Reflector 和 APIServer 建立长连接，并使用 ListAndWatch 方法获取并监听某一个资源的变化。List 方法将会获取某个资源的所有实例(如ReplicaSet、Deployment等)，Watch 方法则监听资源对象的创建、更新以及删除事件，获取到的事件称之为一个增量(Delta)，该增量会被放进一个称之为 Delta FIFO Queue，即增量先进先出队列中。
Reflector 包负责与 apiserver 建立连接，Reflector 使用 ListAndWatch 的方法，会先从 apiserver 中 list 该资源的所有实例，list 会拿到该对象最新的 resourceVersion，然后使用 watch 方法监听该 resourceVersion 之后的所有变化，若中途出现异常，reflector 则会从断开的 resourceVersion 处重现尝试监听所有变化，一旦该对象的实例有创建、删除、更新动作，Reflector 都会收到"事件通知"，这时，该事件及它对应的 API 对象这个组合，被称为增量（Delta），它会被放进 DeltaFIFO 中。


#### deltaFIFO
1. Informer 会不断地从这个 DeltaFIFO 中读取增量，每拿出一个对象，Informer 就会判断这个增量的事件类型，然后创建或更新本地的缓存，也就是 store。
2. 如果事件类型是 Added（添加对象），那么 Informer 会通过 Indexer 的库把这个增量里的 API 对象保存到本地的缓存中，并为它创建索引，若为删除操作，则在本地缓存中删除该对象。
3. DeltaFIFO 再 pop 这个事件到 controller 中，controller 会调用事先注册的 ResourceEventHandler 回调函数进行处理。


#### workqueue
Informer 根据事件类型来触发事先注册好的 Event Handler。在回调函数中通常只会做一些简单的过滤处理，然后将该事件的Key(注意不是事件本身，只是事件的key，key的格式如<resource_namespace>/<resource_name>)添加到 Work Queue 这个工作队列中。

在 ResourceEventHandler 回调函数中，其实只是做了一些很简单的过滤，然后将关心变更的 Object 放到 workqueue 里面。

Controller 从 workqueue 里面取出 Object，启动一个 worker 来执行自己的业务逻辑，业务逻辑通常是计算目前集群的状态和用户希望达到的状态有多大的区别，然后孜孜不倦地让 apiserver 将状态演化到用户希望达到的状态，比如为 deployment 创建新的 pods，或者是扩容/缩容 deployment。

在worker中就可以使用 lister 来获取 resource，而不用频繁的访问 apiserver，因为 apiserver 中 resource 的变更都会反映到本地的 cache 中。


--------------


####  reflector
- list: /api/v1/namespaces/{namespace}/pods

- watch: /api/v1/namespaces/default/pods?watch=true
- List的实现容易理解，那么Watch是如何实现的呢？Watch是如何通过HTTP 长链接接收apiserver发来的资源变更事件呢？
> HTTP 分块传输编码允许服务器为动态生成的内容维持 HTTP 持久链接。通常，持久链接需要服务器在开始发送消息体前发送Content-Length消息头字段，但是对于动态生成的内容来说，在内容创建完之前是不可知的。使用分块传输编码，数据分解成一系列数据块，并以一个或多个块发送，这样服务器可以发送数据而不需要预先知道发送内容的总大小。

秘诀就是Chunked transfer encoding(分块传输编码)，它首次出现在HTTP/1.1。正如维基百科所说：

- ResourceVersion 和Bookmarks
- ResourceVersion
   - 保证客户端数据一致性和顺序性
   - 并发控制
- Bookmarks
  - 减少API Server负载
  - 更新客户端保存的最近一次ResourceVersion  





