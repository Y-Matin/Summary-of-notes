# Spring Cloud 核心组件使用
### 核心组件有：
> 1. eureka....
## eureka 使用及总结
1. Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。
2. Eureka包含两个组件：Eureka Server和Eureka Client
Eureka Server提供服务注册服务
各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到.
3. EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）.
4. 必备的三个角色：
   > 1. Eureka Server 提供服务注册和发现
   > 2. Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到
   > 3. Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务
5. Eureka Server==注册中心 的构建：
   > 1. 添加依赖，导入jar包，server模块的Pom文件中：  
     ```
        <dependency>  
               <groupId>org.springframework.cloud</groupId>  
               <artifactId>spring-cloud-starter-eureka-server</artifactId>  
            </dependency>   
     ```
    > 2. 配置注册中心的 域名，端口，以及访问路径：      
    ```
        server:
          port: 7001
        eureka:
          instance:
            hostname: eureka7001.com #eureka服务端的实例名称
          client:
            #false表示不向注册中心注册自己。
            register-with-eureka: false     
            #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
            fetch-registry: false     
            service-url:
              #该行配置 代表目前只有一台服务器
              #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
              #该行配置 代表 目前有多台服务器，其他服务器的地址要写入defaultZone中声明。
              defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
     ```
    > 3. 在该模块的启动类中添加 注解: "@EnableEurekaServer"，表明当前是eureka的服务器-server端，负责维护服务实例
6. 微服务的注册： 
   > 1.添加依赖，导入jar包： 在需要注册微服务模块中pom文件中添加 eureka client的依赖.
    ```
    <!-- 将微服务provider侧注册进eureka -->
       <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-eureka</artifactId>
       </dependency>
    ```
    > 2. 配置注册中心的地址：
    ```
    eureka:
      client: #客户端注册进eureka服务列表内
        service-url: 
          #单个eureka 注册中心
          #defaultZone: http://localhost:7001/eureka
          #多个 注册中心
          defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    ```
    > 3. 在微服务模块的启动类中添加注解："@EnableEurekaClient"。表明当前模块是一个客户端。 
7. actuator与注册微服务信息完善，优化
   > 1. 主机名称:服务名称修改:在yml配置文件中添加对实例id的配置
   ```
   eureka:
     instance:
        #自定义服务名称信息
       instance-id: microservicecloud-dept8001
   ```
   > 2. 访问信息添加IP信息提示
   ```
   eureka:
     instance:
        #访问路径可以显示IP地址
       prefer-ip-address: true     
   ```
   > 3. 微服务info内容详细信息   
        1. 先在微服务端添加pom依赖：
    ```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>    
    ```
    2......
8. 集群配置: 当存在多个eureka服务器时，假设本地有三台eureka注册服务器
   > 1. 修改host文件，添加映射：域名与ip地址的映射关系:   
   127.0.0.1  eureka7001.com  
   127.0.0.1  eureka7002.com  
   127.0.0.1  eureka7003.com
   > 2. 分别在注册服务的模块中将eureka.instance.hostname修改为对应的域名； 在将eureka.client.service-url.defaultZone的值修改为其他注册服务器的域名。如下面所示：
    ```
    eureka:
      instance:
          #eureka服务端的实例名称
        hostname: eureka7001.com 
      client:
        service-url:
            #其他服务器的地址
          defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
   ```
   
   


    




