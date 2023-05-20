## 并发编程系列（8）ReentrantLock源码-公平锁&非公平锁
<!-- TOC -->

- [并发编程系列（8）ReentrantLock源码-公平锁&非公平锁](#%e5%b9%b6%e5%8f%91%e7%bc%96%e7%a8%8b%e7%b3%bb%e5%88%978reentrantlock%e6%ba%90%e7%a0%81-%e5%85%ac%e5%b9%b3%e9%94%81%e9%9d%9e%e5%85%ac%e5%b9%b3%e9%94%81)
  - [1. ReentrantLock 部分源码](#1-reentrantlock-%e9%83%a8%e5%88%86%e6%ba%90%e7%a0%81)
  - [2. 抽象Sync内部类结构如下：](#2-%e6%8a%bd%e8%b1%a1sync%e5%86%85%e9%83%a8%e7%b1%bb%e7%bb%93%e6%9e%84%e5%a6%82%e4%b8%8b)
  - [3. 非公平锁源码](#3-%e9%9d%9e%e5%85%ac%e5%b9%b3%e9%94%81%e6%ba%90%e7%a0%81)
  - [4. 公平锁源码](#4-%e5%85%ac%e5%b9%b3%e9%94%81%e6%ba%90%e7%a0%81)
  - [5. 公平锁与非公平锁在获取锁的源码对比：](#5-%e5%85%ac%e5%b9%b3%e9%94%81%e4%b8%8e%e9%9d%9e%e5%85%ac%e5%b9%b3%e9%94%81%e5%9c%a8%e8%8e%b7%e5%8f%96%e9%94%81%e7%9a%84%e6%ba%90%e7%a0%81%e5%af%b9%e6%af%94)

<!-- /TOC -->

-  ReentrantLock内部关系

![](images/ReentrantLock_class.png)

- 1. ReentrantLock实现了Lock接口
- 2. ReentrantLock有三个内部类(Sync,FairSync,NonFairSync)
- 3. 三个内部类的关系：FairSync,NonFairSync都是Sync的子类
- 4. Sync继承了AQS抽象类-AbstractQueuedSynchronizer

***

### 1. ReentrantLock 部分源码
- 默认创建的是锁类型是：非公平锁-NonFairSync
```java
    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```
***
### 2. 抽象Sync内部类结构如下：

![](images/Sync.png)

- 由上面代码可以看出
- 1. Sync 自己新建了一个lock抽象方法(由于ReentrantLock实现了Lock接口，需要实现lock()方法，所以调用Sync提供的lock(),具体执行的是它的子类的lock()逻辑)
- 2. Sync 新建了一个为非公平锁实现的获取锁的方法。-nonFairTryAcquire()
- 3. 同时也实现了一些 final的方法，体现了模板设计模式的思想，只允许扩展指定的步骤。
  ***

### 3. 非公平锁源码
```java
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```
- **总结**
- 1. 非公平锁重写了AQS的tryAcquire()方法，使用了父类Sync提供的获取锁逻辑
- 2. 获取锁逻辑是 先使用CAS操作，尝试占有锁，如果成功，则将state从0置为1，将当前线程记录下来；如果失败，再调用AQS的获取逻辑。
- 3. AQS的获取锁逻辑一部分被重写了，在非公平锁中，也是先判断锁是否空闲，如果空闲再次尝试获取锁，不是空闲的，及看当前线程是否是锁的持有者，如果是，state加1，如果不是锁的持有者，则获取锁失败。
***

### 4. 公平锁源码
```java
    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```
- **总结**
- 1. 公平锁获取锁的逻辑是直接调用AQS的获取锁逻辑。
- 2. AQS的获取锁逻辑一部分被重写了，在公平锁中，是先判断锁是否空闲，如果空闲或查询等待队列中是否有其他线程，如果没有其他线程则获取锁；如果不是空闲的，及看当前线程是否是锁的持有者，如果是，state加1，如果不是锁的持有者，则获取锁失败。
  

### 5. 公平锁与非公平锁在获取锁的源码对比：
![](images/compare_fair_nonfair.png)




