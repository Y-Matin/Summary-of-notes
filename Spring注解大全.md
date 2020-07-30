Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。 
注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。包含在 java.lang.annotation 包中。

## **1、元注解**

元注解是指注解的注解。包括  @Retention @Target @Document @Inherited四种。
###   @Retention 定义注解的保留策略
功能：指明修饰的注解的生存周期，即会保留到哪个阶段。
RetentionPolicy的取值包含以下三种：

    SOURCE：源码级别保留，编译后即丢弃。
    CLASS:编译级别保留，编译后的class文件中存在，在jvm运行时丢弃，这是默认值。
    RUNTIME： 运行级别保留，编译后的class文件中存在，在jvm运行时保留，可以被反射调用。

### @Target：定义注解的作用目标
ElementType的取值包含以下几种： 

    TYPE:类，接口或者枚举
    FIELD:域，包含枚举常量
    METHOD:方法
    PARAMETER:参数
    CONSTRUCTOR:构造方法
    LOCAL_VARIABLE:局部变量
    ANNOTATION_TYPE:注解类型
    PACKAGE:包

由以上的源码可以知道，他的elementType 可以有多个，一个注解可以为类的，方法的，字段的等等
### @Documented 注解
功能：指明修饰的注解，可以被例如javadoc此类的工具文档化，只负责标记，没有成员取值
### @Inherited注解
功能：允许子类继承父类中的注解。
注意！：
@interface意思是声明一个注解，方法名对应参数名，返回值类型对应参数类型。

##  常见注解
### @component 
功能：将普通pojo添加到spring 容器中 等同于在xml配置中：<bean id="" class=""/>
使用场景：泛指组件，当组件不好归类<repostory、service、controller...>的时候，我们可以使用这个注解进行标注。
### @repository @service @controller 
功能：与@component功能几乎一致，使用具体的层次注解，对spirng跟友好一些。
### @ResponseBody

### @RestController
功能：由@ResponseBody和 @Controller注解修饰
### @bean 

### @Configuration  
相当于一个spring的xml文件，修饰语类上。里面常常配合@Bean注解，可以在里面添加需要Spring容器管理的bean。
### @EnableConfigurationProperties

### @ConfigurationProperties

### @RefreshScope

### @import 
源码：
```
/*<p>Provides functionality equivalent to the {@code <import/>} element in Spring XML.
 * Only supported for classes annotated with {@code @Configuration} or declaring at least
 * one {@link Bean @Bean} method, as well as {@link ImportSelector} and
 * {@link ImportBeanDefinitionRegistrar} implementations.*/
```
功能：导入一个类到容器中。  
由官方的文档得知：
- 导入一个类，该类至少有一个方法被@bean注解修饰。
- 导入@Configuration注解的配置类
- 导入ImportSelector的实现类
- 导入ImportBeanDefinitionRegistrar的实现类
场景：将以上四种情况的类导入容器中。
使用实例：[详情](https://blog.csdn.net/panchao888888/article/details/82882279)

### conditional

### ConditionalOnWebApplication

### ConditionalOnProperty

### ComponentScan


### PreHttpCheck

### RequestMapping PostMapping 

### @Profile

### @CheckPrd

### @Deprecated 不赞成使用的，在被调用时，会提示横线，表明虽然可用但不建议使用
### @AliasFor 
功能：给属性设置别名  
场景：  
- 在注解接口中，为同一个功能定义两个名称不一样的属性，使用次注解这两个属性彼此互为别名。
优点：  
- 当注解接口中，只有一个属性别名对时，可以省略属性名，或者说选择跟具有实际含义的属性名作为key。  

属性别名对要求：  
- 1.组成别名对的每个属性都必须用@AliasFor进行注释,并且AliasFor中的值必须指向别名对中的另一个属性
- 2.别名化的属性必须声明相同的返回类型
- 3.别名化的属性必须声明默认值
- 4.别名化的属性默认值必须相同
### @PostConstruct
执行顺序：
- @PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。