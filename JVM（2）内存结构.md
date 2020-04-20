## JVM内存结构
<!-- TOC -->

- [JVM内存结构](#jvm%e5%86%85%e5%ad%98%e7%bb%93%e6%9e%84)
  - [1. 背景](#1-%e8%83%8c%e6%99%af)
  - [2. JVM 内存结构](#2-jvm-%e5%86%85%e5%ad%98%e7%bb%93%e6%9e%84)
    - [2.1. Java堆（Heap）](#21-java%e5%a0%86heap)
    - [2.2. 方法区（Method Area）](#22-%e6%96%b9%e6%b3%95%e5%8c%bamethod-area)
    - [2.3. 程序计数器（Program Counter Register）](#23-%e7%a8%8b%e5%ba%8f%e8%ae%a1%e6%95%b0%e5%99%a8program-counter-register)
    - [2.4. 虚拟机栈](#24-%e8%99%9a%e6%8b%9f%e6%9c%ba%e6%a0%88)
      - [2.4.1. 栈帧](#241-%e6%a0%88%e5%b8%a7)
    - [2.5. 本地方法栈（Native Method Stacks）](#25-%e6%9c%ac%e5%9c%b0%e6%96%b9%e6%b3%95%e6%a0%88native-method-stacks)
  - [3. JVM常见参数以及对应空间位置](#3-jvm%e5%b8%b8%e8%a7%81%e5%8f%82%e6%95%b0%e4%bb%a5%e5%8f%8a%e5%af%b9%e5%ba%94%e7%a9%ba%e9%97%b4%e4%bd%8d%e7%bd%ae)
  - [4. 内存管理](#4-%e5%86%85%e5%ad%98%e7%ae%a1%e7%90%86)

<!-- /TOC -->
### 1. 背景
> 所有的Java开发人员可能会遇到这样的困惑:  
> 1. 我该为堆内存设置多大空间呢？  
> 2. 如何最大化的利用机器的空间呢？  
> 3. OutOfMemoryError的异常到底涉及到运行时数据的哪块区域？该怎么解决呢？

### 2. JVM 内存结构
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/JVM_detail.png)
***
**方法区和堆是所有线程共享的内存区域；而java栈、本地方法栈和程序员计数器是运行是线程私有的内存区域。**
***
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/jvm-runData.png)

#### 2.1. Java堆（Heap）
1. Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。
2.  Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建
3. 此内存区域的唯一目的就是存放对象实例。
4. 根据Java虚拟机规范的规定Java堆可以处于物理上不连续的内存空间中
5. 通过-Xmx和-Xms控制）。

#### 2.2. 方法区（Method Area）
1. 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域
2. 它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
3. 这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载
#### 2.3. 程序计数器（Program Counter Register）
1. 程序计数器（Program Counter Register）是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器. 
2. 由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。
3. 如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie方法，这个计数器值则为空（Undefined）。  
**此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。**
#### 2.4. 虚拟机栈
1. Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。 
   
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/stackFrame.jpg) 
1. 局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。

2. 在Java虚拟机规范中，对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常
3. 如果虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出OutOfMemoryError异常。
##### 2.4.1. 栈帧
1. 局部变量：用于存放方法参数和方法内部定义的局部变量
2. 操作数栈：用于计算，做出栈，入栈操作
3. 动态链接：栈帧持有一个指向方法区常量池中所属方法的引用。这个引用是为了支持动态连接。动态连接是指符号引用在运行时转化为直接引用。
4. 返回地址：在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信息
#### 2.5. 本地方法栈（Native Method Stacks）
1. 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。
2. native 方法与 系统底层进行交付，比如本地的 DDL文件。
3. 与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。


### 3. JVM常见参数以及对应空间位置
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/JVM-param.png)
**控制参数**
- -Xms设置堆的最小空间大小。
- -Xmx设置堆的最大空间大小。
- -XX:NewSize设置新生代最小空间大小。
- -XX:MaxNewSize设置新生代最大空间大小。
- -XX:PermSize设置永久代最小空间大小。
- -XX:MaxPermSize设置永久代最大空间大小。
- -Xss设置每个线程的堆栈大小。
-没有直接设置老年代的参数，但是可以设置堆空间大小和新生代空间大小两个参数来间接控制。  老年代空间大小=堆空间大小-年轻代大空间大小
### 4. 内存管理

只有垃圾收集器才能释放分配在堆中的对象和数组，工作流程如下：

1. 新对象和数组分配到新生代的Eden区；

2. 新生代发生Major GC时，对象或数组如果仍然存活，则会从Eden区移到Survivor区；

3. Major GC通常会导致应用线程暂停，它会在两个区中移动对象，如果对象仍然存活，并达到设定的年龄后，将会从新生代移到老年代；

4. 当老年代进行垃圾回收时，会同时触发持久代也进行一次回收。同样，在发生Full GC的时候也会触发这两个回收。
