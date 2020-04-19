## 并发编程系列（12）ThreadLock使用
<!-- TOC -->

- [并发编程系列（12）ThreadLock使用](#%e5%b9%b6%e5%8f%91%e7%bc%96%e7%a8%8b%e7%b3%bb%e5%88%9712threadlock%e4%bd%bf%e7%94%a8)
  - [1. 简介](#1-%e7%ae%80%e4%bb%8b)
  - [2. 常见函数](#2-%e5%b8%b8%e8%a7%81%e5%87%bd%e6%95%b0)
  - [3. 使用](#3-%e4%bd%bf%e7%94%a8)

<!-- /TOC -->
### 1. 简介
- ThreadLock是线程变量，是一个以ThreadLock对象为键、任意对象为值的存储结构。
- 为每个线程单独存放一份变量副本，也就是说一个线程可以根据一个ThreadLock对象查询到绑定在则线程的一个值。
- 只要线程处于活动状态，并且ThreadLocal对象可访问，name每个线程都拥有对其本地线程副本的隐式引用变量。
- 一个线程消失后，他的所有副本线程局部实例受垃圾回收（除非有其他引用在使用这些副本）
  
### 2. 常见函数
- ThreadLocal.get: 获取ThreadLocal中当前线程共享变量的值。
- ThreadLocal.set: 设置ThreadLocal中当前线程共享变量的值。
- ThreadLocal.remove: 移除ThreadLocal中当前线程共享变量的值。
- ThreadLocal.initialValue: 获取第一次或者默认值。
  
### 3. 使用

```java
/**
 * @description: 使用ThreadLocal的示例demo
 * ThreadLock是线程变量，是一个以ThreadLock对象为键、任意对象为值的存储结构。
 * @author: Yeds
 * @create: 2020-04-16 14:44
 **/
public class ThreadLocalDemo {
    ThreadLocal<Integer> number = ThreadLocal.withInitial(() -> 0);

    public void set() {
        Integer num = number.get();
        num++;
        number.set(num);
        System.out.println( Thread.currentThread().getName()+"的number 的值：" + number.get());
    }

    public static void main(String[] args) {
        ThreadLocalDemo demo = new ThreadLocalDemo();
        for (int i = 1; i <= 2; i++) {
            int finalI = i;
            new Thread(() -> {
                while (true) {
                    demo.set();
                    try {
                        Thread.sleep(finalI * 1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}

```
- 运行结果：
> 运行结果如下： 可见 两个线程的 number值并不是共享的，而是各自线程独自拥有的线程变量
```java
Thread-1的number 的值：1
Thread-0的number 的值：1
Thread-0的number 的值：2
Thread-1的number 的值：2
Thread-0的number 的值：3
Thread-0的number 的值：4
Thread-1的number 的值：3
Thread-0的number 的值：5
```