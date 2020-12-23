## 超详细的Java代理模式

<!-- TOC -->

- [超详细的Java代理模式](#超详细的java代理模式)
  - [1. 什么代理](#1-什么代理)
    - [1.1. 常见的代理UML关系图：](#11-常见的代理uml关系图)
  - [2. 代理种类](#2-代理种类)
  - [3. 静态代理](#3-静态代理)
    - [3.1. 需求背景](#31-需求背景)
    - [3.2. **总结：**](#32-总结)
    - [3.3. **优点**](#33-优点)
    - [3.4. **缺点**](#34-缺点)
  - [4. jdk动态代理](#4-jdk动态代理)
    - [4.1. 需求背景](#41-需求背景)
  - [5. JDK动态代理到底是怎么实现的呢](#5-jdk动态代理到底是怎么实现的呢)
  - [6. cglib动态代理](#6-cglib动态代理)
    - [6.1. cglib总结：](#61-cglib总结)
  - [7. **总结**](#7-总结)

<!-- /TOC -->

### 1. 什么代理
```text
    代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。
```
#### 1.1. 常见的代理UML关系图：
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/proxy.png)
这是常见代理模式常见的 UML 示意图。 需要注意的有下面几点：

1. 用户只关心接口功能，而不在乎谁提供了功能。上图中接口是 Subject。
2. 接口真正实现者是上图的 RealSubject，但是它不与用户直接接触，而是通过代理。
3. 代理就是上图中的 Proxy，由于它实现了 Subject 接口，所以它能够直接与用户接触。
4.  用户调用 Proxy 的时候，Proxy 内部调用了 RealSubject。所以，Proxy 是中介者，它可以增强 RealSubject 操作。

### 2. 代理种类
- 静态代理
- 动态代理
### 3. 静态代理
#### 3.1. 需求背景
> 电影是电影公司委托给影院进行播放的，但是影院可以在播放电影的时候，产生一些自己的经济收益，比如提供按摩椅，娃娃机（这个每次去电影院都会尝试下，基本上是夹不起来，有木有大神可以传授下诀窍），卖爆米花、饮料（贵的要死，反正吃不起）等。我们平常去电影院看电影的时候，在电影开始的阶段是不是经常会放广告呢？然后在影片开始结束时播放一些广告。 下面我们通过代码来模拟下电影院这一系列的赚钱操作。 首先得有一个接口，通用的接口是代理模式实现的基础。
- **这个接口我们命名为 Movie**，代表电影播放的能力。
    ```java
    package com.workit.demo.proxy;

    public interface Movie {
        void play();
    }

    ```

- 接下来我们要**创建一个真正的实现这个 Movie 接口的类**，和一个实现该接口的代理类。 真正的类《美国队长》电影：
    ```java
    package com.workit.demo.proxy;

    public class CaptainAmericaMovie implements Movie {
        @Override
        public void play() {
            System.out.println("普通影厅正在播放的电影是《美国队长》");
        }
    }
    ```

- **代理类：**
    ```java
    package com.workit.demo.proxy;

    public class MovieStaticProxy implements Movie {
        Movie movie;

        public MovieStaticProxy(Movie movie) {
            this.movie = movie;
        }

        @Override
        public void play() {
            playStart();
            movie.play();
            playEnd();
        }

        public void playStart() {
            System.out.println("电影开始前正在播放广告");
        }
        public void playEnd() {
            System.out.println("电影结束了，接续播放广告");
        }
    }
    ``` 
- **测试类：**
    ```java
    package com.workit.demo.proxy;

    package com.workit.demo.proxy;

    public class StaticProxyTest {
        public static void main(String[] args) {
            Movie captainAmericaMovie = new CaptainAmericaMovie();
            Movie movieStaticProxy = new MovieStaticProxy(captainAmericaMovie);
            movieStaticProxy.play();
        }
    }
    ```
    ***
- **运行结果：**
    ```java
    电影开始前正在播放广告
    正在播放的电影是《美国队长》
    电影结束了，接续播放广告
    ```
#### 3.2. **总结：**
  1. 现在可以看到，代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。
  2. 值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。这个就是是静态代理的内容.  
  3. 为什么叫做静态呢？因为它的类型是事先预定好的，比如上面代码中的MovieStaticProxy 这个类。
#### 3.3. **优点**
  1. 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用
  2. 代理对象可以扩展目标对象的功能
  3. 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度。
#### 3.4. **缺点**
  1. 代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护。

### 4. jdk动态代理
```text
    与静态代理类对照的是动态代理类，动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。java.lang.reflect 包中的Proxy类和InvocationHandler 接口提供了生成动态代理类的能力。
```
#### 4.1. 需求背景
> 接着上面的例子，刚看完《美国队长》不过瘾，还想继续去看一场《钢铁侠》。一直在普通影厅看电影觉得没啥意思，那就赶紧去VIP影厅（至今不知道长啥样子）体验一把。既然 实体店没体验过那就用代码来体验一次吧。
- 创建一个VIPMovie电影接口
    ```java
    package com.workit.demo.proxy;
    public interface VIPMovie {
        void vipPlay();
    }
    ```
- 紧接着创建一个VIP影厅的播放实现类
    ```java
    package com.workit.demo.proxy;

    public class IronManVIPMovie implements VIPMovie {
        @Override
        public void vipPlay() {
            System.out.println("VI影厅正在播放的电影是《钢铁侠》");
        }
    }
    ```
- 如果按照静态代理我们是不是又要创建一个VIP影厅播放的代理实现类，这种方式我们就不演示了。下面我们来看看通过动态代理怎么来实现吧。
    ```java
    package com.workit.demo.proxy;

    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;

    public class MyInvocationHandler implements InvocationHandler {

        private Object object;

        public MyInvocationHandler(Object object) {
            this.object = object;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            playStart();
            Object invoke = method.invoke(object, args);
            playEnd();
            return invoke;
        }

        public void playStart() {
            System.out.println("电影开始前正在播放广告");
        }
        public void playEnd() {
            System.out.println("电影结束了，接续播放广告");
        }
    }
    ```
- MyInvocationHandler实现了 InvocationHandler 这个类，这个类是什么意思呢？大家不要慌张，下面我会解释。然后，我们就可以在VIP影厅看电影了。
    ```java
    package com.workit.demo.proxy;

    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Proxy;

    public class DynamicProxyTest {
        public static void main(String[] args) {
            IronManVIPMovie ironManVIPMovie = new IronManVIPMovie();
            InvocationHandler invocationHandler = new MyInvocationHandler(ironManVIPMovie);
            VIPMovie dynamicProxy = (VIPMovie) Proxy.newProxyInstance(IronManVIPMovie.class.getClassLoader(),
                    IronManVIPMovie.class.getInterfaces(), invocationHandler);
            dynamicProxy.vipPlay();
        }
    }
    ```
- **运行结果**
    ```java
    电影开始前正在播放广告
    VI影厅正在播放的电影是《钢铁侠》
    电影结束了，接续播放广告
    ```
- 看到没有，我们并没有像静态代理那样为 VIPMovie接口实现一个代理类，但最终它仍然实现了相同的功能，这其中的差别，就是之前讨论的动态代理所谓“动态”的原因。 我们顺带把《美国队长》也用动态代理实现下吧。
    ```java
    package com.workit.demo.proxy;

    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Proxy;

    public class DynamicProxyTest {
        public static void main(String[] args) {
            // VIP 影厅《钢铁侠》
            IronManVIPMovie ironManVIPMovie = new IronManVIPMovie();
            InvocationHandler invocationHandler = new MyInvocationHandler(ironManVIPMovie);
            VIPMovie dynamicProxy = (VIPMovie) Proxy.newProxyInstance(IronManVIPMovie.class.getClassLoader(),
                    IronManVIPMovie.class.getInterfaces(), invocationHandler);
            dynamicProxy.vipPlay();
            
            // 普通影厅《美国队长》
            CaptainAmericaMovie captainAmericaMovie = new CaptainAmericaMovie();
            InvocationHandler invocationHandler1 = new MyInvocationHandler(captainAmericaMovie);
            Movie dynamicProxy1 = (Movie) Proxy.newProxyInstance(CaptainAmericaMovie.class.getClassLoader(),
                    CaptainAmericaMovie.class.getInterfaces(), invocationHandler1);
            dynamicProxy1.play();
        }
    }
    ```
- **运行结果**
    ```java
    电影开始前正在播放广告
    VI影厅正在播放的电影是《钢铁侠》
    电影结束了，接续播放广告
    电影开始前正在播放广告
    正在播放的电影是《美国队长》
    电影结束了，接续播放广告
    ```
**我们通过 Proxy.newProxyInstance() 方法，却产生了 Movie和 VIPMovie两种接口的实现类代理，这就是动态代理的魔力。**
### 5. JDK动态代理到底是怎么实现的呢
> 动态代码涉及了一个非常重要的类 Proxy。正是通过 Proxy 的静态方法 newProxyInstance 才会动态创建代理。具体怎么去创建代理类就不分析了,感兴趣的可以去看下源码。我们直接看下生成的代理类。 如何查看生成的代理类？
    ```java
    //在生成代理类之前加上以下代码(我用的jdk1.8)：

    //新版本 jdk产生代理类   
    System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
    ```

- 最终代码结构如下：
    ```java
    public static void main(String[] args) {
            //新版本 jdk产生代理类
        System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
            // VIP 影厅《钢铁侠》
            IronManVIPMovie ironManVIPMovie = new IronManVIPMovie();
            InvocationHandler invocationHandler = new MyInvocationHandler(ironManVIPMovie);
            VIPMovie dynamicProxy = (VIPMovie) Proxy.newProxyInstance(IronManVIPMovie.class.getClassLoader(),
                    IronManVIPMovie.class.getInterfaces(), invocationHandler);
            dynamicProxy.vipPlay();

            // 普通影厅《美国队长》
            CaptainAmericaMovie captainAmericaMovie = new CaptainAmericaMovie();
            InvocationHandler invocationHandler1 = new MyInvocationHandler(captainAmericaMovie);
            Movie dynamicProxy1 = (Movie) Proxy.newProxyInstance(CaptainAmericaMovie.class.getClassLoader(),
                    CaptainAmericaMovie.class.getInterfaces(), invocationHandler1);
            dynamicProxy1.play();
            System.out.println("VIP 影厅《钢铁侠》代理类："+dynamicProxy.getClass());
            System.out.println("普通影厅《美国队长》："+dynamicProxy1.getClass());
        }
    ```
- 可以看到结果如下：
    ```java
    电影开始前正在播放广告
    VI影厅正在播放的电影是《钢铁侠》
    电影结束了，接续播放广告
    电影开始前正在播放广告
    正在播放的电影是《美国队长》
    电影结束了，接续播放广告
    VIP 影厅《钢铁侠》代理类：class com.sun.proxy.$Proxy0
    普通影厅《美国队长》：class com.sun.proxy.$Proxy1
    ```

- 产生了两个代理类分别是$Proxy0和$Proxy1。 下面们来看下"钢铁侠"的代理类$Proxy0
    ```java
    public final class $Proxy0 extends Proxy implements VIPMovie {
        private static Method m1;
        private static Method m3;
        private static Method m2;
        private static Method m0;

        public $Proxy0(InvocationHandler var1) throws  {
            super(var1);
        }

        public final boolean equals(Object var1) throws  {
            try {
                return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }

        public final void vipPlay() throws  {
            try {
                super.h.invoke(this, m3, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final String toString() throws  {
            try {
                return (String)super.h.invoke(this, m2, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final int hashCode() throws  {
            try {
                return (Integer)super.h.invoke(this, m0, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        static {
            try {
                m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
                m3 = Class.forName("com.workit.demo.proxy.VIPMovie").getMethod("vipPlay");
                m2 = Class.forName("java.lang.Object").getMethod("toString");
                m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            } catch (NoSuchMethodException var2) {
                throw new NoSuchMethodError(var2.getMessage());
            } catch (ClassNotFoundException var3) {
                throw new NoClassDefFoundError(var3.getMessage());
            }
        }
    }

    ```
    **通过上述代码我们可以看到 $Proxy0 extends Proxy implements VIPMovie继承了Proxy 且实现了VIPMovie接口，这也就是为什么jdk动态代理必须基于接口,java 是单继承的。**
-  然后再看下代理类实现的方法：
    ```java
    public final void vipPlay() throws  {
            try {
                super.h.invoke(this, m3, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }
    ```
    **这个supper.h.invoke Proxy中的h的invoke方法，即InvocationHandler.invoke也就是上面 MyInvocationHandler.invoke方法，至此整个流程就清晰了。这就是jdk的动态代理。**

### 6. cglib动态代理
> 上面说jdk动态代理只能基于接口，那么如果是类要动态代理怎么办呢？cglib动态代理就可解决关于类的动态代理。 
- 下面我们来创建一个“《美国队长2》”
    ```java
    public class CaptainAmerica2MovieImpl {

        public void play(){
            System.out.println("正在播放的电影是《美国队长2》");
        }
    }
    ```
- 创建一个自定义的MethodInterceptor（拦截器）
    ```java
    import net.sf.cglib.proxy.MethodInterceptor;
    import net.sf.cglib.proxy.MethodProxy;

    import java.lang.reflect.Method;

    public class CglibProxyInterceptor implements MethodInterceptor {
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            playStart();
            Object object = methodProxy.invokeSuper(o, objects);
            playEnd();
            return object;
        }

        public void playStart() {
            System.out.println("电影开始前正在播放广告");
        }

        public void playEnd() {
            System.out.println("电影结束了，接续播放广告");
        }
    }
    ```
- 测试类
  
    ```java
    import net.sf.cglib.core.DebuggingClassWriter;
    import net.sf.cglib.proxy.Enhancer;

    public class CglibProxyTest {
        public static void main(String[] args) {
            // //在指定目录下生成动态代理类
            System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "C:\\class");
            //创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
            Enhancer enhancer = new Enhancer();
            //设置目标类的字节码文件
            enhancer.setSuperclass(CaptainAmerica2MovieImpl.class);
            //设置回调函数
            enhancer.setCallback(new CglibProxyInterceptor());
            //这里的creat方法就是正式创建代理类
            CaptainAmerica2MovieImpl captainAmerica2Movie = (CaptainAmerica2MovieImpl)enhancer.create();
            //调用代理类的play方法
            captainAmerica2Movie.play();
            System.out.println("cglib动态代理《美国队长2》："+captainAmerica2Movie.getClass());
        }
    }
    ```
- 输出结果：

    ```java
    电影开始前正在播放广告
    正在播放的电影是《美国队长2》
    电影结束了，接续播放广告
    cglib动态代理《美国队长2》：class com.workit.demo.proxy.CaptainAmerica2MovieImpl$$EnhancerByCGLIB$$5c3ddcfe
    ```
- 我们看下最终创建的代理类生成的play方法
    ```java
    public class CaptainAmerica2MovieImpl$$EnhancerByCGLIB$$5c3ddcfe extends CaptainAmerica2MovieImpl implements Factory {
    public final void play() {
            MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
            if (var10000 == null) {
                CGLIB$BIND_CALLBACKS(this);
                var10000 = this.CGLIB$CALLBACK_0;
            }

            if (var10000 != null) {
                var10000.intercept(this, CGLIB$play$0$Method, CGLIB$emptyArgs, CGLIB$play$0$Proxy);
            } else {
                super.play();
            }
        }
    ```
    **从代理对象反编译源码可以知道，代理对象继承于CaptainAmerica2MovieImpl ，拦截器调用intercept()方法， intercept()方法由自定义CglibProxyInterceptor实现，所以，最后调用CglibProxyInterceptor中的intercept()方法，从而完成了由代理对象访问到目标对象的动态代理实现。**
#### 6.1. cglib总结：
1. CGlib是一个强大的,高性能,高质量的Code生成类库。它可以在运行期扩展Java类与实现Java接口。
2. 用CGlib生成代理类是目标类的子类。
3. 用CGlib生成 代理类不需要接口。
4. 用CGLib生成的代理类重写了父类的各个方法。
5. 拦截器中的intercept方法内容正好就是代理类中的方法体。

### 7. **总结**
- 代理分为静态代理和动态代理两种。
- 静态代理，代理类需要自己编写代码写成。
- 动态代理有**jdk**和**cglib**，代理类通过 Proxy.newInstance()或者ASM 生成。
- 静态代理和动态代理的区别是在于要不要开发者自己定义 Proxy 类。 动态代理通过 Proxy 动态生成 proxy class，但是它也指定了一个 InvocationHandler 或者 MethodInterceptor的实现类。
- 代理模式本质上的目的是为了增强现有代码的功能。

*本文章转载于微信工作号<java金融>*
