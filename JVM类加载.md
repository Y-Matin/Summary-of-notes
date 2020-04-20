<!-- ## JVM类加载 -->
### JVM的基本结构
- 类加载子系统
- 运行时数据区（内存结构）
- 执行引擎
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/jvm-all.png)

### 类加载子系统
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/jvm-classLoad.png)
#### 加载
- 将.class文件从磁盘中读到内存

#### 连接

##### 验证
- 验证字节码文件的正确性
##### 准备
- 给类的静态变量分配内存，并呼吁默认值
##### 解析
- 类装载器装入类所引用的其他所有类（静态链接）
#### 初始化
- 为类的静态变量赋予正确的初始值，上述的**准备**阶段为静态变量赋予的是虚拟机默认的初始值，此处赋予的才是开发人员为变量分配的真正的初始值，指定静态代码块
#### 使用
#### 卸载
***
### 类加载器的种类
- 不同的类加载器负责加载的路径不同
- 如何唯一标识类：
  - ①.该类的全限定类名
  - ②.加载该类的类加载器

![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/classLoadType.png)

1. 启动类加载器（Bootstrap ClassLoader）
    > 负责加载JRE中核心类库，如JRE目录下的rt.jar等
2. 扩展类加载器（Extension ClassLoader）
    > 负责加载JRE扩展目录ext中的jar包中的类
3. 系统类加载器（Application ClasssLoader）
    > 负责加载ClassPath路径下的类
4. 用户自定义加载器（User ClassLoader）
    > 负责加载用户自定义路径下的类包

***
### 双亲委派机制

#### 工作过程
1. 类加载器收到类加载的请求；
2. 把这个请求委托给父加载器去完成，一直向上委托，直到启动类加载器；
3. 启动器加载器检查能不能加载（使用findClass()方法），能就加载（结束）；否则，抛出异常，通知子加载器进行加载。
4. 重复步骤三；
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/parentLoadRule.png)

#### 优点
1. 可以避免重复加载，父类已经加载了，子类就不需要再次加载
2. 安全性高，避免用户使用自定义的类加载器来加载核心api


#### 自定义类加载
- 那么我们如何自己定义类加载器呢？这主要有两种方式
  1. 遵守双亲委派模型：继承ClassLoader，**重写findClass()方法。**
  2. 破坏双亲委派模型：继承ClassLoader,**重写loadClass()方法。** 通常我们推荐采用第一种方法自定义类加载器，最大程度上的遵守双亲委派模型。