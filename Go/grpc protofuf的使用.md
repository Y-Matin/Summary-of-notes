<!-- ## Grpc protobuf 的使用 -->

- 核心思路：
> 编译自己自定义的proto文件，得到对应编程语言的版本，将这版本分别放到server端和client端  
> server端利用该编译版本，得到server，将实现了rpc方法的结构体注册到该server中，负责处理client端的grpc请求  
> client端利用该编译版本，得到client，使用client 构造参数，调用rpc方法。
- grpc-gateway，主要是为了提供http形式的rpc调用。

###  简单示例
```proto
// 指定protobuf的版本，proto3是最新的语法版本
syntax = "proto3";
package service;
import "google/api/annotations.proto";

message ProdRequest{
  int32 prod_id = 1;  // 传入的商品id
}

message ProdResponse{
  int32 prod_stock = 1;// 商品库存
}

service ProdService{
  rpc GetProdStock (ProdRequest) returns (ProdResponse);

}
```

- message 定义一个结构。等同于go中的struct，java中的class
> 声明字段时，必须标注该字段的序号
- rpc  定义了一个远程调用的方法，包含入参，出参信息
- service 定义了一个包含**rpc** 远程调用方法 的接口 。 由服务端去实现接口中的方法，客户端调用服务端实现的方法，得到返回值。

### protobuf 关键词、语法 
#### repeated 表示多个（数组）
- 修饰词， 可重复的，可以等同于 go 中的切片， java中的list 。
- 示例：
```java
message ProdList{
//  repeated ：重复的， 表示多个，可等同于go中的silence或java中的list
  repeated ProdResponse prods=1;
}
```
#### enum 枚举
- 常量  
- 示例：
```java
enum ProdAreas{
  A=0;  // 默认值。
  B=1;
  C=2;
}
```
#### 导入外部proto文件 import 
- import " XX.proto "

#### 日期格式


###  编译命令

- 编译proto文件生成 rpc接口
> protoc --go_out=pluins=grpc:../service Prod.proto

- 既支持rpc也支持http 
> protoc --grpc-gateway_out=logtosderr=true:../services Order.proto



### 流模式
- 场景：
  > 在批量查询时：  
  传统方式：一边要等待client端组装请求数据，然后在发送给server端，在等server端查询数据并组装返回数据，在一并返回。   
  缺点：  
  当 并发高，请求数据量大时，耗时增加，响应变慢

- 流的分类
  1. 服务端流 《服务端耗时占比较大》
    - 客户端一次性把要查询的客户列表发送给server端，server端分批次返回到流中，client端从流中读取，可以一边读取数据，一边处理数据，知道流中没有数据
  2. 客户端流 《客户端耗时占比较大》
    - 
  3. 双向流