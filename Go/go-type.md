#### type

### type 类型别名 
> type  str  = string
> type rune = int32
> type byte = uint8
- 相当于 str 和 string 是同一个类型， 有两个名字，可以互相赋值

### type 类型定义 

> type IntegerType int
- 基于int 创建了一新类型 ，IntegerType， 两个是不同的类型，相互赋值时，需要显式强转
