# Docker安装



## 1. 使用原生yum源，下载docker,并设置开机自启

```shell
sudo yum install docker 
sudo systemctl enable docker --now 
```



## 2. 配置docker配置文件

```shell
vim /etc/docker/daemon.json  
{
        "registry-mirrors": ["https://b6jgdug7.mirror.aliyuncs.com"],
        "exec-opts": ["native.cgroupdriver=systemd"]
}

sudo vim /lib/systemd/system/docker.service  
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd \
          --host=fd:// \
#          --exec-opt native.cgroupdriver=systemd \  注释掉此行
          $OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
```



## 3. 重启服务

```shell
sudo systemctl daemon-reload 
sudo systemctl restart docker 
```





