<!-- ## Spring 事务失效的场景以及对应的解决方案 -->

1.  **数据库引擎是不支持事务**
    > 这里以 MySQL 为例，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB  
    从 MySQL 5.5.5 开始的默认存储引擎是：InnoDB，之前默认的都是：MyISAM，所以这点要值得注意，底层引擎不支持事务再怎么搞都是白搭。
    ***
2. 该bean没有被spring容器管理
    ```java
    // @Service
    public class OrderServiceImpl implements OrderService {

        @Transactional
        public void updateOrder(Order order) {
            // update order
        }

    }
    ```
    > 把 @Service 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。因为spring是通过AOP来实现代理，完成事务的增强功能的。
***
3. 事务注解修饰的方法不是public的
    > When using proxies, you should apply the @Transactional annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the @Transactional annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ (see below) if you need to annotate non-public methods.
    > 大概意思就是 @Transactional 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式。  
    > 原因同理：非publlic方法无法被代理类继承。无法完成增强功能。
***
4. 自身调用问题
    - 来看两个示例：

        ```java
        @Service
        public class OrderServiceImpl implements OrderService {

            public void update(Order order) {
                updateOrder(order);
            }

            @Transactional
            public void updateOrder(Order order) {
                // update order
            }

        }
        ```
        > update方法上面没有加 @Transactional 注解，调用有 @Transactional 注解的 updateOrder 方法，updateOrder 方法上的事务管用吗？

    - 示例2：
        ```java
        @Service
        public class OrderServiceImpl implements OrderService {

            @Transactional
            public void update(Order order) {
                updateOrder(order);
            }

            @Transactional(propagation = Propagation.REQUIRES_NEW)
            public void updateOrder(Order order) {
                // update order
            }
        }
        ```
        > 这次在 update 方法上加了 @Transactional，updateOrder 加了 REQUIRES_NEW 新开启一个事务，那么新开的事务管用么？
    > 这两个例子的答案是：不管用！  
    > 因为它们发生了自身调用，就调该类自己的方法，而没有经过 Spring 的代理类，默认只有在外部调用事务才会 生效，这也是老生常谈的经典问题了。  
    > 这个的解决方案之一就是在的类中注入自己，用注入的对象再调用另外一个方法，这个不太优雅，另外一个可行的方案可以参考《Spring 如何在一个事务中开启另一个事务？》这篇文章。

***
5. 事务传播机制设置错误

    ```java
    @Service
    public class OrderServiceImpl implements OrderService {

        @Transactional
        public void update(Order order) {
            updateOrder(order);
        }

        @Transactional(propagation = Propagation.NOT_SUPPORTED)
        public void updateOrder(Order order) {
            // update order
        }
    }
    ```
    > **Propagation.NOT_SUPPORTED**： 表示不以事务运行，当前若存在事务则挂起，详细的可以参考《事务隔离级别和传播机制》这篇文章。

***
6. 异常未抛出，或者抛出的异常类型错误
    - 未抛异常是比较常见的场景
        ```java
        @Service
        public class OrderServiceImpl implements OrderService {

            @Transactional
            public void updateOrder(Order order) {
                try {
                    // update order
                } catch {

                }
            }
        }

        ```
        > 未抛异常，就无法让程序感知到出错，无法触发回滚操作。
    - 抛出的事务类型不匹配 
        ```java
        @Service
        public class OrderServiceImpl implements OrderService {

            @Transactional
            public void updateOrder(Order order) {
                try {
                    // update order
                } catch {
                    throw new Exception("更新错误");
                }
            }

        }
        ```
        > 这样事务也是不生效的，因为默认回滚的是：**RuntimeException** 类型  
        如果你想触发其他异常的回滚，需要在注解上配置一下，如：
        
        ```text
        <!-- 这个配置仅限于 Throwable 异常类及其子类。 -->
        @Transactional(rollbackFor = Exception.class)
        ```
        
。。。。。


