### Spring学习总结：
1.sprring 中 ioc 容器 依赖于一下4个jar包  
        spring-core、  
        spring-beans、  
        spring-context、  
        spring-expression  
2.spring配置xml中 配置bean时：<property>标签 的name属性指的是 该bean中setXXX方法的XXX首字母小写，<setter注入>，一半XXX是根据成员名智能创建的，可以等同于成员变量名  
2.ioc容器类型：  
```
        applicationContext 
                ClassPathXmlApplicationContext
                FileSystemXmlApplicationCOntext
        BeanFactory
                XmlBeanFactory 已过时
```
3.数据装配：对类中的成员属性进行初始化。
        1.八种基本类型、以及对应的包装类，和特殊的类型：String
                byte、short、int、long、float、double、char、boolean
                Byte、Short、Integer、Long、Float、Double、Character、Boolean
              赋值时都是使用value属性进行赋值。
        2.包含Bean，比如在IOC容器中声明的javaBean，使用ref属性对其进行赋值
        3.对于集合类型：数组、List、Set、Map、Properties
          在<property>标签中，再添加一个子标签，比如：<array><lsit>......
```
                数组：
                <property name="ageArray" >
                        <array>
                                <value>1</value>
                                <value>2</value>
                                <value>3</value>
                        </array>
                </property>
                List：
                <!--对list集合进行初始化-->
                <property name="usersList" >
                <list >
                        <ref bean="User"></ref>
                        <ref bean="User"></ref>
                        <bean class="ioc03.User">
                        <property name="name" value="yds"></property>
                        <property name="age" value="24"></property>
                        </bean>
                </list>
                </property>

                Set：
                <!--Set集合的初始化-->
                <property name="userSet">
                <set>
                        <ref bean="User"></ref>
                        <ref bean="User"></ref>
                        <bean class="ioc03.User">
                        <property name="age" value="12"></property>
                        <property name="name" value="name"></property>
                        </bean>
                </set>
                </property>
```
