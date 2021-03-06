### 并发编程系列（7）AQS源码解析

AbstractQueuedSynchronizer --为实现依赖于先进先出(FIFO)的阻塞锁

> 为等待队列的阻塞锁和相关同步器(信号量、事件，等等)提供一个框架。
此类的设计目标是成为依靠单个原子int值来表示状态的大多数同步器的一个有用基础。子类必须定义更改此状态的受保护方法，并定义哪种状态对于此对象意味着被获取或被释放。假定这些条件之后，此类中的其他方法就可以实现所有排队和阻塞机制。子类可以维护其他状态字段，但只是为了获得同步而只追踪使用getState()、 setState(int)和

> compareAndSetState(int, int) 方法来操作以原子方式更新的int值。

> 应该将子类定义为非公共内部帮助器类，可用它们来实现其封闭类的同步属性。  

> 类AbstractQueuedSynchronizer没有实现任何同步接口。而是定义了诸如acquireInterruptib1y(int)之类的一些方法，在适当的时候可以通过具体的锁和相关同步器来调用它们，以实现其公共方法。

> 此类支持默认的独占模式和共享模式之一，或者二者都支持。

> 处于独占模式下时，其他线程试图获取该锁将无法取得成功。

> 在共享模式下，多个线程获取某个锁可能(但不是一定)会获得成功。此类并不“了解”这些不同除了机械地意识到当在共享模式下成功获取某一锁时， 下一个等待线程(如果存在)也必须确定自己是否可以成功获取该锁。处于不同模式下的等待线程可以共享相同的FIFO队列。

> 通常，实现子类只支持其中一种模式，但两种模式都可以在(例如) ReadWriteLock

> 中发挥作用。只支持独占模式或者只支持共享模式的子类不必定义支持未使用模式的方法。

> 此类通过支持独占模式的子类定义了一个嵌套的AbstractQueuedSynchronizer . ConditionObject

>继承 关系 


- AQS中提供的判断队列是否有其他等待线程 方法 
```java
 public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```