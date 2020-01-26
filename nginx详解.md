## Nginx 详解
- 简介
> Nginx做为一个强大的Web服务器软件，具有高性能、高并发性和低内存占用的特点。此外，其也能够提供强大的反向代理功能。俄罗斯大约有超过20%的虚拟主机采用Nginx作为反向代理服务器,在国内也有腾讯、新浪、网易等多家网站在使用Nginx作为反向代理服务器。
- 特性
> 反向代理  
> 负载均衡  
> 动静分离

#### 1. 反向代理
- 定义
> 反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

>有反向代理，当然也存在正向代理的概念咯。正向代理指的是，一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

我们通过一个简单例子，我们就很好的理解两者之间的区别了。

![](https://user-gold-cdn.xitu.io/2016/11/30/fa276a0b51bdf992f5ceaeea3d698a17.png?imageView2/0/w/1280/h/960/ignore-error/1)
***
![](https://user-gold-cdn.xitu.io/2016/11/29/c53a707e65c569f1ff35cef04ecc5435.png?imageView2/0/w/1280/h/960/ignore-error/1)
- 为什么使用反向代理
> 1. 可以起到保护网站安全的作用，因为任何来自Internet的请求都必须先经过代理服务器。  
> 2. 通过缓存静态资源，加速Web请求。    
> 3. 实现负载均衡。顺便说下，目前市面上，主流的负载均衡方案，硬件设备有F5，软件方案有四层负载均衡的LVS，七层负载均衡的Nginx、Haproxy等。

#### 2. 负载均衡
- 均衡策略
>   nginx的负载均衡策略可以划分为两大类：内置策略和扩展策略。
内置策略包含加权轮询和ip hash，在默认情况下这两种策略会编译进nginx内核，只需在nginx配置中指明参数即可。扩展策略有很多，如fair、通用hash、consistent hash等，默认不编译进nginx内核。
##### 1. 加权轮询（weighted round robin）
> 如下是处理一次请求的流程图：  

![](https://user-gold-cdn.xitu.io/2016/11/30/a804ea9edb963db6404d0e1c4391af78.png?imageView2/0/w/1280/h/960/ignore-error/1)  

> 图中有两点需要注意，第一，如果可以把加权轮询算法分为先深搜索和先广搜索，那么nginx采用的是先深搜索算法，即将首先将请求都分给高权重的机器，直到该机器的权值降到了比其他机器低，才开始将请求分给下一个高权重的机器；第二，当所有后端机器都down掉时，nginx会立即将所有机器的标志位清成初始状态，以避免造成所有的机器都处在timeout的状态，从而导致整个前端被夯住。
##### 2.ip hash 
> ip hash是nginx内置的另一个负载均衡的策略，流程和轮询很类似，只是其中的算法和具体的策略有些变化，如下图所示:  
![](https://user-gold-cdn.xitu.io/2016/11/29/e7b6e27d1342a18f19a3fef794eb8159.png?imageView2/0/w/1280/h/960/ignore-error/1)

##### 其他策略
> fair  
通用hash、一致性hash  
session_sticky  

#### 配置示例：
##### 1.http
```
 upstream upstream_test{  
     server 192.168.0.1:8080;
     server 192.168.0.2:8080;

     #ip_hash;
     keepalive 30;

     ## tengine config
     #check interval=300 rise=10 fall=10 timeout=100 type=http port=80;
     #check_http_send "GET / HTTP/1.0\r\n\r\n";
     #check_http_expect_alive http_2xx http_3xx;

     ## tengine config
     #session_sticky cookie=cookieTest mode=insert;
 }  

 location / {

        proxy_pass         http://upstream_test;
       proxy_set_header   Host             $host;
       proxy_set_header   X-Real-IP        $remote_addr;
       proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

    }
```
##### 2. TCP - ngx_tcp_proxy_module
```
 tcp {
     upstream cluster {
         #simple round-robin

         server 192.168.0.1:8080;

         server 192.168.0.2:8080;

         check interval=3000 rise=2 fall=5 timeout=1000;
         #check interval=3000 rise=2 fall=5 timeout=1000 type=ssl_hello;
         #check interval=3000 rise=2 fall=5 timeout=1000 type=http;
         #check_http_send "GET / HTTP/1.0\r\n\r\n";
         #check_http_expect_alive http_2xx http_3xx;
     }
     server {

             listen 8888;

             proxy_pass cluster;

     }
 }
```

#### 配置结构

![](https://user-gold-cdn.xitu.io/2016/11/29/def3b184d4b73d4d890728a06f5f3072.png?imageView2/0/w/1280/h/960/ignore-error/1)