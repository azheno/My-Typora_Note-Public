[toc]

# 简介

process-exporter是一个第三方的exporter，是对node_exporter的有力补充，可以监控指定进程的详细信息。

其代码托管地址： https://github.com/ncabatoff/process-exporter


# 部署

## 1. 下载安装包

```
wget https://github.com/ncabatoff/process-exporter/releases/download/v0.6.0/process-exporter-0.6.0.linux-amd64.tar.gz

tar xf process-exporter-0.6.0.linux-amd64.tar.gz

mv process-exporter-0.6.0.linux-amd64 /usr/local/process-exporter

chown prometheus.prometheus -R /usr/local/process-exporter/
```

## 2. 修改配置文件

## 2.1 配置文件示例

默认情况下，process-exporter没有配置文件，我们可以添加一个示例：

```
process_names:
  - name: "{{.Matches}}"
    cmdline:
      - '/alidata/server/tengine/sbin/nginx'
```

## 2.2 配置项说明

上面的示例中，name选项有四个：
* {{.Comm}} 包含原始可执行文件的基本名称，即/proc/<pid>/stat这个文件中的第2个字段
* {{.ExeBase}} 包含可执行文件的基名
* {{.ExeFull}} 包含可执行文件的完全限定路径
* {{.Username}} 包含有效用户的用户名
* {{.Matches}} map包含应用cmdline regexps产生的所有匹配项

假设说我们有一个进程如下：
```
# ps -ef | grep redis
redis 771 1 0 Jun05 ? 00:45:49 /usr/bin/redis-server *:6379
```
下面是四个name的分别说明；
name|groupname|description
---|---|---
{{.Comm}} |	"redis-server"|	exe或者sh文件名称
{{.ExeBase}}|	"redis-server *:6379"|	
{{.ExeFull}} |	"/usr/bin/redis-server *:6379"|	ps中的进程完成信息
{{.Username}}	|"redis"|	使用进程所属的用户进行分组
{{.Matches}}|	"map[:redis]"	|表示配置到关键字“redis”

## 3. 配置启动文件
```
[Unit]
Description=blackbox_exporter
After=network.target 

[Service]
User=prometheus
Group=prometheus
WorkingDirectory=/usr/local/process-exporter
ExecStart=/usr/local/process-exporter/process-exporter -config.path=/usr/local/process-exporter/process.yml
[Install]
WantedBy=multi-user.target
```

## 4. 启动process-exporter

```
systemctl daemon-reload
systemctl enable process-exporter
```

# 修改prometheus配置添加job

在prometheus配置文件中添加如下job：

```
  - job_name: "process"
    static_configs:
    - targets:
      - "192.168.0.107:9256"
```

> process-exporter默认监听在9256端口上

# 导入监控模板

## 1. prometheus web ui查询示例

查询指定process的进程总数：
![image](http://note.youdao.com/yws/res/105/B914C7D35116488980F7C0F077E41695)


查询僵尸进程：

![image](http://note.youdao.com/yws/res/107/C3176C6AB88445FDBE00D96A82C4B883)

## 2. grafana监控图

导入dashboard 8378：

![image](http://note.youdao.com/yws/res/106/7EBE9D57A33F4AB69933061626995234)


# 附录


通过kubernetes部署prcess-exporter:  https://blog.51cto.com/juestnow/2412634