## kubebuilder

```cmd
<!-- 主要流程 -->
 kubebuilder init
 kubebuilder  create api --group ingress  --version v1 --kind App     
 kubebuilder create webhook --group ingress --version v1 --kind App --defaulting --programmatic-validation  --conversion
 填充 app_type.go的App的spect 结构体
 实现app_controller.go的 Reconcile逻辑， 对比spec 和当前status的差异，不同就进行调整
 make manifest // 生成相关的 crd的yml文件
 make install   // 声明一个crd ，并创建一个 实例
 make run  // 直接运行发布一个controller，  或则  kubectl apply -f  deploy.yml 来部署controller
```
 - 全局标记

``` 
<!-- 格式如下： -->
//+kubebuilder:object:root=true
//+kubebuilder:subresource:status
```



##### 坑点
- 1 ： kubebuilder的版本和golang的版本不匹配，可以看kubebuilder的go mod查看
- 2 kubebuilder的init失败，go get 失败，报错 not file or dir， 或则 cant open /home/zic
  - 可以看下 go env的环境变量，是否是路径配置问题
  - ```golang
  $ go env |grep zic
    GOMODCACHE="/home/zic/gopath/pkg/mod"
    GOPATH="/home/zic/gopath"
  ```

- kubebuilder create api command results in "cgo_linux.go:12:8: no such package located"
  - gcc 的问题 