## 并发编程系列（6）Lock接口

### Lock接口的使用
> 在同步代码块上下，手动添加加锁，解锁操作

### lock与synchronized区别
- Lock
 > lock 需要手动控制加锁，解锁。  
 > 用的是乐观锁方式
- synchronized
 >  托管给了JVM执行
 >  用的是悲观锁