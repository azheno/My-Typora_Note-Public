# RH134考试





使用RH134环境来模拟RHCSA考试: 

模拟系统： 

| **system**              | **IP Address** | **Role**         |
| ----------------------- | -------------- | ---------------- |
| servera.lab.example.com | 172.25.250.10  | 需要配置的虚拟机 |
| serverb.lab.example.com | 172.25.250.11  | 需要配置的虚拟机 |



帐户信息:

f0主机的root密码为Asimov，f0上其他用户的密码均为redhat

f0里面所有虚拟系统的 root 密码是 redhat，请勿更改 root 密码。serverb的root密码未知，需要进行密码重置，要求请查看后面的题目。所有系统上已预装了 SSH 密钥，允许在不输入密码的前提下通过 SSH 进行 root 访问。请勿对系统上的 root SSH 配置文件进行任何修改。



初始化环境

```shell
[root@foundation0 ~]# rht-vmctl fullreset classroom
[root@foundation0 ~]# rht-vmctl fullreset all
```



开启虚拟机：

```shell
[root@foundation0 ~]# rht-vmctl start all
```

>   说明：考试需要通过图形界面对虚拟机进行开机(start)，关机(poweroff)，重启(reboot)和重置(rebuilt)操作，重置虚拟机后，虚拟机所有的配置将会清空。



## servera

### 1、配置网络地址

虚拟机servera的roo密码为redhat，请为此虚拟机配置以下网络参数：

1）主机名 ： servera.lab.example.com

2）IP地址：172.25.250.10/24

3）默认网关：172.25.250.254

4）DNS服务器：172.25.250.254

```shell
# 设置主机名
[root@localhost ~]# hostnamectl set-hostname servera.lab.example.com
[root@localhost ~]# su -
[root@servera ~]# su -
# 设置ip地址，网关和dns
# 首先查看网卡配置文件，如果网卡没有配置文件需要自己创建并设置；若有配置文件直接修改即可
[root@servera ~]# nmcli con show
# 创建配置文件并设置地址
[root@servera ~]# nmcli con add ifname enp1s0 type ethernet con-name ens1s0 ipv4.addresses 172.25.250.10/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254 ipv4.method manual
# 修改配置文件并设置地址
[root@servera ~]# nmcli con mod ens1s0 ipv4.addresses 172.25.250.10/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254 ipv4.method manual connection.autoconnect yes
```



### 2、配置软件仓库

请配置你虚拟机的yum仓库，地址如下：

>   http://content.example.com/rhel8.2/x86_64/dvd/BaseOS 
>
>   http://content.example.com/rhel8.2/x86_64/dvd/AppStream

```shell
# 准备工作，清空yum仓库，考试不需要做
[root@servera ~]# rm -rf /etc/yum.repos.d/*
#实际操作
[root@servera ~]# vim /etc/yum.repos.d/rhel_dvd.repo
[baseos]
name = Red Hat Enterprise Linux 8.2 BaseOS
baseurl = http://content.example.com/rhel8.2/x86_64/dvd/BaseOS
enabled = 1
gpgcheck = 0
[appstream]
name = Red Hat Enterprise Linux 8.2 AppStream
baseurl = http://content.example.com/rhel8.2/x86_64/dvd/AppStream
enabled = 1
gpgcheck = 0
[root@servera ~]# yum clean all
12 files removed
[root@servera ~]# yum repolist all
Red Hat Enterprise Linux 8.2 AppStream (dvd) 51 MB/s | 5.3 MB
00:00
Red Hat Enterprise Linux 8.2 BaseOS (dvd) 38 MB/s | 2.2 MB
00:00
repo id repo name
status
rhel-8.0-for-x86_64-appstream-rpms Red Hat Enterprise Linux 8.2 AppStream
(dvd) enabled: 4,672
rhel-8.0-for-x86_64-baseos-rpms Red Hat Enterprise Linux 8.2 BaseOS
(dvd) enabled: 1,65
```





### 3、调试 SELinux

配置 httpd 在 82 端口上提供 Web 服务，满足以下要求：

1）在/var/www/html/下添加 index.html 文件

2）此 Web 服务器在系统启动时自动启动

3）确保 SELinux 保护机制运行在 Enforcing 模式

```shell
# 准备工作，安装httpd服务，修改监听的端口号为80，考试不需要做。
[root@servera ~]# yum -y install httpd
[root@servera ~]# systemctl enable --now httpd
[root@servera ~]# echo hello yutianedu > /var/www/html/index.html
[root@servera ~]# vim /etc/httpd/conf/httpd.conf # 修改监听的端口号
Listen 82
# 实际操作
[root@servera ~]# systemctl restart httpd #重启服务报错，原因是selinux
# 考试需要查看监听的端口号为多少，再将对应的端口号（82端口）设置为httpd的标准端口即可，
man semanage-port
[root@servera html]# semanage port -l | grep http #搜索80对应的端口号
[root@servera ~]# semanage port -a -t http_port_t -p tcp 82
#重启服务
[root@servera ~]# systemctl restart httpd
#查看端口号82已经监听
[root@servera ~]# netstat -tulnp | grep httpd
tcp6 0 0 :::82 :::* LISTEN
25747/httpd
[root@servera ~]# curl http://servera:82
hello yutianedu
# 说明：考试环境http已经配置好了，考生只需要将82号端口设置为httpd的标准端口即可
```



### 4、创建用户账户

根据下列要求创建用户及组账号：

1）名为 admins 的组

2）用户 harry，其附属组为 admins 

3）用户 natasha，其附属组还属于 admins

4）用户 alice，没有可交互的登录 Shell，且不属于 admins 组

5）harry、natasha、alice 的密码都应该是 redhat

```shell
[root@servera ~]# groupadd admins
[root@servera ~]# useradd -G admins harry
[root@servera ~]# useradd -G admins natasha
[root@servera ~]# useradd -s /sbin/nologin alice
[root@servera ~]# echo redhat | passwd --stdin harry
[root@servera ~]# echo redhat | passwd --stdin natasha
[root@servera ~]# echo redhat | passwd --stdin alice
```







### 5、配置 cron 计划任务

配置计划任务，以用户 harry 的身份每天 14:23 分执行一次命令 echo hello

```shell
[root@servera ~]# crontab -u harry -e
23 14 * * * echo hello
[root@servera ~]# crontab -u harry -l
23 14 * * * echo hello
分时日月周
*/2 每隔2分钟
10-12 10到12点
9,11 9点多，11点多
```





### 6、创建共享目录

创建具有以下特点的共用目录：

1）/home/tools 的所有组是 admins

2）此目录能被 admins 组的成员读取、写入和访问，除 root 外其他用户没有这些权限

3）在此目录下创建的文件，其组的所有权自动设置为 admins 组

```shell
[root@servera ~]# mkdir /home/tools
[root@servera ~]# chgrp admins /home/tools
[root@servera ~]# chmod g+w,o-rx /home/tools
[root@servera ~]# chmod g+s /home/tools
[root@servera ~]# ll -d /home/tools
drwxrws---. 2 root admins 6 Jun 17 16:30 /home/tools
[root@servera ~]# ll -d /home/tools
drwxrws---. 2 root admins 6 Jun 17 16:30 /home/tools
[root@servera ~]# touch /home/tools/file01
[root@servera ~]# ll /home/tools/file01
-rw-r--r--. 1 root admins 0 Jun 17 16:31 /home/tools/file01
```





### 7、配置 NTP 时间客户端

配置你的系统，使其成为 classroom.example.com 的 NTP 客户端。

```shell
[root@servera ~]# vim /etc/chrony.conf
server classroom.example.com iburst
[root@servera ~]# systemctl restart chronyd
[root@servera ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address Stratum Poll Reach LastRx Last sample
===============================================================================
^* classroom.example.com 8 6 17 1 -23us[ -23us] +/-561us
```





### 8、配置 autofs

准备工作：在 serverb 主机准备 NFS 服务端环境，过程如下，考试不需要配置按照以下要求，在 servera 上配置 autofs 自动挂载：

1） serverb 通过 NFS 共享目录/rhome 到你的系统，此文件系统中包含为用户 ldapuser0 预配置的家目录

2） 预设用户 ldapuser0 的家目录应自动挂载到本地的/rhome/ldapuser0 目录3）预设用户 ldapuser0 的家目录是 serverb.lab.example.com:/rhome/ldapuser0 4）预设用户 ldapuser0 的家目录应自动挂载到本地的/rhome/ldapuser0 目录

5）挂载后的家目录必须可读写

```shell
#准备工作
[root@serverb ~]# mkdir -p /rhome/ldapuser0
[root@serverb ~]# vim /etc/exports
/rhome/ldapuser0 *(rw)
[root@serverb ~]# systemctl restart nfs-server
[root@serverb ~]# systemctl enable nfs-server
[root@serverb ~]# firewall-cmd --permanent --add-service=rpc-bind
[root@serverb ~]# firewall-cmd --permanent --add-service=nfs
[root@serverb ~]# firewall-cmd --permanent --add-service=mountd
[root@serverb ~]# firewall-cmd --reload

#正式写题
[root@servera ~]# showmount -e serverb.lab.example.com
Export list for 172.25.250.11:
/rhome/ldapuser0 *
[root@servera ~]# yum -y install autofs
[root@servera ~]# systemctl enable --now autofs
[root@servera ~]# vim /etc/auto.master
/rhome /etc/auto.nfs
[root@servera ~]# vim /etc/auto.nfs
ldapuser0 -rw serverb.lab.example.com:/rhome/ldapuser0
[root@servera ~]# systemctl restart autofs
[root@servera ~]# cd /rhome/
[root@servera rhome]# ls
[root@servera rhome]# cd ldapuser0 # 触发挂载，考试的时候用su - ldapuser0来触
发，cd触发如果报权限拒绝，此为正常现象。
[root@servera ldapuser0]# df -h
Filesystem Size Used Avail Use% Mounted on
devtmpfs 892M 0 892M 0% /dev
tmpfs 915M 0 915M 0% /dev/shm
tmpfs 915M 17M 898M 2% /run
tmpfs 915M 0 915M 0%
/sys/fs/cgroup
/dev/vda1 10G 1.6G 8.5G 16% /
tmpfs 183M 0 183M 0% /run/user/0
serverb.lab.example.com:/rhome/ldapuser0 10G 1.5G 8.6G 15%
/rhome/ldapuser0
#如果出现挂载不上的情况，先停止autofs服务，再进行排错
[root@servera ~]# systemctl stop autofs
```





### 9、配置文件权限

将文件/etc/fstab 复制到/var/tmp/，并按要求配置/var/tmp/fstab 的权限：

1）文件/var/tmp/fstab 属于 root 用户

2）文件/var/tmp/fstab 属于 root 组

3）任何用户对/var/tmp/fstab 都没有可执行权限4）用户 harry 能够读取和写入/var/tmp/fstab 文件5）用户 natasha 无法读取或写入/var/tmp/fstab 文件

6）所有其他用户（包括未来创建的用户）能够读取/var/tmp/fstab 文件

```shell
[root@servera ~]# cp /etc/fstab /var/tmp/
[root@servera ~]# ll /var/tmp/fstab
-rw-r--r--. 1 root root 427 Jun 17 17:06 /var/tmp/fstab
[root@servera ~]# setfacl -m u:harry:rw- /var/tmp/fstab
[root@servera ~]# setfacl -m u:natasha:--- /var/tmp/fstab
[root@servera ~]# getfacl /var/tmp/fstab
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/fstab
# owner: root
# group: root
user::rw-
user:harry:rw-
user:natasha:---
group::r--
mask::rw-
other::r--
```





### 10、配置用户账户

创建用户 tammy，其用户 ID 为 2020，此用户的密码应当是 redhat。

```shell
[root@servera ~]# useradd -u 2020 tammy
[root@servera ~]# echo redhat | passwd --stdin tammy
```



### 11、查找文件

请把系统上拥有者为用户的所有文件，并将其拷贝到/root/findfiles 目录中。

```shell
[root@servera ~]# mkdir /root/findfiles
[root@servera ~]# find / -user tammy -exec cp -a {} /root/findfiles/ \;
[root@servera ~]# ll findfiles/
-size : +10k 大于10K -10 小于10k
find / -size +10k -size -20k 大于10k小于20k的文件
-perm : 755 /2000 查找用sgid权限的文件
find / -perm /2000
```





### 12、查找字符串

找出文件/etc/man_db.conf 中包含字符串sbin 的所有行，将其按原始顺序导入到文件/root/out.txt 中， 文件/root/out.txt 中不得包含空行。

```shell
[root@servera ~]# grep sbin /etc/man_db.conf >> /root/out.txt
#验证
[root@servera ~]# grep sbin /etc/man_db.conf | wc -l
[root@servera ~]# wc -l /root/out.txt
```





### 13、创建归档

创建一个名为/root/backup.tgz 的 tar 归档，其应该包含/usr/local/的内容。此归档文件必须使用gzip 进行压缩。

```shell
[root@servera ~]# tar -jcf /root/backup.tar.bz2 /usr/local/
[root@servera ~]# file backup.tar.bz2 # 验证
backup.tar.bz2: bzip2 compressed data, block size = 900k
#几种压缩：
-z : gzip
-j : bzip2
-J : xz
```





### 14、配置 sudo 提权

允许 admins 组的成员可以以 root 的身份执行任何命令，并且不需要密码。

```shell
[root@servera ~]# vim /etc/sudoers.d/admins
%admins ALL=(root) NOPASSWD: ALL
#测试：
[root@servera ~]# su - harry
[harry@servera ~]$ sudo useradd testuser #创建成功
```





### 15、设置密码默认策略

为新创建的用户设置密码策略，要求创建用户时，密码默认 20 天后过期

```shell
[root@servera ~]# vim /etc/login.defs
PASS_MAX_DAYS 20
# 测试：
[root@servera ~]# useradd user1
[root@servera ~]# cat /etc/shadow
user1:!!:18782:0:20:7:::
```





### 16、配置 umask 值

设置 natasha 用户创建的目录权限默认为 733，文件权限为 622，要求永久生效

```shell
# 只需要根据目录的权限推算出umask即可，目录最大权限为777，去掉733的权限，得到umask为044
# 为natasha设置umask值
[root@servera ~]# su - natasha
[natasha@servera ~]$ vim .bashrc
umask 044
[natasha@servera ~]$ source .bashrc
[natasha@servera ~]$ mkdir test
[natasha@servera ~]$ touch file
[natasha@servera ~]$ ll
total 0
-rw--w--w-. 1 natasha natasha 0 Jun 5 01:37 file
drwx-wx-wx. 2 natasha natasha 6 Jun 5 01:37 test
```





### 17、编辑应用程序

编辑 ex200程序，如果pandora用户执行则输出This is a rhcsa exam

```shell
# 查询程序位置
[root@node1 ~]# which ex200
/usr/bin/ex200
# 修改文件内容
[root@node1 ~]# echo "echo 'This is RHCSA Exam' " > /usr/bin/ex200
# 验证
[root@node1 ~]# ex200
This is RHCSA Exam
```



### 18、创建用于定位文件的脚本

（1）   创建名为/usr/local/bin/file.sh 的脚本，

（2）   查找 /usr 下面所有小于 10M 并有设置 sgid 权限的文件

（3）   将查找到的文件列表保存到 /root/myfile 中

```shell
[root@servera ~]# vim /usr/local/bin/file.sh
#!/bin/bash
find /usr -size -10M -perm -2000 > /root/myfile
[root@servera ~]# chmod a+x /usr/local/bin/file.sh
[root@servera ~]# file.sh
[root@servera ~]# cat /root/myfile
/usr/bin/write
/usr/bin/locate
/usr/libexec/utempter/utempter
/usr/libexec/openssh/ssh-keysign
```





### 19、利用注册表服务器上的 httpd 镜像，创建名为 httpserver的容器

1）用 registry 服务器提供的 httpd-24:1-105 镜像创建容器 , 容器仓库服务器为

registry.lab.example.com/rhel8/httpd-24:1-105

说明：考试没有告诉镜像的地址，需要用 podman   search  httpd-24:1-105 搜索得到镜像仓库的地址。

2）将其配置为以 systemd 服务的形式运行，且仅面向现有用户 contsvc

3）该服务应命名为 container-httpserver，此服务在系统重启后将自动启动

### 20、通过以下方式扩展上一个任务

1）配置 systemd-journald 服务，要求永久保留日志数据

2）将 /var/log/journal 目录及任何子目录中的任何 *.journal 文件复制到 /home/contsvc/container_journal 中

3）容器服务启动时能自动将 /home/contsvc/container_ journal 挂载到容器中的/var/log/journal 上



```shell
#准备工作
[root@servera ~]# wget -P /etc/yum.repos.d/
http://materials.example.com/updates.repo
#开始做实验之前，需要先更新yum仓库，再敲下面的lab命令
[student@workstation ~]$ lab containers-services start
#实际操作
[student@workstation ~]$ ssh root@servera
[root@servera ~]# useradd contsvc
[root@servera ~]# echo redhat|passwd --stdin contsvc
#创建日志永久存储目录，并重启journald服务，这一步是为了生成在/var/log/journal 目录中的
生成 *.journal的日志文件
[root@servera ~]# mkdir /var/log/journal
[root@servera ~]# systemctl restart systemd-journald.service
[root@servera ~]# exit
#注意后面的步骤需要用contsvc用户ssh登录servera操作，不能su -切换到contsvc
[student@workstation ~]$ ssh contsvc@servera
[contsvc@servera ~]$ mkdir -p ~/.config/containers/
[contsvc@servera ~]$ cp /tmp/containers-services/registries.conf
~/.config/containers
#说明：考试时该配置被复制的文件路径是/etc/containers/registries.conf
#cp /etc/containers/registries.conf ~/.config/containers
[contsvc@servera ~]$ mkdir -p ~/container_journal
[contsvc@servera ~]$ cp -a /var/log/journal/*/*.journal
/home/contsvc/container_journal/
[contsvc@servera ~]$ podman login registry.lab.example.com
Username: admin
Password: redhat321
#启动容器，普通用户无法使用1024以下的端口，因此选择8080，考试看题目要
求，/var/log/journal:Z 后面的Z是podman将Selinux的container_file_t的标签值应用到宿
主机的目录上，以便容器可以访问宿主机的目录。
[contsvc@servera ~]$ podman search httpd-24:1-105 #考试需要自己搜索路径，该环境目
前无法搜索到路径。
[contsvc@servera ~]$ podman run -d --name httpserver -p 8080:8080 -v
~/container_journal:/var/log/journal:Z registry.lab.example.com/rhel8/httpd24:1-105
#选项解释：
run: 运行容器
-d: 后台运行容器
--name: 指定容器的名字
-p: 指定宿主机和容器的端口映射，宿主机端口:容器端口
-v: 指定挂载映射，宿主机路径:容器路径
#查看容器进程
[contsvc@servera ~]$ podman ps
#进入容器
[contsvc@servera ~]$ podman exec -it fe74aba1d977 /bin/bash
bash-4.4$
bash-4.4$ cd /var/log/journal/
bash-4.4$ ls
user-1002.journal #挂载成功
#创建systemd单元文件，
[contsvc@servera ~]$ mkdir -p ~/.config/systemd/user/
[contsvc@servera ~]$ cd ~/.config/systemd/user/
[contsvc@servera user]$ podman generate systemd --name httpserver --files --
new
#选项解释：
--name: 容器名称
--files: 生成服务配置文件，不加则内容在终端显示
--new: 设置服务启动时创建容器，在服务停止时删除容器，这种模式下，容器时临时的，需要配
置永久存储来保存数据。如果不加，服务启动和停止时，容器也会相应启动和停止，而不会删除容器。
#停止容器并删除容器
[contsvc@servera ~]$ podman stop fe74aba1d977
[contsvc@servera ~]$ podman rm fe74aba1d977
#重新加载配置文件
[contsvc@servera user]$ systemctl --user daemon-reload

```







## serverb

### 21、设置 root 密码

将 serverb 的 root 密码设置为 123456

```shell
1. 重启系统
2. 将光标移动到要启动的内核
3. 按e编译当前条目
4. 将光标移至以linux开头的行，此为内核命令行
5. 在末尾添加rd.break
6. 按ctrl+x继续启动
7.重启以读写方式挂载/sysroot
switch_root:/# mount -o remount,rw /sysroot
8. 切换至真正操作系统的根/sysroot
switch_root:/# chroot /sysroot
9. 重置密码
sh-4.4# echo 123456 | passwd --stdin root
10. 打标签
sh-4.4# touch /.autorelabel
exit退出两次
```





### 22、配置软件仓库

请配置你虚拟机的yum仓库,地址如下： 

>   http://content.example.com/rhel8.2/x86_64/dvd/BaseOS 
>
>   http://content.example.com/rhel8.2/x86_64/dvd/AppStream

```shell
# 准备工作，清空yum仓库，考试不需要做
[root@serverb ~]# rm -rf /etc/yum.repos.d/*
#实际操作
[root@serverb ~]# vim /etc/yum.repos.d/rhel_dvd.repo
[baseos]
name = Red Hat Enterprise Linux 8.2 BaseOS
baseurl = http://content.example.com/rhel8.2/x86_64/dvd/BaseOS
enabled = 1
gpgcheck = 0
[appstream]
name = Red Hat Enterprise Linux 8.2 AppStream
baseurl = http://content.example.com/rhel8.2/x86_64/dvd/AppStream
enabled = 1
gpgcheck = 0
[root@serverb ~]# yum clean all
12 files removed
[root@serverb ~]# yum repolist all
[root@serverb ~]# yum list all

```





### 23、调整逻辑卷大小

1）预先创建 2GiB 的分区/dev/vdb1，并用于创建卷组 testvg

2）创建大小为 200MiB 的逻辑卷/dev/testvg/vo，格式化为 ext4 文件系统，并挂载在/mnt/vo上

3）将逻辑卷/dev/testvg/vo 及其文件系统大小调整到 300MiB，确保文件系统内容保持不变。

```shell
# 准备工作
[root@serverb ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xae75bf0a.
Command (m for help): n
Partition type
p primary (0 primary, 0 extended, 4 free)
e extended (container for logical partitions)
Select (default p):
Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759):
+2G
Created a new partition 1 of type 'Linux' and of size 2 GiB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
[root@serverb ~]# pvcreate /dev/vdb1
[root@serverb ~]# vgcreate testvg /dev/vdb1
[root@serverb ~]# lvcreate -L 200M -n vo testvg
[root@serverb ~]# mkfs.ext4 /dev/testvg/vo
[root@serverb ~]# blkid /dev/testvg/vo
/dev/testvg/vo: UUID="fe323058-93db-428d-883c-2c0210cd10c6" TYPE="ext4"
[root@serverb ~]# vim /etc/fstab
/dev/testvg/vo /mnt/vo ext4 defaults 0 0
[root@serverb ~]# mount -a

# 实际操作
[root@serverb ~]# df -hT /dev/testvg/vo # 查看文件系统的类型和大小
Filesystem Type Size Used Avail Use% Mounted on
/dev/mapper/testvg-vo xfs 195M 12M 183M 6% /mnt/vo
[root@serverb ~]# lvextend -L 300M /dev/testvg/vo
[root@serverb ~]# lvs
LV VG Attr LSize Pool Origin Data% Meta% Move Log Cpy%Sync
Convert
vo testvg -wi-ao---- 300.00m
# 扩展文件系统，xfs类型的文件系统用xfs_growfs /mnt/vo，后面接的是挂载点。
[root@serverb ~]# resize2fs /dev/testvg/vo # 后面接的是挂载点的路径
[root@serverb ~]# df -hT /dev/testvg/vo
Filesystem Type Size Used Avail Use% Mounted on
/dev/mapper/testvg-vo ext4 295M 13M 283M 5% /mnt/vo
```





### 24、添加交换分区

在系统上添加一个 512MiB 的交换分区，设置交换分区应在系统启动时自动挂载，不要删除或修改系统上已存在的交换分区。

```shell
[root@serverb ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): n
Partition type
p primary (1 primary, 0 extended, 3 free)
e extended (container for logical partitions)
Select (default p):
Using default response p.
Partition number (2-4, default 2):
First sector (4196352-10485759, default 4196352):
Last sector, +sectors or +size{K,M,G,T,P} (4196352-10485759, default
10485759): +512M
Created a new partition 2 of type 'Linux' and of size 512 MiB.
Command (m for help): w
The partition table has been altered.
Syncing disks.
[root@serverb ~]# mkswap /dev/vdb2
[root@serverb ~]# blkid /dev/vdb2
/dev/vdb2: UUID="f626f11c-4363-4caf-aba6-5a418ea04079" TYPE="swap"
PARTUUID="ae75bf0a-02"
[root@serverb ~]# vim /etc/fstab
UUID=f626f11c-4363-4caf-aba6-5a418ea04079 swap swap defaults 0 0
[root@serverb ~]# swapon -a
[root@serverb ~]# swapon -s
Filename Type Size Used Priority
/dev/vdb2 partition 524284 0 -2
```





### 25、创建逻辑卷

根据以下要求，创建新的逻辑卷：

1）逻辑卷的名字为 mylv，属于 myvg 卷组，大小为 50 个 pe

2）卷组 myvg 中的逻辑卷的 pe 大小应当为 16MiB

3）使用ext3 文件系统将逻辑卷 mylv 格式化

4）此逻辑卷应当在系统启动时自动挂载到/mnt/mydata 目录下

```shell
[root@serverb ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): n
Partition type
p primary (2 primary, 0 extended, 2 free)
e extended (container for logical partitions)
Select (default p):
Using default response p.
Partition number (3,4, default 3):
First sector (5244928-10485759, default 5244928):
Last sector, +sectors or +size{K,M,G,T,P} (5244928-10485759, default
10485759): +1G
Created a new partition 3 of type 'Linux' and of size 1 GiB.
Command (m for help): w
The partition table has been altered.
Syncing disks.
[root@serverb ~]# pvcreate /dev/vdb3
[root@serverb ~]# vgcreate -s 16M myvg /dev/vdb3
[root@serverb ~]# lvcreate -l 50 -n mylv myvg
[root@serverb ~]# mkfs.ext3 /dev/myvg/mylv
[root@serverb ~]# mkdir /mnt/mydata
[root@serverb ~]# vim /etc/fstab
/dev/myvg/mylv /mnt/mydata ext3 defaults 0 0
[root@serverb ~]# mount -a
[root@serverb ~]# df -h /mnt/mydata/
Filesystem Size Used Avail Use% Mounted on
/dev/myvg/mylv 799M 4.0K 799M 1% /mnt/mydata
```





### 26、创建VDO卷

根据如下要求，创建新的 VDO 卷：

1）使用未分区的磁盘（/dev/vdc） 

2）此 VDO 卷的名称为 myvdo 

3）此 VDO 卷的逻辑大小为 50G

4）此 VDO 卷使用 xfs 文件系统格式化

5）此 VDO 卷在系统启动时自动挂载到/vblock 目录下

```shell
[root@serverb ~]# yum -y install vdo kmod-kvdo #安装软件包，考试需要安装
[root@serverb ~]# vdo create --name=myvdo --device=/dev/vdc --
vdoLogicalSize=50G # 命令man vdo 找example
Creating VDO myvdo
Starting VDO myvdo
Starting compression on VDO myvdo
VDO instance 0 volume is ready at /dev/mapper/myvdo
[root@serverb ~]# mkfs.xfs /dev/mapper/myvdo
[root@serverb ~]# vim /etc/fstab
/dev/mapper/myvdo /vblock xfs defaults,x-systemd.requires=vdo.service 0 0
#man vdo
[root@serverb ~]# mount -a
[root@serverb ~]# df -h /vblock
Filesystem Size Used Avail Use% Mounted on
/dev/mapper/myvdo 50G 390M 50G 1% /vblock
#删除vdo卷
[root@serverb ~]# vdo stop -n myvdo
[root@serverb ~]# vdo remove -n myvdo
```





### 27、配置系统调优

先查看 tuned 推荐的配置文件，并将系统的调优配置文件设置为 throughput-performance 提示：考试要求是将系统的调优配置文件设置为 tuned 推荐的配置文件。

```shell
# 说明：考试题目中当前生效和配置文件和推荐的配置文件不一致，而我们的虚拟机是一样的，考试只
需要按照下面的配置做即可。
[root@serverb ~]# tuned-adm recommend
virtual-guest
[root@serverb ~]# tuned-adm profile virtual-guest
[root@serverb ~]# tuned-adm list # 确保当前生效的配置文件是系统推荐的配置文件即可
Available profiles:
- balanced - General non-specialized tuned profile
- desktop - Optimize for the desktop use-case
- latency-performance - Optimize for deterministic performance at
the cost of increased power consumption
- network-latency - Optimize for deterministic performance at
the cost of increased power consumption, focused on low latency network
performance
- network-throughput - Optimize for streaming network throughput,
generally only necessary on older CPUs or 40G+ networks
- powersave - Optimize for low power consumption
- throughput-performance - Broadly applicable tuning that provides
excellent performance across a variety of common server workloads
- virtual-guest - Optimize for running inside a virtual guest
- virtual-host - Optimize for running KVM guests
Current active profile: virtual-guest
```









