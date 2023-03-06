###

- 使用 函数名后加上`[T XX]` XX为T的类型约束，等同于java中的基类
```
func compare[T constraints.Ordered](a T, b T) bool {
	return a > b
}
```

- 泛型的 声明
- 1. ~表示底层类型是string类型的别名或引用，也属于ShowName这个泛型类型
- 2. 表示ShowName包含这几种指定类型
  ```
    type ShowName interface {
	~string | int | float32
    }

  ```****
