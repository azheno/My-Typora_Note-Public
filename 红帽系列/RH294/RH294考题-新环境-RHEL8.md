# RH294考试



RHCE模拟练习题目，请使用教室环境RH294： 在练习期间，您将操作下列虚拟系统：

 

| **system**                  | **IP Address** | **Role**             |
| --------------------------- | -------------- | -------------------- |
| workstation.lab.example.com | 172.25.250.9   | Ansible control node |
| servera.lab.example.com     | 172.25.250.10  | Ansible managed node |
| serverb.lab.example.com     | 172.25.250.11  | Ansible managed node |
| serverc.lab.example.com     | 172.25.250.12  | Ansible managed node |
| serverd.lab.example.com     | 172.25.250.13  | Ansible managed node |
| bastion.lab.example.com     | 172.25.250.254 | Ansible managed node |

 

这些系统的 IP 地址采用静态设置，主机名称解析已配置为解析上方列出的主机名。 请勿更改这些设置。

帐户信息：

f0主机的root密码为Asimov，f0上其他用户的密码均为redhat

f0里面所有虚拟系统的 root 密码是 redhat，请勿更改 root 密码。所有系统上已预装了 SSH 密钥， 允许在不输入密码的前提下通过 SSH 进行 root 访问。请勿对系统上的 root SSH 配置文件进行任何修改。

Ansible 控制节点上已创建了用户 student。此帐户预装了 SSH密钥，允许在 Ansible 控制节点和各个

Ansible 受管节点之间进行SSH 登录。请勿对系统上的 student SSH 配置文件进行任何修改。

Ansible 被管理节点上已创建了用户devops。用于控制节点连接使用，考试时ssh免密和sudo提权已经全部配置好，请勿修改。



初始化环境： 

```shell
[root@foundation0 ~]# rht-vmctl fullreset classroom
[root@foundation0 ~]# rht-vmctl fullreset all
```

开启虚拟机：

```shell
[root@foundation0 ~]# rht-vmctl start all
```





### 一、安装和配置Ansible

按照下方所述，在控制节点workstation.lab.example.com 上安装和配置 Ansible：

1.   安装所需的软件包

2.   创建名为/home/student/ansible/inventory的静态清单文件, 以满足以下需求:

*   servera是dev主机组的成员

*   serverb是test主机组的成员

*   serverc和serverd是prod主机组的成员
*   bastion是balancers主机组的成员
*   prod组是webservers主机组的成员

3.   创建名为/home/student/ansible/ansible.cfg的配置文件, 以满足以下要求:

*	主机清单文件为/home/student/ansible/inventory
*	playbook中使用的角色的位置包括/home/student/ansible/roles

```shell
# 准备工作，由于教室环境中bastion虚拟机的用户devops未创建，需要创建，并设置sudo提权，和免密登陆，考试已经做好了。 
[kiosk@foundation0 ~]$ ssh root@workstation
[root@workstation ~]# for i in server{a..d} bastion; do ssh root@$i "useradd devops; echo redhat | passwd --stdin devops" ; done
[root@workstation ~]# for i in server{a..d} bastion; do ssh root@$i "echo 'devops ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/devops" ; done
[student@workstation ~]$ ssh devops@bastion #免密登陆成功
```





### 二、创建和运行Ansible临时命令

请按照下方所述, 创建一个名为/home/student/ansible/adhoc.sh的shell脚本, 该脚本将使用Ansible临时命令在各个受管节点上安装yum存储库:

存储库1:

*   存储库的名称为 rh294_BASE

*   描述为 rh294 base software

*   基础URL为 http://content.example.com/rhel8.4/x86_64/dvd/BaseOS

*   GPG签名检查为启用状态

*   GPG密钥URL为 http://content.example.com/rhel8.4/x86_64/dvd/RPM-GPG-KEY-redhat-releas 

*   存储库为开启状态

存储库2:

*   存储库的名称为 rh294_STREAM

*   描述为 rh294 stream software

*   基础URL为 http://content.example.com/rhel8.4/x86_64/dvd/AppStream

*   签名检查为启用状态

*   GPG密钥URL为 http://content.example.com/rhel8.4/x86_64/dvd/RPM-GPG-KEY-redhat-releas

*   存储库为开启状态

```shell
# 因为虚拟机自带yum源, 需要先删除, 考试时不需要操作 
[student@workstation ansible]$ for i in server{a..d} bastion; do ssh root@$i "rm -rf /etc/yum.repos.d/*"; done
```



### 三、安装软件包

创建一个名为 /home/student/ansible/packages.yml的 playbook：

1.   将 php 和 mariadb 软件包安装到 dev、test 和 prod 主机组中的主机上

2.   将 Development Tools 软件包组安装到 dev 主机组中的主机上

3.   将 dev 主机组中主机上的所有软件包更新为最新版本



### 四、使用RHEL系统角色

安装 RHEL 系统角色软件包，并创建符合以下条件的playbook

/home/student/ansible/timesync.yml：

1.   在所有受管节点上运行

2.   使用 timesync 角色

3.   配置该角色，以使用当前有效的 NTP 提供

4.   配置该角色，以使用时间服务器 classroom.example.com





### 四、使用RHEL系统角色  （更新）

安装RHEL角色，并使用SeLinux角色，要求在所有节点运行，将SELINUX设置为强制模式





### 五、使用Ansible Galaxy安装角色

使用 Ansible Galaxy 和要求文件 /home/student/ansible/roles/requirements.yml，从以下 URL 下载角色并安装到 /home/student/ansible/roles：

*   http://classroom.example.com/content/haproxy.tar.gz 此角色的名称应当为 balancer

*   http://classroom.example.com/content/phpinfo.tar.gz 此角色的名称应当为 phpinfo



### 六、创建和使用角色

根据下列要求，在 /home/student/ansible/roles中创建名为 apache 的角色：

1.   httpd软件包已安装，设为在系统启动时启用

2.   防火墙已启用并正在运行，并使用允许访问 Web 服务器的规则

3.   模板文件 index.html.j2 已存在，用于创建具有以下输出的文件 /var/www/html/index.html： 

     *   Welcome to HOSTNAME on IPADDRESS

     *   其中，HOSTNAME 是受管节点的完全限定域名，IPADDRESS 则是受管节点的 IP 地址。

4.   按照下方所述，创建一个使用此角色的 playbook /home/student/ansible/newrole.yml：该playbook 在webservers 主机组中的主机上运行





### 七、从Ansible Galaxy使用角色

根据下列要求，创建一个名为 /home/student/ansible/roles.yml 的 playbook：

1.   playbook 中包含一个 play，该play 在balancers 主机组中的主机上运行并将使用 balancer 角色。

*   此角色配置一项服务，以在 webservers 主机组中的主机之间平衡 Web 服务器请求的负载。

*   浏览到 balancers 主机组中的主机（例如 http://bastion.lab.example.com/ ）将生成以下输出：
    *   Welcome to serverc.example.com on 172.25.250.12

*   重新加载浏览器将从另一 Web 服务器生成输出：
    *   Welcome to serverd.example.com on 172.25.250.13

2.   playbook 中包含一个 play，该 play 在 webservers主机组中的主机上运行并将使用 phpinfo 角色。

*   通过 URL /hello.php 浏览到 webservers 主机组中的主机将生成以下输出：

    *   Hello PHP World from FQDN

    * 其中，FQDN是主机的完全限定名称。

* 例如，浏览到 http://serverc.lab.example.com/hello.php 会生成以下输出：

    *   Hello PHP World from serverc.lab.example.com

    *   另外还有 PHP 配置的各种详细信息，如安装的PHP 版本等。

*   同样，浏览到 http://serverd.lab.example.com/hello.php 会生成以下输出：

    *   Hello PHP World from serverd.lab.example.com

    *   另外还有 PHP 配置的各种详细信息，如安装的PHP 版本等。

 

### 八、创建和使用逻辑卷

创建一个名为/home/student/ansible/lv.yml 的playbook，它将在所有受管节点上运行以执行下列任务

1.    创建符合以下要求的逻辑卷：

*   逻辑卷创建在 research 卷组中逻辑卷名称为 data

*   逻辑卷大小为 600MiB

2.    使用 ext4 文件系统格式化逻辑卷

3.    如果无法创建请求的逻辑卷大小，应显示错误消息

*   `Could not create logical volume of that size`，并且应改为使用大小 400MiB。

4.   如果卷组research 不存在 ， 应显示错误消息

*   `Volume group does not exist`。

5.   不要以任何方式挂载逻辑卷。



### 八、创建分区

说明：由于只有一块可用磁盘，与上一道题冲突，需要重置磁盘后再做

创建一个名为/home/student/ansible/parted.yml 的playbook，它将在dev主机组上运行下列任务

1.   如果磁盘/dev/vdd存在，则创建1500m分区

2.   如果无法创建请求的分区大小，应显示错误消息

*   `Could not create partition of that size`，并且应改为使用大小 800m。

3.   如果磁盘/dev/vdd不存在，应显示错误消息

*   `disk /dev/vdd does not exist`。

4.   如果磁盘/dev/vdb存在，则创建1500m分区

5.   如果无法创建请求的分区大小，应显示错误消息

*   `Could not create partition of that size`，并且应改为使用大小 800m。

6.   最后分区都要格式化为ext4文件系统，并挂载在/mnt/fs01





### 九、生成主机文件

1.   编写模板文件/home/student/ansible/hosts.j2 ，针对每个清单主机包含一行内容，其格式与/etc/hosts 相同。

2.   创建名为 /home/student/ansible/hosts.yml 的playbook，它将使用此模板在 dev 主机组中的主机上生成文件 /etc/myhosts。

3.   该playbook 运行后，dev 主机组中主机上的文件/etc/myhosts 应针对每个受管主机包含一行内容。

注：清单主机名称的显示顺序不重要。





### 十、修改文件内容

按照下方所述，创建一个名为 /home/student/ansible/issue.yml 的 playbook：

1.   该 playbook 将在所有清单主机上运行

2.   该 playbook 会将 /etc/issue 的内容替换为下方所示的一行文本：

*   在 dev 主机组中的主机上，这行文本显示为：Development 

*   在 test 主机组中的主机上，这行文本显示为：Test

*   在 prod 主机组中的主机上，这行文本显示为：Production



### 十一、创建 Web 内容目录

按照下方所述，创建一个名为 /home/student/ansible/webcontent.yml 的 playbook：

1.   该 playbook 在 dev 主机组中的受管节点上运行

2.   创建符合下列要求的目录 /webdev： 拥有组为 devops 组

*   具有常规权限：owner=read+write+execute，group=read+write+execute，other=read+execute

*   具有特殊权限: set group ID

3.   用符号链接将 /var/www/html/webdev 链接到 /webdev

4.   创建文件 /webdev/index.html，其中包含如下所示的单行文本：Development

5.   在 dev 主机组中主机上浏览此目录（例如 http://servera.lab.example.com/webdev/ ）将生成以下输出：

*   Development





### 十二、生成硬件报告

创建一个名为 /home/student/ansible/hwreport.yml的 playbook，它将在所有受管节点上生成含有以下信息的输出文件 /root/hwreport.txt：

*   清单主机名称

*   以 MB 表示的总内存大小

*   BIOS 版本

*   磁盘设备 vda 的大小
*   磁盘设备 vdb 的大小

*   输出文件中的每一行含有一个 key=value 对。

您的 playbook 应当：

1.   从 http://172.25.254.254/content/hwreport.empty 下载文件，并将它保存为/root/hwreport.txt

2.   使用正确的值修改 /root/hwreport.txt

3.   如果硬件项不存在，相关的值应设为 NONE



### 十三、创建密码库

按照下方所述，创建一个 Ansible 库来存储用户密码：

1.    库名称为  /home/student/ansible/locker.yml

2.    库中含有两个变量，名称如下：

*   pw_developer，值为

*   Imadev pw_manager，值为 Imamgr

3.    用于加密和解密该库的密码为whenyouwishuponastar

4.    密码存储在文件  /home/student/ansible/secret.txt中



### 十四、创建用户账户

1.    从  http://172.25.254.254/content/user_list.yml 下载要创建的用户的列表，并将它保存到 /home/student/ansible，用户来自于/home/student/ansible/locker.yml文件。


2.    创建名为/home/student/ansible/users.yml 的playbook，从而按以下所述创建用户帐户： 

*   职位描述为 developer 的用户应当：
    *   在 dev 和test 主机组中的受管节点上创建
    *   从 pw_developer 变量分配密码
    *   是附加组 student 的成员

*   职位描述为 manager 的用户应当：
    *   在 prod 主机组中的受管节点上创建
    *   从 pw_manager 变量分配密码
    *   是附加组 opsmgr 的成员

3.   密码应采用 SHA512 哈希格式。

4.   您的 playbook 应能够在本次考试中使用在其他位置创建的库密码文件 /home/student/ansible/secret.txt 正常运行。

 

### 十五、更新Ansible库的密钥

按照下方所述，更新现有 Ansible 库的密钥：

1.   从 http://172.25.254.254/content/salaries.yml 下 载 Ansible 库 到 /home/student/ansible

2.   当前的库密码为 insecure4sure

3.   新的库密码为 bbe2de98389b

4.   库使用新密码保持加密状态





### 十六、创建计划任务 (新增)

为natasha创建一个计划任务，要求每隔2分钟执行一次echo hello，playbook 文件名为 cron.yml，该 playbook 在dev主机组上运行
