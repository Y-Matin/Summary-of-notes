<!-- ## Kubernet系列（5）常用命令 -->


- 创建Pod
  > kubectl apply -f {PodName}.yaml

  > kubectl create -f {PodName}.yaml
  

- 删除pod
  > kubectl delete pod {podName}

- 查看某个命名空间下的所有pod
   > kubectl get pod -n XX

- 查看 更多 
  > 在其他命令的后面加上："-0 wide"
 
- 查看 某一个具体pod 中的详细描述信息
   > kubectl describe pod XX
- 查看 指定pod的日志
  > kubectl logs {podName}
  > kubectl logs -f  {podName}   
  *类似于 tail -f 的方式查看(tail -f 实时查看日志文件 tail -f 日志文件log)*
- 查看指定pod中具体容器的日志
  > kubectl logs {podName} -c {containerNmme}
- 进入Pod文件系统
  > kubectl exec -it {podName} -c {containerName} -- sh


```
martin@yds-server:~$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
service/hello-minikube exposed
```
