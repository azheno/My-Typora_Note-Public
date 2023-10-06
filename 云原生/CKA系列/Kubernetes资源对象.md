[toc]



# Kuberbetes资源对象



## Global类型对象：



### namespaces

### nodes







## namespace类型对象：



### Pods



#### 说明：

- kubernetes中的最小资源单位是pods 
- 一个pods当众可以包含多个container
- kubernetes不直接管理容器，哪怕一个pods中只有一个容器，kubernetes的管理对象也是pods
- 对于kubernetes来讲，一个pods最少包含两个容器，一个是我们运行的容器，另一个成为pause容器，该容器非常小，用于接管pods的网络，也就是一个pods多个容器共享一个网络地址 
- 一个pods永远只有一个IP，所有的流量都是由pause来进行劫持，每个pods中的容器的端口不同，通过端口来进行区分访问的容器 



#### 操作：

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



#### yaml文件来创建pods

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

#### 创建一个资源对象的步骤

##### group version kind

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



##### spec

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
  IfNotPresent #默认策略
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

13. 容器的持久存储

```yaml
#和containers平级,将宿主机的目录或文件挂载到容器的路径中
##hoatPath的用法，但是不能解决pod重新调度的问题，会导致目录不一致，hostPath应用的方法多数在于每个机器都拥有的自带的配置文件。
volumes: 
  - name: rootdir 
    hostPath: 
      path: /data/nginx 
  - name: localtime 
    hostPath: 
      path: /etc/localtime 
containers: 
  - name: nginx 
    image: nginx:1.21 
    volumeMount: 
      - name: rootdir 
        mountPath: /usr/shar/nginx/html 
      - name: loclatime 
        mountPath: /etc/localtime
        readOnly: true #允许只读挂载
##emptyDir临时目录 在宿主机中创建一个随机目录 挂载到容器中，生命周期和容器相同，emptyDir不是持久存储，无法保证数据的持久化
#用处：多容器之间共享临时数据、emptydir的目录可以是宿主机的内存
volumes:
  - name: rootdir 
    emptyDir:       #用法和hostPath一样，但是不需要写Path
  - name: rootdir 
    emptyDir: 
      medium: Memory 
      sizeLimit: 32M    #将数据挂载到内存中，容器死亡，数据随着一同删除
```

14. 初始化容器

```yaml
#初始化容器用于帮助主容器初始化，会先于主容器启动，然后执行相关任务，任务不是长时任务，只有当初始化容器执行任务完毕，并且正常退出之后，主容器才会启动，初始化容器可以有多个，中文写初始化容器是顺序执行的，任意一个初始话容器执行失败，则整个pod退出，只有初始化容器正常运行并且正常退出，长时容器才会正常运行
initcontainers: 
  - name: busybox
    image: busybox:1.28 
    volumeMounts: 
      - name: rootdir 
        mountPath:  /data 
     command: 
       - /bin/bash 
       - -c 
       - "echo 'hello word' > /data/index.html" 
#和containers平级存放，本质上只有一个容器，初始化完成之后退出
```

15. 容器的网络模式

```yaml 
#直接使用宿主机的IP地址，不会通过k8s的网络，containers的子级
hostNetwork: true 
```



16. 容器的资源限制

```yaml 
#containers的子级: 一次指定一个containers
resources: 
  requests: 
    #
    cup: 0.1
    memory: 32Mi
  limits:
    # CPU限流，分配的是核心数，但是实际上还是会按照权重，k8s中，当所有的节点都空闲的时候，CPU还是会限制CPU的流量
    cup: 0.3
    memory: 64Mi
```



17. 静态pod

控制面的pod是静态pod，只会运行在当前节点上 没有能力自己调度自己 



18. 容器的健康检查

- 探活类型

  ​	(1) 基于端口

  ​		telnet podIP 8080 

  ​	(2) 基于http

  ​		curl http://podIP/healthz statusCode < 400  

  ​	(3) 基于命令

  ​		/healthcheck.sh returnCode 0 



探活参数



- 探活的时机

  startupProbe ：启动阶段探活，探活失败是预期的状态；如果探活成功，则startipProbe退出；如果超出失败的最大次数 ，容器被杀掉 

```yaml
startupProbe: 
  tcpSocket:
    port: 80 
  failureThreshold: 30 
  seccessThreshold: 1 
  periodSeconds: 10 
  timeoutSeconds: 1 
```

livenessProbe：运行周期内探活，预期成功；如果探活失败，会杀死容器，再startupprobe探活成功退出之后，livenessprobe才会开始启动

```yaml
livenessProbe: 
  tcpSocket:
    port: 80 
  failureThreshold: 3 
  seccessThreshold: 1 
  periodSeconds: 10 
  timeoutSeconds: 1 
```



readinessProbe：运行周期内探活，预期成功；如果探活失败，不会杀死容器，将容器标记为未就绪，在startupprobe探活成功退出之后，readinessprobe才会开始启动

```yaml
readinessProbe: 
  tcpSocket:
    port: 80 
  failureThreshold: 3 
  seccessThreshold: 1 
  periodSeconds: 1 
  timeoutSeconds: 1 
```



### Deployments



#### 说明

生产环境最常用的控制器，高级资源对象，用来控制pod ，以及pod的变更，用来接管pod

支持多种更新方式 来实现对pod的更新 

支持多副本 

监视pod的

pod资源对象面临的问题： 

1. 没法修改
2. 指定副本数



#### 用法：

创建一个deployments的yaml文件

```yaml
#gvk
apiVersion: apps/v1
kind: Deployment

#元数据
metadata:
  creationTimestamp: null
  labels:
    app: webserver
  name: webserver
  
#spec 
spec:
  #指定pod的副本数
  replicas: 5
  #监视pod的副本数
  selector:
    matchLabels:
      app: webserver
      
  #pod的更新策略 两种方式更新：
  #recreate：把该deployment创建出来的pod全部删掉，再全部重建
  #rollingupdata：滚动更新
    #maxUnavailable：指定该滚动更新最大不可用数，
    #maxSurge：指定滚动更新时最大可用数量，使用百分比进行指定，会四舍五入，当和maxSurge和maxUnavailable同时存在的时候，        maxSurge的优先级会最高，滚动更新策略是先新建百分比的pod，然后删除百分比的pod，持续完成，直至更新结束  
如果删除了pod 监视器会匹配副本数并且会创建出来，不会在意在那个node上，只会在意是否创建出来 

  strategy: 
    type: RollingUpdate 
    rollingUpdate: 
      maxUnavailable: 25% 
      maxSurge: 25% 
      
  #pod的描述，之中写的是pod的语法
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webserver
    spec:
      containers:
      - image: nginx:1.21
        name: nginx
        resources: {}
```



### Statefulests

有状态集

1. 支持多副本
2. 不支持更新策略
3. 保证创建副本的名称保持不变
4. 不能定义滚动更新策略

需要定义serviceName，指定副本统一名称

```yaml
#gvk
apiVersion: apps/v1
kind: Statefulest
#元数据
meatdata: 
  .....
spce
  replicas: 5
  #监视pod的副本数
  selector:
    matchLabels:
      app: webserver
  serviceName: webserver-sts
  #pod的信息
  template: 
    meatdata: 
      ......
    containers: 
      - name: nginx
        image: nginx:1.21
        .........
```



### daemonsets

```yaml
#使用daemonstes资源对象启动的pod在每个kubernetes集群的节点都会运行一个，且只会运行一个，通常用于agent场景如： 
#  Prometheus的exporter 
#  zabbix的agent 
spec:
  selector:
    matchLabels:
      app: webserver
  template: 
  .........
```



### jobs

运行一次性任务，任务执行完成，容器自动退出

```yaml
apiVersion: batch/v1 
kind: Job 
meatdata: 
  name: helloword
  namespace: default 
spec:
  activeDeadlineSeconds: 10
  backoffLimit: 3 
  selector:
  matchLabels:
      app: webserver
  template:
    metadata: 
      name: helloword
    spec: 
      restartPolicy: Never 
      containers: 
        - name: helloword
          image: busy:1.28 
          command: 
          - /bin/sh 
          - -c 
          - "echo 'helloword'"
  .........
```



### cronjobs

```yaml
apiVersion: batch/v1 
kind: Job 
meatdata: 
  name: helloword
  namespace: default 
spec: 
  schedule: "* * * * *"
  jobTemplate: 
    metadata: 
      name: hellowrod
    spec:
      activeDeadlineSeconds: 10
      backoffLimit: 3 
      selector:
      matchLabels:
          app: webserver
      template:
        metadata: 
          name: helloword
        spec: 
          restartPolicy: Never 
          containers: 
            - name: helloword
              image: busy:1.28 
              command: 
              - /bin/sh 
              - -c 
              - "echo 'helloword'"
```





## 附加在资源对象之上的资源对象：



### Label

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
