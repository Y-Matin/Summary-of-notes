### Go微服务容器化 
####  配置 go-micro环境
1. 安装micro install  
``` go get -u github.com/micro/micro ```




编译proto命令:
> protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=. path/to/greeter.proto