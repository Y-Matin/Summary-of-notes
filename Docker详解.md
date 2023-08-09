### Docker详解
> 简介：Docker 是一个开源的应用容器引擎，基于Go 语言并遵从Apache2.0协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口,更重要的是容器性能开销极低。  
#### 关键概念
1. 镜像(Images)：Docker 镜像是用于创建Docker 容器的模板。
2. 容器(Container)：容器是独立运行的一个或一组应用。
3. 客户端(Client)：客户端通过命令行或者其他工具使用Docker API(https://docs.docker.com/reference/api/docker_remote_api) 与Docker 的守护进程通信.
4. 主机(Host)：一个物理或者虚拟的机器用于执行Docker 守护进程和容器。
5. 仓库(Registry)：Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。
#### 常用命令<CentOS>
##### 前置条件
   1. CentOS版本大于3.1
       > 查看CentOS版本号：uname -r
   2. 若需升级：
        > 升级软件包及内核:yum update
#### Docker的常见操作 
   1. 安装docker 
        > systemctl install docker
   2. 卸载docker
        > yum remove docker
   2. 启动docker
        > systemctl start docker
   3. 将docker服务设置为开机启动
        > systemctl enable docker
        ****
   4. 检索镜像
       - 默认是在docker hub上检索镜像
        > docker search 关键字  
        eg：docker search redis
   5. 拉取镜像
        > docker pull 镜像名:tag
        - ":tag"是可选的，tag表示标签，多为软件的版本，默认是latest 
   6. 查看本地镜像
        > docker images
   7. 删除本地镜像
        > docker rmi image-id
        ***
   8. 借助镜像，新增一个容器
        > docker run --name container-name -d image-name  
        eg: docker run –name myredis –d redis 
        - "--name"：自定义容器名
        - "-d"：后台运行
        - "image-name":指定镜像模板
   9. 查看运行中的容器
        > docker ps  
        docker ps -a 
        - 加上"-a"可以看到所有容器
   10. 停止运行中的容器
        > docker stop container-name/container-id 
   11. 启动已存在容器
        > docker start container-name/container-id
   12. 删除容器
        > docker rm container-name/container_id
   13. 端口映射(命令参数)
        > -p 80:8080  
          eg：docker run -d -p 80:8080 tomcat
        - 将主机端口映射到容器内部的端口
   14. 容器日志
        > docer logs container-Name/container-id
   15. 进入正在运行的容器目录
        > docker exec -it container-id/container-name /bin/bash
   16. 

    
### 更多命令
https://docs.docker.com/engine/reference/commandline/docker/


#### DockerFile 的写法

##### From ubantu

#####  ADD XXX
- add p1 p2 
- p1 可以是 
- ADD:从源地址（文件，目录或者URL)复制文件到目标路径
- ADD [--chown=<user>:<group>]<src>...<dest>
- ADD[-chown=<user>:<group:>][“<src>”,“<dest>"](路径中有空格时使用)
- ADD支持Go风格的通配符，如ADD check*/testdir,/
- srC如果是文件，则必须包含在编译上下文中，ADD指令无法添加编译上下文之外的文件
- src如果是URL
   - 如果dest结尾没有/，那么dest是目标文件名，如果dest结尾有/，那么dest是目标目录名
- 如果src是一个目录，则所有文件都会被复制至dest
- 如果src是一个本地压缩文件，则在ADD的同时完整解压操作
- 如果dest不存在，则ADD指令会创建目标目录
- 应尽量减少通过ADD URL添加remote文件，建议使用curl或者wget&&untar
- **ADD命令官方推荐只有在解压缩文件并把它们添加到镜像中时才需要。**

##### COPY XX
- COPY:从源地址（文件，目录或者URL)复制文件到目标路径
- COPY [--chown=<user>:<group>]<src>...<dest>
- COPY[-chown=<user>:<group>]["<src>"."<dest>"]/路径中有空格时使用
> COPY的使用与ADD类似，但有如下区别
- COPY只支持本地文件的复制，不支持URL
- COPY不解压文件
- COPY可以用于多阶段编译场景，可以用前一个临时镜像中拷贝文件
- COPY --from=build /bin/project /bin/project
- COPY语义上更直白，复制本地文件时，优先使用COPY


```
add 比copy 多一些功能： 自动把压缩包解压；源文件可以来源于url中，但官方不推荐，因为这样的镜像体积比较大
```



#### CMD
- 指定启动容器时执行的命令，每个 Dockerfile只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。


####  ENTRYPOINT
- ENTRYPOINT:定义可以执行的容器镜像入口命令
- ENTRYPOINT["executable","param1","param.2"]/docker run参数追加模式
- ENTRYPOINT command param1 param2/∥docker run参数替换模式
- docker run-entrypoint可替换Dockerfile中定义的ENTRYPOINT
- ENTRYPOINT的最佳实践是用ENTRYPOINT定义镜像主命令，并通过CMD定义主要参数，
- 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

如下所示
ENTRYPOINT ["s3cmd"]
CMD [--help"]

#### cmd 和endpoint的区别
- 1、CMD指令指定的容器启动时命令可以被docker run指定的命令覆盖,ENTRYPOINT指令指定的命令不能被覆盖，而是将docker run指定的参数当做ENTRYPOINT指定命令的参数。
- 2、CMD与ENTRYPOINT同时存在时，CMD指令可以为ENTRYPOINT指令设置默认参数，而且CMD可以被docker run指定的参数覆盖；

##### Dockerfile最佳实践

- 不要安装无效软件包
,应简化镜像中同时运行的进程数，理想状况下，每个镜像应该只有一个进程。
当无法避免同一镜像运行多进程时，应选择合理的初始化进程(init process)
最小化层级数
。
最新的docker只有RUN，COPY,ADD创建新层，其他指令创建临时层，不会增加镜像大小
·比如EXPOSE指令就不会生成新层
·多条RUN命令可通过连接符连接成一条指令集以减少层数
·通过多段构建减少镜像层数
·把多行参数按字母排序，可以减少可能出现的重复参数，并且提高可读性
。
编写dockerfile的时候，应该把变更频率低的编译指令优先构建以便放在镜像底层以有效利用build cache
复制文件时，每个文件应独立复制，这确保某个文件变更时，只影响改文件对应的缓存




#### 打包  
- 文件名："Dockerfile"
```
docker build -t martinyds/ingress-manager:1.0.0 .
docker push martinyds/ingress-manager:1.0.0


```

### 多过程打包示例：
```dockerfile
FROM golang:1.19 as builder
WORKDIR /app
COPY .  . 
ENV GOPROXY="https://goproxy.io"
RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o gatewayService /app/cmd/


FROM alpine AS runner
WORKDIR /project
COPY --from=builder /app/gatewayService .
COPY --from=builder /app/config/ ./config/
EXPOSE  2000
CMD /project/gatewayService

```