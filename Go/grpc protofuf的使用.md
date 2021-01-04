<!-- ## Grpc protobuf 的使用 -->

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
字段格式：如 order_id，orderId在翻译为 go语言中 会自动 转换为驼峰格式： **OrderId**

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

