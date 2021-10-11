### 命名空间
1. kubernetes初始4个命名空间：
+ default: 默认命名空间
+ kube-system: 由kubernetes系统创建对象的命名空间
+ kube-public: 所有用户都可以读取它。提供集群使用，以防止某些资源在整个集群中应该是可见和可读的。该命名空间只是一种约定，而不是要求。
+ kube-node-lease: 该命名空间存放各个节点续期相关的对象，能够满足集群规模不断扩大时对节点心跳检测性能的要求。

> 避免使用kube-创建命名空间，因为它是kubernetes系统保留的命名空间

2. 创建
+ 基于.yaml文件创建(my-namespace.yaml)
```
apiVersion: v1
kind: Namespace
metadata:
  name: <name-of-namespace>
```
```
kubectl create -f ./my-namespace.yaml
```
+ 命令行创建
```
kubectl create namespace \<name-of-namespace\>
```
3. 删除
```
kubectl delete namespaces <name>
```
> 删除该命名空间下的所有内容
删除是异步的，所以有一段时间该命名空间处于Terminating状态
4. 查看
+ 查看集群中所有的命名空间
```
kubectl get namespaces
```
+ 查看指定命名空间的摘要
```
kubectl get namespaces <name>
```
+ 查看指定命名空间的详细信息
```
kubectl describe namespaces <name>
```

