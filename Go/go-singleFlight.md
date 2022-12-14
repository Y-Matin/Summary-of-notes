### singleFlight
#### 功能
> 能让同一类请求，只执行一次处理函数，剩下的请求阻塞住，得到的结果后，这类请求共享这个返回值， 减少相同逻辑的重复执行。
#### 适用场景
1. 处理缓存击穿的场景：
2. 处理dashboard首页场景， 结果缓存一分钟
#### 源码解析
##### call :描述 一个请求的结果
```golang
// call is an in-flight or completed singleflight.Do call
type call struct {
	wg sync.WaitGroup  // 互斥锁 比 sync.Mutex 好处是：在请求出完后，等待的其他请求可以同时继续业务， mutex只能单个加锁，释放锁，效率低;用sync.RWMutex，似乎能达到WG的效果，但可能后一个请求先拿到读锁，导致原始请求无法拿到写锁，结果出错。

	val interface{}
	err error

	dups  int
	chans []chan<- Result
}

```
#### Group
``` golang
// Group represents a class of work and forms a namespace in
// which units of work can be executed with duplicate suppression.
type Group struct {
	mu sync.Mutex       // protects m
	m  map[string]*call // lazily initialized   用于记录  正在运行的key对应的处理请求
}
```
######  Group.do() 
> Do executes and returns the results of the given function, making sure that only one execution is in-flight for a given key at a time. If a duplicate comes in, the duplicate caller waits for the original to complete and receives the same results. The return value shared indicates whether v was given to multiple callers.  
 确保 一个时间点，最多只会执行一次，重复调用者会等待原始调用后的结果。

2. Group.DoChan()
> DoChan is like Do but returns a channel that will receive the results when they are ready.   
 和Do()类似， 但是会直接返回一个chan， 当原始调用者完成调用后，会从chan中读取值。 可以添加超时逻辑

3. Group.Forget() 
> 删除 对应key的记录