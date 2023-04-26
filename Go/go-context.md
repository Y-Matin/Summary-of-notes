#### go中 Context 的设计解析
> context

详细解析文章url: https://mp.weixin.qq.com/s/OCpVRwtiphFRZgu9zdae5g



```
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
    Err() error
	Value(key any) any
}

type canceler interface {
	cancel(removeFromParent bool, err error)
	Done() <-chan struct{}
}

```


```
type emptyCtx int

type cancelCtx struct {
	Context

	mu       sync.Mutex            // protects following fields
	done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}


type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}

type valueCtx struct {
	Context
	key, val any
}


``