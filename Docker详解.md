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

