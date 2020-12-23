### Spring-处理器-02（BeanFactoryPostcessor）
- #### 简介  
> 容器后处理器在bean创建之前，修改bean的定义属性
- #### 生命周期
>``(BeanFactoryPostcessor)``->代码块->实例化(constructor)->数据装配(@value?)->``(BeanPostcessor)``->初始化方法(init-method)->``(BeanPostcessor)``->就绪->使用->销毁方法->从容器销毁
-  #### 练习实例：
> (在Spring-处理器-01的基础上添加了BeanFactoryPostcessor、代码块)
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
1.  在Book类中实现constructor方法、init-method方法、destroy-method方法
   
> Book.java
```
package com.example.demo01;

public class Book {
    private String name;
    {
        System.out.println("调用Book：代码块方法");
    }
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
2.  实现一个BeanPostProcessor 类

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

3.  实现一个BeanFactoryPostProcessor 类
   
```
package com.example.demo01;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Service;

@Service
public class BeanFactoryPostProcessorImpl implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("调用 BeanFactoryPostProcessor 中的方法");
    }
}

```
>在调用spring boot启动类后，运行结果如下：
```
调用 BeanFactoryPostProcessor 中的方法
调用 BeanPostProcessor:(before) class com.example.demo01.DemoApplication$$EnhancerBySpringCGLIB$$3aa44f4b
BeanPostProcessor(after) class com.example.demo01.DemoApplication$$EnhancerBySpringCGLIB$$3aa44f4b
调用 BeanPostProcessor:(before) class com.example.demo01.BeanConfigAndLifeCycle$$EnhancerBySpringCGLIB$$3fe7f055
BeanPostProcessor(after) class com.example.demo01.BeanConfigAndLifeCycle$$EnhancerBySpringCGLIB$$3fe7f055
开始new一个Book对象
调用Book：代码块方法
调用Book：constructor方法
调用 BeanPostProcessor:(before) class com.example.demo01.Book
调用Book：init-method方法
BeanPostProcessor(after) class com.example.demo01.Book
```
**结论**:  
>执行顺序：BeanFactoryPostProcessor->代码块->构造方法—>BeanPostProcessor的before—>init-method—>BeanPostProcessor的after。 