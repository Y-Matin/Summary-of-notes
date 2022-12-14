### 使用docker 部署 golang程序

#### 方案 
1. 基于源码 编译二进制，运行  ：缺陷比较明显, 步骤较多，打出来的镜像比较大
2. 基于二进制，运行  ： 缺点是有前置要求，提前准备好二进制；优点是 打包逻辑简单，镜像体积很小。
> 一般采用第二种，基于二进制来构建镜像

#### 示例：
> 前置依赖： golang环境+docker环境+要部署的项目
- 以linux 系统为例：
 
    go build
    ```
    2. 进入main目录执行后，得到 linux下的二进制文件
    3. 编写DockerFile文件
    ```
    # 前置条件 ：1.已编译好的二进制 2. go环境
    # docker build -f DockerFile . -t ws_client_image
    From alpine:3.12  # 基于该系统
    WorkDir /opt      # 设置工作目录
    Add ./wsClient /opt  #将 二进制文件 复制到容器的工作目录下

    CMD ["/opt/wsClient"]  # 运行 该二进制
    ```
    4. 执行docker的打包命令；得到镜像文件
    > docker build -f DockerFile . -t ws_client_image
    5. 使用doker的界面或者命令行运行镜像得到容器
    > docker run –name wsClient  -p 9090:9090 -d  ws_client_image

- 注意点：容器间请求ip不能是127.0.0.1，要为物理机的ip