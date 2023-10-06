# Kubernetes集群基本操作：



Linux的哲学：一切皆文件 

kubernetes：一切皆对象 

## 命令关键字：



kubectl 



## Kubernets中包含的对象：



* nodes  节点对象
* pods 
* namespaces 用来存储对象的对象，用于做对象隔离
* deployments 
* statefulsets 
* jobs 
* cronjobs 
* services 
* ingresses 
* persistentvolume 
* persistentvolumeclaim 
* role 
* clusterrole 
* rolebinding 
* clusterrolebinding
* limitrange 
* resourcequota
* networkkpolicy 
* configmaps 
* secrets 
* ..........



### 对象分为两类： 

- 全局资源对象： 这种对象全局可见，不能被namespace所隔离 
- namespaced资源对象：这种对象必须位于同一个namespaced，不同namespaced中的资源对象不可见 
- kubectl api-resources 查看所有的资源对象 
- namespaced是针对集群来讲的，不关心在哪个node上，k8s对node的概念很弱 

```shell
[root@kubernetes-master ~]# kubectl  api-resources 
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
```

NAME：对象名称

SHORTNAMES：对象段名称 

APIVERSION：API版本 

NAMESPACED：是否是namespaced资源对象，如果是则创建的时候 需要指定一个namespaced





## 针对对象的操作：

- 增

  - create
  - apply 


```shell
kubectl create ns k8s 
```



- 删

  - delete 


```shell
kubectl delete ns k8s 
```



- 改

  - edit 
- patch 
  - apply 


```shell
kubectl edit ns k8s 
```



- 查

  - describe 


```shell
kubectl describe ns k8s 
```



- 列出

  - get 

```shell
kubectl get ns 
```



## 命令参数：

-n 

-o 

--show-labels 

-A

-o yaml 

-l key=value 


