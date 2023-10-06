[toc]



# Kubernetes的网络服务：



## service：

### 说明：

实际的负载均衡功能由kube-proxy实现

kube-proxy 不能直接操作，需要写一个service对象，用于配置负载均衡 

配置service就是为kube-proxy生成负载均衡配置







kube-proxy支持两种负载均衡模式： 

1. iptables  默认的负载均衡模式 

2. ip_vs  推荐的负载均衡模式 ，在使用kubeadm部署kube-proxy的时候 通过修改配置文件可以修改为ip_vs

kube-proxy是以daemonsets的模式在每个节点都部署了一个



### 类型：

clusterIP：这种类型的service会生成为负载均衡器生成一个cuusterIP地址，该地址只能在集群内部访问到，一般用于集群内部的pod之间的访问

  service域名： 

1. 在创建service的时候，为service指定clusterIP 
2. clusterIP可以为None，此时只能通过域名访问
3. 当clusterIP存在的时候，域名会解析至clusterIP；当clusterIP为None的时候，域名会直接解析至service关联的所有pod 
4. 当clusterIP为None的时候，kube-proxy是不提供代理的

nodePort：

1. nodePort类型的service也会创建clusterIP 
2. nodeoirt类型的service同时会在所有的kube-proxy所在的节点上创建一个基于deployment

loadBalancer： 

externalName：



例：

```yaml
#gvk
apiVersion: v1
kind: Service
#元数据
metadata:
  creationTimestamp: null
  labels:
    app: webserver-sts
  name: webserver-sts
#描述：
spec:
  ports:
  - name: 80-80
    port: 80                 #指定需要映射的端口
    protocol: TCP            #指定协议
    targetPort: 80           #指定带有标签的pod的端口
  #选择带有特定标签的pod   
  selector:
    app: webserver-sts
  type: ClusterIP
```



## ingress：



支持四层转发，可以监听在端口上，一般用于七层转发 



有两部分组成

1. 负责提供负载均衡能力的负载均衡器，被称为ingresscontroller 
2. 负责为ingresscontroller生成配置的资源对象是ingress 

ingresscontroller必备的能力： 

1. 能够解析ingress资源对象生成相应的转发规则
2. 能够实时感知后端pod的变化以更新负载均衡后端地址 

开源的ingresscontroller

1. haproxy-ingress 
2. traefik
3. nginx-ingress  kubernetes官方的 
4. kubernetes-ingress ngxin官方的

针对一个pod 如果想配置一个ingress 必须配置一个service，因为ingress并不负责去发现后端的pod的ip以及pod所监听的端口，他会watch一个service 这个service会发现后端的IP以及pod所监听的端口，只要service发现了更新 ingress就能感知到后端pod的变化 将service所关联的这一组pod和端口挂载到ingress的后端 







负载均衡器