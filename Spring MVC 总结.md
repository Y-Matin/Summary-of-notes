# Spring mvc 总结
## 
## 注意点：
1. 当页面显示为jsp文件的源代码时，此错误的发生应该是由于在spring mvc 配置文件中，disptcherServlet 的 接受请求的路径配置的有问题。
在配置路径是 应该是"/"而不是"/*":

        <!--2.spring MVC前端控制器 ，负责拦截请求-->
        <servlet>
            <servlet-name>dispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--如果没有设置springmvc的配置文件路径，则默认是在web.xml统计目录下的 servlet-name后加servlet 作为配置文件的名字-->
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>dispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
2. 