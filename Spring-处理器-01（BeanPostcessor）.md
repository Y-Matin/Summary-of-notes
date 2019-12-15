### Spring-处理器-01（BeanPostcessor）
#### 1. 处理器的类型
- Bean后处理器，实现BeanPostProcesesor接口
- BeanFactory后处理器，实现BeanFactoryPostProcessor接口，也成为容器后处理器
#### 2. BeanPostcessor
##### 2.1 简介
> Bean后处理器：是用来对Bean的功能进行扩展增强，对ioc容器中的所有Bean都有效。  
> 时机：执行初始化方法(init-method) 之前和之后  

背景  
> Bean的生命周期:代码块->实例化(constructor)->数据装配(@value?)->``(BeanPostcessor)``->初始化方法(init-method)->``(BeanPostcessor)``->就绪->使用->销毁方法->从容器销毁
具体常见过程参考:[文章](https://blog.csdn.net/mawming/article/details/52287786)  
流程图如下：
![实例化bean流程](https://static.oschina.net/uploads/space/2011/1222/204039_9370_218421.jpg)
##### 2.2 实现步骤
1. 定义一个类，实现BeanProcessor接口
2. 将该处理器添加到ioc容器中

实例：  
1.  在配置类中指定类的init-method、destroy-method方法
> BeanConfigAndLifeCycle.java  
```
package com.example.demo01;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfigAndLifeCycle {
    // 在配置类中返回一个Book对象，
    //并指定init-method、destroy-method方法
    @Bean(initMethod = "init", destroyMethod = "destroy")
    Book getBook() {
        System.out.println("开始new一个Book对象");
        return new Book();
    }

}
```
2.  在Book类中实现constructor方法、init-method方法、destroy-method方法
> Book.java
```
package com.example.demo01;

public class Book {
    private String name;

    public Book() {
        System.out.println("调用Book：constructor方法");
    }

    public void init() {
        System.out.println("调用Book：init-method方法");
    }

    public void destroy() {
        System.out.println("调用Book：destroy-Method方法");
    }
}
```
3.  实现一个BeanPostProcessor 类
> BeanPostProcessorImpl.java
```
package com.example.demo01;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Service;

@Service
public class BeanPostProcessorImpl implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用 BeanPostProcessor:(before) " + bean.getClass());
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor(after) " + bean.getClass());
        return bean;
    }

}
```
>在调用spring boot启动类后，运行结果如下：
```
调用 BeanPostProcessor:(before) class com.example.demo01.DemoApplication$$EnhancerBySpringCGLIB$$b3ab3853
BeanPostProcessor(after) class com.example.demo01.DemoApplication$$EnhancerBySpringCGLIB$$b3ab3853
调用 BeanPostProcessor:(before) class com.example.demo01.BeanConfigAndLifeCycle$$EnhancerBySpringCGLIB$$b8eed95d
BeanPostProcessor(after) class com.example.demo01.BeanConfigAndLifeCycle$$EnhancerBySpringCGLIB$$b8eed95d
开始new一个Book对象
调用Book：constructor方法
调用 BeanPostProcessor:(before) class com.example.demo01.Book
调用Book：init-method方法
BeanPostProcessor(after) class com.example.demo01.Book
```
**结论**:  
>执行顺序：构造方法—>BeanPostProcessor的before—>init-method—>BeanPostProcessor的after。 

  