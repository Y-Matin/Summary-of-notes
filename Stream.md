## Sream 知识 
### Stream的特性
    1. 不是数据结构，没有内部存储
    2. 不支持索引访问
    3. 延迟计算
    4. 支持并行
    5. 很容易生成数组或集合（List、Set)
    6. 支持过滤、查找、转换、汇总、聚合等操作
### Stream 运行机制
> - Stream 分为 源source、中间操作、终止操作  
> - 源source：可以是一个数组、一个集合、一个生成器方法、一个I/O通道等等。  
> - 一个流可以有零个或者多个中间操作，每一个中间操作都会返回一个新的流，供下一个操作使用。一个流只会有一个终止操作。
> - Stream只有遇到终止操作，它的源才会开始执行遍历操作
### Steam 常见API
#### 中间操作
> 过滤 : filter  
> 去重 ：distinct  
> 排序 ：sorted
> 截取 ：limit、skip  
> 转换 ：map/flatMap 
> 其他 ：peek(一般用于debug、打印日志等)
#### 终止操作
> 循环 : forEach  
> 计算 ：min、max、count、average  
> 匹配 ：anyMatch、allMatch、moneMatch、findMatch、findFirst、findAny
> 汇聚 ：reduce 
> 收集器 ：toArray、collect 

### 总结

stream和collectors对象中都有排序，比较大小、取最大最小值的函数。