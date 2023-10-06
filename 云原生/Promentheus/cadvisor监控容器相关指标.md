# cadvisor简介

CAdvisor是Google开源的一款用于展示和分析容器运行状态的可视化工具。通过在主机上运行CAdvisor用户可以轻松的获取到当前主机上容器的运行统计信息，并以图表的形式向用户展示。

# cadvisor基本安装

直接以docker的形式启动cadvisor：
```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```


访问： http://localhost:8080

> 在kubernetes中，cadvisor默认集成在kubelet中，metrics访问地址： http://ip:10255/metrics/cadvisor ， 这个地址只暴露指标，没有webui。另外 10250是https端口，10255是http端口

# cadvisor常用指标说明

cadvisor输出的一些常用指标：
<div style="width: 200pt">指标名称</div>|<div style="width:40pt">类型</div>|含义
---|---|---
container_cpu_load_average_10s|gauge|过去10秒容器CPU的平均负载
container_cpu_usage_seconds_total|counter|容器在每个CPU内核上的累积占用时间 (单位：秒)
container_cpu_system_seconds_total|counter|System CPU累积占用时间（单位：秒）
container_cpu_user_seconds_total|counter|User CPU累积占用时间（单位：秒）
container_fs_usage_bytes|gauge|容器中文件系统的使用量(单位：字节)
container_fs_limit_bytes|gauge|容器可以使用的文件系统总量(单位：字节)
container_fs_reads_bytes_total|counter|容器累积读取数据的总量(单位：字节)
container_fs_writes_bytes_total|counter|容器累积写入数据的总量(单位：字节)
container_memory_max_usage_bytes|gauge|容器的最大内存使用量（单位：字节）
container_memory_usage_bytes|gauge|容器当前的内存使用量（单位：字节
container_spec_memory_limit_bytes|gauge|容器的内存使用量限制
machine_memory_bytes|gauge|当前主机的内存总量
container_network_receive_bytes_total|counter|容器网络累积接收数据总量（单位：字节）
container_network_transmit_bytes_total|counter|容器网络累积传输数据总量（单位：字节）


# cadvisor常用监控查询语法

## cpu查询语法

```
sum(irate(container_cpu_usage_seconds_total{pod=~'bd-ocean-getui-callback.*',image!=""}[1m])
```



cpu的配额：
```
container_spec_cpu_quota{pod=~'bd-ocean-getui-callback.*',image!=""} /100000
```
或者：

```
kube_pod_container_resource_limits_cpu_cores{image!=""}
```

计算pod cpu使用率： 

```
sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) by (pod) / sum(kube_pod_container_resource_limits_cpu_cores{}) by (pod)
```

或者： 

```
sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) by (pod) / sum(container_spec_cpu_quota{image!=""} /100000) by (pod)
```

## mem查询语法

pod内存使用率：

```
sum(container_memory_working_set_bytes{image!=""}) by(pod) / sum(container_spec_memory_limit_bytes{image!=""}) by(pod) * 100 
```

> 如果没有设置container_spec_memory_limit_bytes的限制，会出现+Inf的结果


关于内存的几个指标： 

<div style="width: 185pt">名称</div> | <div style="width: 40pt">类型</div>| <div style="width: 30pt">单位</div>| 说明
---|---|---|---
container_memory_rss|	gauge	|bytes |RSS内存，即常驻内存集（Resident Set Size），是分配给进程使用实际物理内存，而不是磁盘上缓存的虚拟内存。RSS内存包括所有分配的栈内存和堆内存，以及加载到物理内存中的共享库占用的内存空间，但不包括进入交换分区的内存。
container_memory_usage_bytes|	gauge	|bytes|container_memory_rss + container_memory_cache + container_memory_swap
container_memory_max_usage_bytes|	gauge	|bytes|最大内存使用量的记录。
container_memory_cache|	gauge	|bytes|高速缓存（cache）的使用量。cache是位于CPU与主内存间的一种容量较小但速度很高的存储器，是为了提高cpu和内存之间的数据交换速度而设计的。
container_memory_swap	|gauge|bytes|虚拟内存使用量。虚拟内存（swap）指的是用磁盘来模拟内存使用。当物理内存快要使用完或者达到一定比例，就可以把部分不用的内存数据交换到硬盘保存，需要使用时再调入物理内存
container_memory_working_set_bytes	|gauge	|bytes|当前内存工作集（working set）使用量。
container_memory_failcnt	|counter	|次数|申请内存失败次数计数
container_memory_failures_total	|counter|	次数|累计的内存申请错误次数


另外，关于`container_memory_usage_bytes`和`container_memory_working_set_bytes`这两个指标，这篇文章讲的非常好： https://medium.com/faun/how-much-is-too-much-the-linux-oomkiller-and-used-memory-d32186f29c9d

## 文件系统


查找文件系统使用前10的pod
```
topk(10,sum by (pod) (container_fs_usage_bytes{image!=""} /1024 /1024))
```

查询文件系统读取速率：

```
sum(rate(container_fs_reads_bytes_total{image!=""}[1m])) without (device)

```

查询文件系统写入速率： 

```
sum(rate(container_fs_writes_bytes_total{image!=""}[1m])) without (device)

```

## 网络

接收网络包速率：

```
sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)

```

传输网络包速率： 

```
sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)

```


# 附录

参考： [《容器监控：Cadvisor》](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/exporter/commonly-eporter-usage/use-prometheus-monitor-container)