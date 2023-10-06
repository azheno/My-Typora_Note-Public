# CKA认证题库



## 1. RBCA

1.   创建一个名为deployment-clusterrole且仅允许创建以下资源类型的新ClusterRole：

*   Deployment

*   StatefulSet

*   DaemonSet

2.   在现有的 namespace app-team1中创建一个名为cicd-token的新 ServiceAccount。

3.   限于 namespace app-team1中，将新的ClusterRole deployment-clusterrole绑定到新的 ServiceAccount cicd-token。

```shell
kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployments,statefulsets,daemonsets
kubectl -n app-team1 create serviceaccount cicd-token
kubectl -n app-team1 create rolebinding cicd-token-rolebinding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token
```



## 2. 节点设置不可用

将名为`ek8s-node-1`的node设置为不可用，并重新调度该node上所有运行的pods

```shell
kubectl config use-context ek8s
kubectl drain ek8s-node-1 --ignore-daemonsets # 设置不可调度并驱逐节点上 Pod
```

>   注：如果执行 drain 提示错误，根据提示再加上选项，例如`--delete-local-data --force`





