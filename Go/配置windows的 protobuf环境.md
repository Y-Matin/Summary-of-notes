## 配置windows的 protobuf环境

1. 下载 protoc 二进制 以及 protoc-gen-go 代码执行go build 得到 protoc-gen-go二进制
2. 将以上两个二进制放到path目录下。
3. 编辑proto文件
4. 在文件目录下，执行： protoc --go_out=./ *.proto
