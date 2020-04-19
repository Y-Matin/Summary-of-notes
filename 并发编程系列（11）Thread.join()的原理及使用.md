## 并发编程系列（11）Thread.join()的原理及使用

- Java中如何让多线程按照自己指定的顺序执行？
> 这个问题最简单的回答是通过Thread.join来实现，久而久之就让很多人误以为Thread.join是用来保证线程的顺序性的。

> thread.join的含义是当前线程需要等待previousThread线程终止之后才从thread.join返回。简单来说，就是线程没有执行完之前，会一直阻塞在join方法处。

- 什么时候会使用Thread.join
```java
public void joinDemo(){
   //....
   Thread t=new Thread(payService);
   t.start();
   //.... 
   //其他业务逻辑处理,不需要确定t线程是否执行完
   insertData();
   //后续的处理，需要依赖t线程的执行结果，可以在这里调用join方法等待t线程执行结束
   t.join();
}


```