<!-- ## Kubernet系列（5）常用命令 -->


- 创建Pod
  > kubectl apply -f {PodName}.yaml

- 删除pod
  > kubectl delete pod {podName}

- 查看某个命名空间下的所有pod
   > kubectl get pod -n XX

- 查看 更多 
  > 在其他命令的后面加上："-0 wide"
 
- 查看 某一个具体pod 中的详细描述信息
   > kubectl describe pod XX




- 查看指定容器中的日志
   > kubectl log {podName} -c {containerName}