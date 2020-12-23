##  并发编程系列（10）StampedLock的原理及使用

### 1. 引入原因
- 1.8之前，锁已经有那么多了，为什么还要有StampedLock？
>  一般引用,都是读多写少，RenentranReadwriteLock因为读写互斥，故读时阻塞写，因而性能上不去，可能会使写线程饥饿。

### 2. StampedLock的特点
- 所有获取锁的方法，都返回一个邮戳（Stamp），Stamp为0表示获取失败，其余表示成功。
- 所有释放锁的方法，都需要一个邮戳（Stamp），这个Stamp必须和成功获取锁时得到的Stamp一致。
- 支持锁升级跟降级
- 可以乐观读也可以悲观读
- 使用有限次自旋，增加锁的获得几率，避免上下文切换带来的开销
- 乐观读不阻塞写操作，悲观读会阻塞写操作。

### 3. StampedLock的优点
- 相比于ReentryReadWriteLock，吞吐量大幅提升。
### 4. StampedLock的缺点
- api相对复杂，容易用错
- 内部实现相比于ReentryReadWriteLock复杂得多。
### 5. StampedLock的原理
- 每次获取锁的时候，都会返回一个邮戳（Stamp），相当于mysql的version字段
- 释放锁的时候，根据之前的邮戳，去进行锁释放
### 6. 使用StampedLock注意点
- 如果使用乐观读，一定要判断返回的邮戳是否是一开始获取到的，如果不是，要去获取悲观读锁，再次去读取