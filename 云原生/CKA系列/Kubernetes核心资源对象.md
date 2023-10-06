[toc]





# Kuberbetes核心资源对象



## namespaces







## Pods



### 说明：

- kubernetes中的最小资源单位是pods 
- 一个pods当众可以包含多个container
- kubernetes不直接管理容器，哪怕一个pods中只有一个容器，kubernetes的管理对象也是pods
- 对于kubernetes来讲，一个pods最少包含两个容器，一个是我们运行的容器，另一个成为pause容器，该容器非常小，用于接管pods的网络，也就是一个pods多个容器共享一个网络地址 
- 一个pods永远只有一个IP，所有的流量都是由pause来进行劫持，每个pods中的容器的端口不同，通过端口来进行区分访问的容器 



### 操作：

增：

```shell
#通过单机节点创建
docker run -d -p 80:80 nginx:1.21
#在集群中创建
kubectl run webserver --image=nginx:1.21 -n k8s # -n 指定命名空间 
#集群创建的时候不会指定ID 必须指定一个名称 每个namespace每一个pod的名称必须唯一 
```



删：

```shell
kubectl delete pods webserver 
```



改：

```shell
kubectl edit pods webserver 
```



查：

```shell
kubectl describe pods webserver 
```



列出：

```shell
kubectl get pods -A 
kubectl get pods -n kube-system 
kubectl get pods -n kube-system -o wide 
```



进入pod

```shell
kubectl exec -it webserver -- /bin/bash 
#当一个pod里面有两个容器，默认不指定的时候会进入第一个容器 
kubectl exec -it webserver -c nginx -- /bin/bash 
#可以通过-c来指定一个容器的进入
```



查看容器的日志： 

```shell
#只有一个容器的情况
kubectl logs webserver 
#有两个容器的情况
kubectl logs webserver nginx
```



### yaml文件来创建pods

```yaml
##查看语法的命令： 
kubectl explain pods --recursive 
##可以查看文件语法结构

[root@node-gksz01040822-1918073-o966n ~]#kubectl run  webserver --image=nginx:1.21 -o yaml --dry-run=client 
## gvk ==>> group version kind 
apiVersion: v1
kind: Pod
#使用kubectl api-resources可以查看kind和apiversion的版本 

##元数据信息
metadata:
  creationTimestamp: null
  labels:
    run: webserver
  name: webserver
##metadata四大信息： name(资源对象的名字) namespace(资源对象的命名空间，如果不指定，默认是default) labels(标签) annotations(注解) 
##用来寻找和判断 用于标识资源对象的用途

##具体的资源对象的创建的信息，也就是怎么创建资源对象
spec:
  containers:
  - image: nginx:1.21
    name: webserver
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

### 创建一个资源对象的步骤

#### group version kind

1. 指定名称

1. 所属namespace
2. 标签
3. 注解

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: webserver 
  namespace: pod
  lables: 
    type: dell 
  annotations: "this is a dell webserver" 
```



#### spec

4. 容器名称

```yaml
#containers的子级
name: webserver01 
```

5. 容器使用的镜像

```yaml
#containers的子级
image: nginx:1.21
```



6. 容器所暴露的端口

```yaml
#containers的子级,可以定义多个port： 
ports: 
  - name: http 
    protocol: TCP 
    containerPort: 80 
    hostPort: 80 
  - name: nginx
    protocol: TCP 
    containerPort: 8080 
```

7. 容器拉取镜像的策略

```yaml
#containers的子级
imagePullpolicy: 
  Alaways      #总是拉取
  IfNotpresent #默认策略
  Never        #从不拉取
```



8. 容器启动时所执行的指令

```yaml
#command关键字,containers的子级：
conmmand: 
  - /bin/bash 
  - -c 
  - "nginx -g 'demon off ;' "     #这种写法不会报错 

#args关键字： 
args: 
  - -g 
  - "demon off;"   #直接将参数传递给镜像中的ENTRYPOINT脚本，镜像支持从args中传参，也可以给command传参
```



9. 容器的重启策略

```yaml
#和containers平级
restartPolicy: 
  Always    #总是重启
  OnFailure #当pod异常退出时则自动重启
  Never     #不要重启
```



10. 容器的DNS配置

```yaml
#和containers平级
dnsPolicy: 
  Dafault                       #配置pod内的DNS是其宿主机的DNS
  ClusterFirst                  #配置pod内的DNS 让其指向集群内的coredns
  None                          #如果不管，则是默认值，默认是集群内部的DNS地址
  DefautWithHostNetwork
  ClusterFirstWithHostNetwork
```



11. 容器中的环境变量

```yaml
#containers的子级
env: 
- name: MYASQL_HOST
  value: "192.168.0.1"
- name: MYSQL_PASSWD 
  value: "000000"
```



12. 一个pods当中运行多个容器

```yaml
#containers的子级 
可以在一个containers下定义多个容器，每个容器定义的方法都是一样的，只是添加了一层 
containers: 
  - name: webserver1 
    image: nginx:1.21 
    env: 
      - name: "test1"
        value: "111" 
    restartPolice: Always 
  - name: webserver2 
    image: nginx:1.21 
    env: 
      - name: "test2"
        value: "222"
    restartPolice: Never 
```



1. 容器的持久存储
2. 容器的资源限制
3. 初始化容器
4. 容器的健康检查
5. 静态容器
6. 容器的网络模式



## Label

标签是一个特殊的资源对象 

一般存在于一个父级资源对象之下 

增

```shell
kubectl label node k8s01 type=dell 
kubectl label pods webserver env=prod 
```



删

```shell
kubectl label node k8s01 type- 
```



改

```shell
kubectl label node k8s01 type=control-plane --overwrite #需要指定一个--overwrite说明是修改标签
```



查

```shell
#查看指定namespace中有app=front标签的pod 
kubectl get pods -n default -l app=front 

#查看pod中的所有的标签值
kubectl get pods --show-labels
```



列出

```shell

```

