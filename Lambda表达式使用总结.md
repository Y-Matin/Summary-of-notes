## Lambda 表达式
### Lambda表达式特点：
1. 函数式编程
2. 参数类型自动判断
3. 代码量少、简洁
### 学好Lambda表达式前提：
1. 熟悉泛型
2. 多练、多用Stream API
### Lambda表达式使用场景
- 任何有函数式接口的地方
### 什么是函数式接口?
- 只有一个抽象方法的接口 <Object 类中的方法除外>
- @FunctionInterfa 注解专门修饰函数式接口
### 常用的基本函数式接口
以下接口来自java8API中的java.util.function包下
- Supplier：代表一个输出
- Consumer：代表一个输入
- BiConsumer: 代表两个输入
- Funtion：代表一个输入，一个输出<输入与输出没有要求>
- UnaryOperator: 代表一个输入，一个输出<输入与输出是相同类型>
- BiFunction: 代表两个输入，一个输出<输入与输出类型没有要求>
- BinaryOperator：代表两个输入，一个输出<输入和输出是相同类型>
### Lambda表达式 是对象，是一个函数式接口的实例
### Lambda 表达式语法
- LambdaParameters -> LambdaBody  
例如：Runnable r = () -> System.out.println("hello!");
> #### 参数
- &#160; &#160; &#160; &#160;()里面的参数的个数 是根据函数式接口中抽象方法的参数个数来决定。当且仅当只有一个参数时，()可以省略。
> #### 返回值
- LambdaBody格式为：
```
{
    express1;
    exlpress2;
    ruturn result;
}
```
- 当只有一条语句时，{}和return以及末尾的";"可以省略。
> #### 常见示例
- () -> {} 无参，无返回值
- () -> {System.out.println(1);} 无参，无返回值
- () -> {return 100;} 无参，有返回值
- () -> 100  简写
- () -> null 无参，有返回值
- (int x) -> {return x+1;} 一个参数，有返回值
- x -> x+1 上面的简写
### 注意事项
1. 参数类型要么全写，要么都不写省略掉
2. 参数不能用final修饰符修饰
3. 不能使用Object类型来接口函数式接口的结果
4. 不需要也不需要使用throws语句来声明它可能会抛出的异常