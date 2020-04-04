## 并发编程系列（4）synchronized、volatile关键字
<!-- TOC -->

- [并发编程系列（4）synchronized、volatile关键字](#%e5%b9%b6%e5%8f%91%e7%bc%96%e7%a8%8b%e7%b3%bb%e5%88%974synchronizedvolatile%e5%85%b3%e9%94%ae%e5%ad%97)
  - [1. 内置锁](#1-%e5%86%85%e7%bd%ae%e9%94%81)
  - [2. 互斥锁](#2-%e4%ba%92%e6%96%a5%e9%94%81)
  - [3. synchronized的三种用法](#3-synchronized%e7%9a%84%e4%b8%89%e7%a7%8d%e7%94%a8%e6%b3%95)
  - [4. volatile关键字机器使用场景](#4-volatile%e5%85%b3%e9%94%ae%e5%ad%97%e6%9c%ba%e5%99%a8%e4%bd%bf%e7%94%a8%e5%9c%ba%e6%99%af)
  - [5. 单例与线程安全](#5-%e5%8d%95%e4%be%8b%e4%b8%8e%e7%ba%bf%e7%a8%8b%e5%ae%89%e5%85%a8)
    - [5.1. 饿汉式 -- 本身线程安全](#51-%e9%a5%bf%e6%b1%89%e5%bc%8f----%e6%9c%ac%e8%ba%ab%e7%ba%bf%e7%a8%8b%e5%ae%89%e5%85%a8)
    - [5.2. 懒汉式](#52-%e6%87%92%e6%b1%89%e5%bc%8f)

<!-- /TOC -->
### 1. 内置锁
> 每个java对象都可以用作一个实现同步的锁，这些锁成为内置锁。线程进入同步代码块或方法的时候回自动获取该锁，在执行完同步代码块或方法时会释放该锁。获得该内置锁的唯一途径就是进入这个锁保护的同步代码块或方法。
### 2. 互斥锁
> 内置锁是一个互斥锁，意味着只有一个线程可以获得该锁，当线程A尝试获取线程B持有的内置锁时，线程A必须等待或者阻塞，直到线程B释放这个锁，如果线程B不释放这个锁，线程A将永远的等待下去。
### 3. synchronized的三种用法

- 1.修饰静态方法：
    >锁住的是整个类。同一时刻，只有一个线程可以调用这个静态方法，与是否是同一个实例无关 ，进去同步代码前要获得当前类对象的锁。 
- 2.修饰实例方法：
    >锁住的是类的实例。同一时刻，只有一个线程可以调用这个实例的方法。进入同步代码前要获得当前实例的锁。
- 3.修饰代码块
    >这需要指定加锁的对象，对所给的指定对象加锁，进入同步代码前要获得指定对象的锁。锁住的是一个对象object "synchronized(object)"  
    - 如果该对象是static修饰的类成员变量，则锁住的是 类的静态变量。
    - 如果该对象是非static修饰的类成员变量，则锁住的是类实例的成员变量。

### 4. volatile关键字机器使用场景
-特点：
> 能且仅能修饰变量  
 只保证该变量的可见性，不保证原子性  
 禁止指令重排序

- 作用：
> A,B两个线程同时读取volatile关键字修饰的对象，A读取之后，修改了变量的值，修改后的值对于B线程来说是可见的。

- 使用场景：
 1. 作为线程开关
 2. 单例，修饰对象实例，禁止指令重排序


### 5. 单例与线程安全


#### 5.1. 饿汉式 -- 本身线程安全
- 在类加载时，就已经实例化，无论后面是否会用到。如果该类比较占内存，就会浪费掉较多资源。
-代码实例：
```java
/**
 * @description: 饿汉式单例 -- 本身线程安全
 * @author: Yeds
 * @create: 2020-04-02 18:41
 **/
public class HungerSingleton {

    // 如果使用 public 来修饰 singleton，从访问层面来看，是没有问题的，但及其的不安全，
    // 别人可以通过这种方式来赋值：HungerSingleton.singleton= null。 因此不采用public修饰
    private static HungerSingleton singleton = new HungerSingleton();

    public static HungerSingleton getinstance() {
        return singleton;
    }

    private HungerSingleton() {
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                System.out.println(HungerSingleton.getinstance());
            }).start();
        }
    }

}
```
#### 5.2. 懒汉式
- 在需要的时候实例化  
- 重要点：synchronized、双重检测锁机制、volatile
```java
/**
 * @description: 懒汉式单例 --最简单的写法是
 * 非线程安全的 ->synchronized修饰方法<性能不行> ->synchronized修饰new代码块<线程不安全> -> double check -> 添加 volatie 修饰符
 * @author: Yeds
 * @create: 2020-04-02 18:45
 **/
public class LazyHungerSingleton {

    // private ：只能通过方法获取，防止别人对singleton赋值
    // static ： 因为getInstance()方法是静态的，所以返回的数据只能是static
    // volatile : 作用是为了禁止指令重排序。对双重检测锁的补充
    private static volatile LazyHungerSingleton singleton = null;

    public static LazyHungerSingleton getInstance() throws InterruptedException {
        if (singleton == null) {
            synchronized (LazyHungerSingleton.class) {
                // double check 双重检测锁(可能会发生指令重排序，导致线程不安全)
                if (singleton == null) {
                    singleton = new LazyHungerSingleton();
                }
            }
        }
        return singleton;
    }

    private LazyHungerSingleton() {}

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    System.out.println(LazyHungerSingleton.getInstance());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }).start();
        }
    }
}
```