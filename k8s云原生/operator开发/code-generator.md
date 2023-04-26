# code-generator介绍

---

### 如何操作自定义资源

`client-go`为每种K8S内置资源提供对应的`clientset`和`informer`。那如果我们要监听和操作自定义资源对象，应该如何做呢？这里我们有两种方式：

- **方式一：** 使用`client-go`提供的``dynamicClient``来操作自定义资源对象，当然由于``dynamicClient``是基于`RESTClient`实现的，所以我们也可以使用``RESTClient`来达到同样的目的。
- **方式二：** 使用`code-generator`来帮助我们生成我们需要的代码，这样我们就可以像使用`client-go`为K8S内置资源对象提供的方式监听和操作自定义资源了。

---
### code-generator

[code-generator](https://github.com/kubernetes/code-generator)是K8S官方提供的一组代码生成工具，它主要有两个应用场景：

- 为CRD编写自定义controller时，可以使用它来生成我们需要的`versioned client`、`informer`、`lister`以及其他工具方法
- 编写自定义API Server时，可以用它来 `internal` 和 `versioned`类型的转换`defaulters`、`internal` 和 `versioned`的`clients`和`informers`

我们本套课程只会涉及为CRD编写自定义controller的场景。