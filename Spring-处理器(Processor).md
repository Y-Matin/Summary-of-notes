### Spring-后(置)处理器
#### 1. 处理器的类型
- Bean后处理器，实现BeanPostProcesesor接口
- BeanFactory后处理器，实现BeanFactoryPostProcessor接口，也成为容器后处理器
#### 2. BeanPostcessor
##### 2.1 简介
> Bean后处理器：是用来对Bean的功能进行扩展增强，对ioc容器中的所有Bean都有效。  
> 时机：执行初始化方法 之前和之后  

背景  
> Bean的生命周期:代码块->实例化->数据装配-> (Postccessor)->初始化方法->(Postcessor)->就绪->使用->销毁方法->从容器销毁
  