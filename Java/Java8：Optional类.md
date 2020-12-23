### Java8:Optional类详解
#### 常见方法：
```
/**
 * Optional<T> 类(java.util.Optional) 是一个容器类，代表一个值存在或不存在，原来用null 表示一个值不存在，
 * 现在Optional 可以更好的表达这个概念。并且可以避免空指针异常。
 * 常用方法：
 * Optional.of(T t) : 创建一个Optional 实例
 * Optional.empty() : 创建一个空的Optional 实例
 * Optional.ofNullable(T t):若t 不为null,创建Optional 实例,否则创建空实例
 * isPresent() : 判断是否包含值orElse(T t) :  如果调用对象包含值，返回该值，否则返回empty
 * orElse(T t) ：如果调用对象包含值，返回该值，否则返回 t
 * torElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回s(Lambda) 获取的值
 * map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 empty
 * flatMap(Function mapper):与map 类似，要求返回值必须是Optional
 */
```
***
```
@Test
    public void createOption() {
        //1.of()中的入参不能为null
        Optional<Book> optionalBook = Optional.of(new Book());
        Book book = optionalBook.get();
        System.out.println(book );
        //2.empty()返回的是一个空的Optional实例。
        Optional<Object> empty = Optional.empty();
        //3.ofNullable(T t):若t 不为null,创建Optional 实例,否则创建空实例<为1,2的结合>
        Optional<Object> optionalEmpty = Optional.ofNullable(null);
        Optional<Book> optionalBook1 = Optional.ofNullable(new Book());
    }

    @Test
    public void getValue() {
        //1. 使用 isPresent() 判断Optional中是否包含值，
        Optional<Book> optional = Optional.ofNullable(null);
        if (optional.isPresent()) {
            System.out.println(optional.get());
        }
        //2. 使用 orElse()方法通过指定一个默认返回值来避免返回null
        Object orElse = optional.orElse(new Book("spring 指南", "yds"));
        System.out.println(orElse);
        //3. 使用 orElseGet(Supplier<? extends T> other)方法：如果没有值，就通过一些处理（Lambda）扩展生成一个目标值
        Object orElseGet = optional.orElseGet(() -> new Book("spring boot 2.0核心思想指导", "小马哥"));
        System.out.println(orElseGet);
        //4. 使用map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 empty
//        Optional<Book> optionalBook = Optional.of(new Book("颈椎康复指南", "苗医生"));
        Optional<Book> optionalBook = Optional.empty();
        Optional<String> optionalMap = optionalBook.map(Book::getName);
        System.out.println(optionalMap);
        //5. 使用flatMap(Function<? super T, Optional<U>> mapper) ，与map()类似，只是对返回值要求是Optional类型
        Optional<String> optionalS = optionalBook.flatMap(book -> Optional.of(book.getName()));
        System.out.println(optionalS);

    }
```