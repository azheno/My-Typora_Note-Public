ansible 自动化运维

 

web网站

 

1.机房

2.买服务器 

3.交换机

4.装系统 u盘 pxe(与启动的执行环境) = dhcp + tftp + http + anaconda-ks.cfg

5.系统初始化

网络配置

sudo

ssh免密

防火墙

selinux

yum

。。。。。

6.部署中间件

mysql/oracle/redis

nginx /http

7.代码发布

  

 

1.puppet

ruby开发

 

2.saltstack 

python开发

模块化

shell: yum -y install httpd

cmd.run: yum -y install httpd

salt 192.168.40.10 -m cmd.run 'yum -y install httpd'

master：主控端安装master进程

agent：被控端安装agent

 

3.ansible

python开发

模块化

shell: yum -y install httpd

shell：yum -y install httpd touch /tmp/file.sh

不支持幂等性

  echo 111 >> /tmp/file.sh    

ansible 192.168.40.10 -m shell -a 'yum -y install httpd'

ansible 192.168.40.10 -m shell -a 'systemctl start httpd'

serverless：不需要部署服务端的进程

agentless：不需要部署agent，默认使用ssh

 

 

shell: systemctl start httpd

翻译成: 

主控端 ：XXX.py ~/ansible/tmp/XXX.py

被控端： ~/ansible/XXX.py

python ~/ansible/XXX.py

 

1.ansible的主机清单

定义主机和主机组[db]

选择主机和主机组(通配符和正则表达式)

--limit @retry.txt（-i retry.txt）

 

2.ansible的配置文件    

ini格式

[defaults] 通用配置

参数=值

优先级：

ANSIBLE_CONFIG=/tmp/ansible.cfg

./ansible.cfg

~/.ansible.cfg

/etc/ansible/ansible.cfg

 

3.ansible命令模块

shell：最强大的模块

command：不能识别 < > | &，如果没有指定模块名，默认是command

raw: 没有chdir creates removes，其他的功能与shell一样

script：执行脚本，该脚本在主控端

 

 

 

 

4.Ad-hoc

语法规则

ansible 主机或组 -m 模块名 -a '模块参数' ansible选项 

ansible选项：

-u：指定远程主机连接的用户

-k：密码验证

-i：指定主机清单文件

-f：指定并发数，默认是5

-m：指定模块名称

-a：指定模块的参数

-e：定义变量

ansible 192.168.40.10 -m shell -a 'systemctl start httpd'

 

 

查找工具

ansible-doc -l | grep yum列出所有的模块名

ansible-doc module 查看模块的参数和例子

 

 

模块

1.shell模块

free_form: shell命令

chdir: 切换工作目录，默认是远程连接用户的家目录

creates：一个文件名，当该文件存在，则该命令不执行

removes：一个文件名，当该文件不存在，则该命令不执行

 

2.file模块：管理被控端的文件，包括对文件和目录的创建，删除，硬链接，软连接

path：必选项，定义文件/目录的路径

state：

file：查看文件状态，默认选项，若文件不存在，也不会被创建，会报错文件不存在

touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新时间戳

absent：删除目录、文件或者取消链接文件

directory：如果目录不存在，创建目录

link：创建软链接

hard：创建硬链接

owner：属主

group：属组

mode：权限

src：要被链接的源文件的路径，只应用于state=link的情况

dest：被链接到的路径，只应用于state=link的情况

force：需要在两种情况下强制创建软链接，一种是源文件不存在但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no

 

 

3.copy模块：默认复制的主控端的文件到被控端

src：指定源文件

dest：指定目标文件，/tmp/redhat/ 后面带/，表示是个目录，目录不存在，则会创建目录再复制

force：默认是yes，目标文件与源文件内容不同，目标会被覆盖，内容相同不会覆盖。

backup：目标文件被覆盖前需要备份吗，默认是no

content: 将内容写入目标文件，可以重复执行

ansible node1 -m copy -a 'content="hello\n" dest=/tmp/file.txt'

remote_src：yes表示复制被控端的文件到被控端，默认是no，默认复制的是主控端的文件

ansible node1 -m copy -a 'src=/tmp/hosts dest=/tmp/redhat/data/ remote_src=yes'

 

 

4.yum_repository配置yum仓库

file：文件名，自动加上.repo

name: 仓库的名字

description：描述信息，相当于仓库里的name参数

baseurl

gpgcheck

enabled

gpgkey

state：present代表创建，absent代表删除，默认present

 

ansible node1 -m yum_repository -a 'file=rhce name=BaseOS state=absent'删除仓库

 

ansible node1 -m yum_repository -a 'file=rhce name=BaseOS description="BaseOS repo" baseurl=file:///media/BaseOS enabled=1 gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release'

ansible node1 -m yum_repository -a 'file=rhce name=AppStream description="AppStream repo" baseurl=file:///media/AppStream enabled=1 gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release'

 

5.yum 安装软件包

name: 软件包的名字

state：present代表创建，absent代表删除，latest安装最新的软件包，默认present

ansible node1 -m yum -a 'name="httpd" state=latest'

ansible node1 -m yum -a 'name="@Development Tools" state=present'

 

 

6.service管理服务

name：软件包的名字

state：reloaded, restarted, started, stopped

enabled： yes no 设置开机启动

 

7.systemd管理服务

name：软件包的名字

state：reloaded, restarted, started, stopped

enabled： yes no 设置开机启动

daemon_reload：当服务配置文件发生变更时重载服务

 

8.cron管理计划任务

 

ansible node1 -m cron -a 'name="display date" state=present user=root weekday=* minute=0 hour=12 job="date"' 相当于用crontab -e -u root创建

 

ansible test -m cron -a 'name="testcron minute" state=absent' 删除计划任务

 

cron_file: 指定配置文件

 

 

9.user管理用户

state：是创建还是删除

name：指定用户名

uid：指定用的uid

group：指定用户属组

groups：指定用户的附加组

append：yes表示追加附加组

comment：定义用户描述信息

create_home：是否创建家目录 yes|no

home：指定用户的家目录，需要与createhome配合使用

shell：指定用户的shell环境

password：指定用户的密码，加密过后的密码 redhat

remove：当state=absent时，remove=yes则表示连同家目录一起删除，等价于userdel -r



9.group管理组

gid：指定组id

name：指定组名

state：创建还是删除组，选项：present|absent

 

 

10.fetch 将被控端的文件复制到主控端，与copy相反

src：指定源文件

dest: 指定目标文件

flat:默认为no，表示在主控端目录下以被控端主机名的形式来组织目录结构。yes表示不会以被控端主机名的形式来组织目录结构，dest必须以/结尾。

 

ansible node1 -m fetch -a 'src=/tmp/hosts dest=/root/data/ flat=yes'

 

11.get_url下载文件

url：指定下载地址

dest：指定目标地址

owner：属主

group：属组

mode：权限

 

12.uncharive 将主控端的归档文件，直接在被控端解压缩

src：指定主控端的文件路径

dest：指定被控端的目标目录，目录必须事先存在

remote_src:将被控端的文件解压缩

creates：指定一个文件名，当该文件存在时，则解压指令不执行

 

 

ansible node1 -m unarchive -a 'src=etc.tgz dest=/tmp/data/'

 

13.synchronize同步，就是rsync

src: 指定被同步的目录 /etc同步整个目录 /etc/ 同步/etc下面的文件

dest：目标路径

rsync_opts:指定rsync的选项，每个选项之间用,隔开

举例：

ansible node1 -m synchronize -a 'src=/etc/ dest=/tmp/redhat'

  ansible node1 -m synchronize -a 'src=/etc/ dest=/tmp/redhat/data archive=no rsync_opts=-tr,--delete'

 

14.firewalld

ansible node1 -m firewalld -a 'service=http immediate=yes permanent=yes state=enabled'

\- firewalld:

  service: http

immediate: yes

  permanent: yes

  state: enabled

 

15.sefcontext管理文件的标签值

\- name: Allow apache to modify files in /srv/git_repos

 sefcontext:

  target: '/srv/git_repos(/.*)?'

  setype: httpd_git_rw_content_t

  state: present

 

\- name: Apply new SELinux file context to filesystem

 command: restorecon -irv /srv/git_repos

 

16.seboolean管理布尔值

\- name: Set httpd_can_network_connect flag on and keep it persistent across reboots

 seboolean:

  name: httpd_can_network_connect

  state: yes

  persistent: yes

 

 

 

## 配置web服务

1.配置yum仓库

ansible node1 -m yum_repository -a 'file=rhce name=BaseOS description="BaseOS repo" baseurl=file:///media/BaseOS enabled=1 gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release'

ansible node1 -m yum_repository -a 'file=rhce name=AppStream description="AppStream repo" baseurl=file:///media/AppStream enabled=1 gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release'

 

2.安装软件包http

ansible node1 -m yum -a 'name=httpd state=present'

 

3.编辑配置文件/etc/httpd/conf/httpd.conf

ansible node1 -m fetch -a 'src=/etc/httpd/conf/httpd.conf dest=/root/ansible/ flat=yes'

ansible node1 -m copy -a 'src=/root/ansible/httpd.conf dest=/etc/httpd/conf/httpd.conf'

 

4.启动httpd

ansible node1 -m service -a 'name=httpd enabled=yes state=started'

 

5.编辑index.html

ansible node1 -m copy -a 'content=hello dest=/var/www/html/index.html'

 

6.防火墙放行80端口

ansible node1 -m firewalld -a 'port=80/tcp immediate=yes permanent=yes state=enabled'

 

yaml语法规则：

1.大小写敏感

2.使用缩进表示层级关系

3.缩进时不允许使用Tab键，只允许使用空格。

4.缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

5.#表示注释，从这个字符一直到行尾。

 

 

playbook

 

\- hosts: node1

 tasks: 

   \- name: install httpd  # {"name": "install httpd" , "yum": {"name": "httpd", "state": "present"}} 

​    yum:

   state: present

​     name: httpd

​           

   \- name: fetch httpd.conf

​    fetch: 

​     src: /etc/httpd/conf/httpd.conf 

​     dest: /root/ansible/ 

​     flat: yes

 

   \- name: copy httpd.conf to remote hosts

​    copy:

​     src: /root/ansible/httpd.conf 

​     dest: /etc/httpd/conf/httpd.conf

 

   \- name: start httpd

​    service:

​     name: httpd 

​     enabled: yes 

​     state: started

   \- name: copy index.html to remote hosts

​    copy:

​     content: "hello\n" 

​     dest: /var/www/html/index.html

   \- name: access 80/tcp

​    firewalld:

​     port: 80/tcp 

​     immediate: yes 

​     permanent: yes 

​     state: enabled

 

\- hosts: node2

 tasks:

   \- name: 

 

[{"name": "install httpd" , "yum": {"name": "httpd", "state": "present"}},{} , {}, {} , {}, {} ]

 

string: [a,b,c,c]

​     0 1 2 3

string[0]

 

列表：

1.有序

2.独立

3.列表中的元素可以重复

\- marry

\- harry

\- john

 

[marry,harry,john]

 

字典：

key: value

1.由键值对组成

2.无序

3.key不可以重复，唯一

 

name: marry

age: 18

 

userinfo

{"name": "marry", "age": "18"}

userinfo[name]

 

name: Tom

age: 27

wife:

  name: Jerry

  age: 25

children:

 \- name: Jack

  age: 15

 \- name: Bob

  age: 14

 

string : {name: Tom, age: 27 , wife: {name: Jerry, age: 25}, children:[{name: Jack, age: 15},{name: Bob, age: 14}]}

 

playbook的基本语法

\- name: create user

 hosts: all

 remote_user: root

 gather_facts: false

 vars:

  user:"test"

 tasks:

  \- name: create user

   user: name="{{ user }}"

 

playbook的组成

Target section： 定义被控端主机和主机组

hosts: node1,node2,db 

Variable section： 定义变量

  vars：定义

Task section： 定义执行的任务

  tasks：

Handler section： 定义task执行完成以后需要调用的任务

 

 

handlers用法

1.handler定义触发的任务，在playbook最后执行

2.重要参数

force_handlers: yes 如果某个任务执行失败，那么handlers依然会执行

 

 

 

\- hosts: node2

 force_handlers: yes #只要触发到handler，handler的任务一定会执行

 tasks:

   \- name: install httpd

​    yum:

​     name: httpd

​     state: present

   \- name: copy httpd.conf to remote hosts

​    copy:

​     src: /root/ansible/httpd.conf

​     dest: /etc/httpd/conf/httpd.conf

​    notify: 

   \- restarted httpd

​     \- restarted mysql

   \- name: start httpd

​    service:

​     name: httpd

​     enabled: yes

​     state: started

   \- name: copy index.html to remote hosts

​    copy:

​     content: "hello node2\n"

​     dest: /var/www/html/index.html

  notify: 

   \- restarted httpd

   \- name: access 80/tcp

​    firewalld:

​     port: 8080/tcp

​     immediate: yes

​     permanent: yes

​     state: enabled

 handlers:

   \- name: restarted httpd

​    service:

​      name: httpd

​      state: restarted

 

task部分：

   \- name: copy index.html to remote hosts

​    copy:

​     content: | #代表换行

​       hello

​       yutianedu

 

忽略错误：

   \- name: copy httpd.conf to remote hosts

​    copy:

​     src: /root/ansible/httpd.con

​     dest: /etc/httpd/conf/httpd.conf

​    ignore_errors: yes # 如果该任务抛出异常，则忽略该错误，playbook继续运行

 

 

[a,b,c,d]

{name: [], age: {}}

 

列表

1.独立不相关[a,b,c]

2.有序

 

描述 

身高：160

体重：50

性别：女

 

[zhangsan, lisi]

 

字典

1.键值对 key:value

2.key

 

 

 

调用变量

1.如果是以变量开头，一定要加"{{ 变量 }}"

2.不以变量开头，变量的前面有字符串，用双引号全部引起来

"hello {{ 变量1 }} {{ 变量2 }}"

 

 

debug模块中

var: username 

 

debug模块：

  msg：后面可以接字符串和变量

  var：后面只能接变量名，不需要加{{}}

 

 

变量

1.在主机清单中定义内置变量（就是ansible.cfg中的参数）

ansible_remote_port: 定义远程主机的连接端口

ansible_remote_user：定义远程主机连接的用户

ansible all -m ping 测试连通性

ansible all -m shell -a 'pwd' 测试配置是否正确

 

 

2.在主机清单中定义自定义变量

node1 username=admin password=redhat #主机变量

[db:vars] #定义主机组变量

username=user1

 

主机变量优先

 

3.在playbook中通过vars引入变量，优先级高于在主机清单中定义，仅针对该playbook有效

\- hosts: node1,node2

 vars:

   username: user2

   password: 123

 tasks:

   \- name: debug some info

​    debug:

​      msg: "hello {{ username }}"

 

 

变量定义：key: value

userid: 2000

username: user1

userhome: /home/user11

groupname: it1

 

{,,,}

 

users:

  bjones:

  first_name: Bob

  last_name: Jones

  home_dirs: /users/bjones

  acook:

  first_name: Anne

  last_name: Cook

  home_dirs: /users/acook

  

  

{"users":"{"bjones":"{"first_name":Bob,"last_name":"Jones","home_dirs":"/users/bjones"}","acook":{"first_name":Anne,"last_name":"Cook","home_dirs":"/users/acook"}}"}

 

 

"{{ users.bjones.first_name }}"

"{{ users.bjones.home_dirs }}"

 

"{{ users['bjones']['first_name'] }}"

 

username: bjones

firstname: first_name

 

中括号里面不加单引号就是变量，加了引号就是字符串

"{{ users[username][firstname] }}"

 

username: bjones

 

 

 

users:

 \- bjones:

​    first_name: Bob

​    last_name: Jones

​    home_dirs: /users/bjones

 \- acook:

​    first_name: Anne

​    last_name: Cook

​    home_dirs: /users/acook

 

"{{ users.0.bjones.0.first_name }}"

 

"{{ users.0.bjones.first_name }}"

"{{ users[0]['bjones']['first_name'] }}"

\#说明：引用列表时，中括号内的索引不需要加引号

 

users: 

  \- first_name: Bob

   last_name: Jones

   home_dirs: /users/bjones

​    

  \- first_name: Anne

   last_name: Cook

   home_dirs: /users/acook

 

"{{ users.0.first_name }}"

"{{ users[0]["first_name"] }}"

 

 

4.在playbook中通过vars_files引入变量，优先级高于在主机清单中定义，仅针对该playbook有效

\- hosts: node1,node2

 vars_files:

   \- userlist

 tasks:

   \- name: debug some info

​    debug:

​      msg: "{{ users[0]['bjones']['first_name'] }}"

 

   \- name: debug some info

​    debug:

​      msg: "{{ users.0.bjones.first_name }}"

 

5.通过host_vars和group_vars引入变量

在playbook所在的目录下面创建host_vars和group_vars，host_vars的优先级高

[root@ansible ansible]# tree host_vars/

host_vars/

└── node1 #文件名是主机名，文件中定义变量的方式与vars是一样的

 

[root@ansible ansible]# tree group_vars/

group_vars/

└── db   #文件名是组名

 

6.注册变量

[root@ansible ansible]# cat register.yml 

\- hosts: node1

 tasks:

   \- name: gather some info

​    shell: ls /root

​    register: result #变量名

   \- name: debug result

​    debug:

​     msg: "{{ result.stdout_lines.0}}"

​    when: '"file" in result.stdout_lines' #一般注册变量和when语句组合使用

 

注册变量的内容

ok: [node1] => {

  "msg": {

​    "changed": true,

​    "cmd": "ls /root",

​    "delta": "0:00:00.007365",

​    "end": "2021-03-20 11:30:40.249555",

​    "failed": false,

​    "rc": 0,

​    "start": "2021-03-20 11:30:40.242190",

​    "stderr": "",

​    "stderr_lines": [],

​    "stdout": "anaconda-ks.cfg\nDesktop\nDocuments\nDownloads\nfile\nindex.html\ninitial-setup-ks.cfg\nMusic\nPictures\nPublic\nTemplates\nVideos",

​    "stdout_lines": [

​      "anaconda-ks.cfg",

​      "Desktop",

​      "Documents",

​      "Downloads",

​      "file",

​      "index.html",

​      "initial-setup-ks.cfg",

​      "Music",

​      "Pictures",

​      "Public",

​      "Templates",

​      "Videos"

​    ]

  }

}

 

7.通过ansible的选项-e或者--extra-vars引入变量，优先级最高

[root@ansible ansible]# ansible-playbook debug.yml -e "username=zhangsan password=redhat"

[root@ansible ansible]# ansible-playbook debug.yml -e "{ "username": "zhangsan", "password": "redhat"}"

 

 

内置变量 在inventory定义，修改ansible.cfg中的配置

自定义变量：

  inventory

​    node1:

​    [db:vars]

  playbook : vars vars_files set_facts模块

  host_vars

  group_vars/

  注册变量

  命令行中定义变量

  fact变量

  

 

定义和引用变量：通过字典和列表的方式

变量: key: value，是键值对，不是列表，但是value可以是列表

 

 

休息20分钟，11：02分回来

 

 

8.facts变量：收集被控端的信息

\- hosts: test

 gather_facts: false #关闭自动收集facts，默认是开启的

 tasks:

   \- name: test gather facts  #用setup模块收集

​    setup:

 

 

说明：filter后面的关键字只能是ansible_fact的下一层key

\# 查看网卡的信息

ansible test -m setup -a 'filter=ansible_ens160' 

\# 查看主机内存信息

ansible test -m setup -a 'filter=ansible_*_mb'

\# 将所有主机的信息输入到/tmp/facts目录下，每台主机的信息输入到主机名文件中（/etc/ansible/hosts里的主机名）

ansible all -m setup --tree /tmp/facts 

 

 

 

9.自定义facts

被控端的配置

[root@node1 ~]# cat /etc/ansible/facts.d/custome.fact 

[general]

package = httpd

service = httpd

state = started

 

[root@ansible ansible]# cat fact2.yml 

\- hosts: node1

 tasks:

   \- name: create facts dir

​    file:

​     path: /etc/ansible/facts.d

​     state: directory

   \- name: copy facts file to remote hosts

​    copy:

​     src: custome.fact

​     dest: /etc/ansible/facts.d/

   \- name: debug facts

​    debug:

​     msg: "{{ ansible_local.custome.general.package }}"

 

\#查找自定义fact变量

ansible node1 -m setup -a 'filter=ansible_local’ 查看被控端已经生成新的facts变量

 

10.用set_fact模块定义变量

  是在task中定义变量，并且可以在同一个文件的，下一个playbook中引用该变量，要在同一个被控端

  vars定义的变量不能再其他的playbook中引用

  注册变量是通set_facts一样，是可以同一个yml文件中下一个plalbook中引用

  

\- hosts: node1

 tasks: 

   \- name: create set facts var

​    set_fact: 

​      version: "{{ ansible_fqdn }} address is {{ ansible_ens160.ipv4.address }}" 

   \- name: debug set fact var 

​    debug:

​     msg: "{{ version }}"

 

\- hosts: node1

 tasks:

   \- name: debug var

​    debug:

​     msg: "{{ version }}"

 

 

11.lookup实现收集主控端的信息

\- hosts: node1

 tasks:

  \- name: set facts

   set_fact:

​    key_content: "{{ lookup('file','/root/.ssh/id_rsa.pub')}}"

  \- name: debug set facts

   debug:

​    msg: "{{ key_content }}"

 

\- hosts: node1

 tasks:

  \- name: set facts

   copy:

   content: "{{ lookup('file','/root/.ssh/id_rsa.pub')}}"

   dest: /tmp/a.key

  \- name: debug

   debug:

​     msg: "{{ lookup('pipe','pwd')}}"

  \- name: debug

   debug:

​     msg: "{{ lookup('env','HOSTNAME')}}"

 

12.魔法变量 

fact：在哪个主机上执行，那么就获取对应主机的信息

魔法变量：可以获取特定主机的fact变量

 

下面四个魔法变量都是从主机清单中获取信息

inventory_hostname: 在哪个主机上执行，就是哪个主机的主机名

groups: 主机清单中所有组的组名，跟在哪个主机上执行无关

groups.db : 打印db组中所有的主机

group_names: 在哪个主机上执行，则打印该主机所在的组

 

 

hostvars[inventory_hostname]

1.关闭facts的情况

收集的是上面四个魔法变量和其他定义的变量等主控端的信息

2.如果开启了facts

收集的内容除了以上四个魔法变量和其他定义的变量等主控端的信息以外，还会收集对应被控端的facts

 

如果想要收集特定主机的facts变量，

\#前提先要收集该主机的facts

\- hosts: node1

 gather_facts: false

 tasks:

   \- setup:

\- hosts: node2

 tasks:

   \- name: debug facts

​    debug:

​     msg: "{{ hostvars['node1'].ansible_ens160.ipv4.address }}"

 

hostvar['node1'] 收集node1的facts，跟在哪个被控端执行无关

 

 

 

whetn语句

\- name: Install vim

 hosts: all

 tasks:

  \- name: Install VIM via yum

   yum: 

​    name: vim-enhanced 

​    state: installed

   when: ansible_os_family =="RedHat"   

  \- name: Install VIM via apt

   apt: 

​    name: vim 

​    state: installed

   when: ansible_os_family =="Debian" 

  \- name: Unexpected OS family

   debug: msg="OS Family {{ ansible_os_family }} is not supported"

   when: not ansible_os_family =="RedHat" or ansible_os_family =="Debian"

 

 

when 后面变量不需要加"{{}}",字符串要用引号引起来

 

\- name: restart httpd if postfix is running

 hosts: test

 tasks:

  \- name: get postfix server status

   command: /usr/bin/systemctl is-active postfix

   ignore_errors: yes

   register: result

  \- name: restart apache httpd based on postfix status

   service:

​    name: httpd

​    state: restarted

   when: result.rc == 0

 

\#判断文件是否存在以及文件的类型：

file：判断指定路径是否为一个文件，是则为真

directory：判断指定路径是否为一个目录，是则为真

link：判断指定路径是否为一个软链接，是则为真

mount：判断指定路径是否为一个挂载点，是则为真

exists：判断指定路径是否存在，存在则为真

特别注意：关于路径的所有判断均是判断主控端上的路径，而非被控端上的路径

 

 

\- hosts: test

 vars:

  testpath: /testdir

 tasks:

  \- debug:

​    msg: "file exist"

   when: testpath is exists

 

\- hosts: test

 gather_facts: no

 vars:

  testpath1: "/testdir/test"

  testpath2: "/testdir"

 tasks:

  \- debug:

​    msg: "file"

   when: testpath1 is file

  \- debug:

​    msg: "directory"

   when: testpath2 is directory

 

 

判断文件类似于shell中test语句，该文件是主控端的文件

file is exists  判断文件存在吗

not file is exists  判断文件不存在

 

\#判断变量

bond0

ens160 

 

\- hosts: test

 gather_facts: no

 vars:

  testvar: "test"

  testvar1:

 tasks:

  \- debug:

​    msg: "testvar is defined"

   when: testvar is defined

 

defined：判断变量是否已定义，已定义则返回真

undefined：判断变量是否未定义，未定义则返回真

none：判断变量的值是否为空，如果变量已定义且值为空，则返回真

 

testvar is defined 判断变量存在吗

testvar is undefined 判断变量不存在

testvar is none 判断变量是否为空

 

 

 

\- hosts: test

 tasks:

  \- shell: 'cat /testdir/aaa'

   register: result

   ignore_errors: true

  \- debug:

​    msg: "success"

   when: result is success

  \- debug:

​    msg: "failed"

   when: result is failure

  \- debug:

​    msg: "changed"

   when: result is change

  \- debug:

​    msg: "skip"

   when: result is skip

 

 

\#判断字符串：

lower：判断字符串中的所有字母是否都是小写，是则为真

upper：判断字符串中的所有字母是否都是大写，是则为真

 

 

示例：

\- hosts: test

 gather_facts: no

 vars: 

  str1: "abc"

  str2: "ABC"

 tasks:

  \- debug:

​    msg: "str1 is all lowercase"

   when: str1 is lower

  \- debug:

​    msg: "str2 is all uppercase"

   when: str2 is upper

 

\#判断整除：

even：判断数值是否为偶数，是则为真

odd：判断数值是否为奇数，是则为真

divisibleby(num)：判断是否可以整除指定的数值，是则为真

 

\- hosts: test

 gather_facts: no

 vars: 

  num1: 6

  num2: 8 

  num3: 15

 tasks:

  \- debug: 

​    msg: "num1 is an even number"

   when: num1 is even

  \- debug:

​    msg: "num2 is an odd number"

   when: num2 is odd

  \- debug:

​    msg: "num3 can be divided exactly by"

   when: num3 is divisibleby(3)

 

休息20分钟，15：47分回来

 

[1] [1,2,3]

 

1 in [1,2,3]

 

\#其他条件测试：

subset 判断一个list是不是另一个list的子集：when: a is subset(b)

superset 判断一个list是不是另一个list的父集：when: b is superset(a)

in 判断一个字符串是否存在于另一个字符串中，也可用于判断某个特定的值是否存在于列表中

 

 

 

block：block里面的任务首先执行，如果写了when，那么when后面为真时才会执行block

rescue：当block执行任务失败时，则执行rescue里面的任务

always：里面的任务永远都会执行

 

 

\- hosts: test

 tasks:

   \- shell: ls /root

​    register: result

   \- debug:

​    msg: "file is exist"

​    when: '"Documents" in result.stdout_lines' #result.stdout_lines的值可以是一个列表,

 

tasks:

  \- block: block下面的任务一定会执行

​    \- name：

   when

   rescue: 当block执行失败时，会执行rescue

​    \- yum:

​    \- service

   always:

  

\- name: Install httpd

 hosts: test

 tasks:

  \- block:

​    \- name: Install httpd via yum

​     yum:

​      name: http

​      state: installed

​    \- name: start httpd via service

​     service:

​      name: httpd

​      state: started

​      enabled: yes

   when: ansible_os_family =="RedHat"

   rescue:

​    \- name: install postfix

​     yum:

​       name: postfix

​       state: present

   always:

​     \- debug:

​       msg: "always display message"

 

 

\- hosts: test

 tasks:

  \- block:

​    \- shell: 'ls /testdir'

   rescue:

​    \- debug:

​      msg: '/testdir is not exists'

   always:

​    \- debug:

​      msg: 'This task always executes'

 

\#fail模块：主动抛出异常，playbook停止运行

\- hosts: test

 tasks:

  \- shell: echo "Just a test--error" 

   register: result

  \- fail:

​    msg: "Conditions established,Interrupt running playbook"

   when: "'error' in result.stdout"

  \- debug:

​    msg: "I never execute,Because the playbook has stopped"

 

 

休息20分钟，10：59分回来

 

文件管理模块

file

copy 

fetch

get_url

unarchive

synchronize 

lineinfile

blockinfile

 

lineinfile:对某个特定的行操作

 

\#修改行

\- hosts: node2

 tasks:

  \- name: modify selinux to disabled

   lineinfile:

​    path: /etc/selinux/config

​    regexp: '^SELINUX'

​    line: 'SELINUX=disabled'

 

 

\#absent删除行

\- hosts: node2

 tasks:

  \- name: modify hosts

   lineinfile:

​    dest: /etc/hosts

​    regexp: '^127\.0\.0\.1'

​    state: absent

 

 

\#create：指文件不存在时，则创建该文件

\- hosts: node2

 tasks:

  \- name: Add a line to a file if the file does not exist, without passing regexp

   lineinfile:

​    path: /tmp/testfile

​    line: 192.168.40.10 node1

​    create: yes

 

 

 

\#validate：校验文件的语法，文件本身要支持校验

\- hosts: node2

 tasks:

  \- name: Validate the sudoers file before saving

   lineinfile:

​    path: /etc/sudoers

​    state: present

​    line: 'admin ALL=(ALL) NOPASSWD: ALL'

​    validate: /usr/sbin/visudo -cf %s

 

 

 

\#参数backrefs说明：

当backrefs为no时，如果regex没有匹配到行，则添加一行，如果Regx匹配到行，则修改该行

当backrefs为yes时，如果regex没有匹配到行，则保持原文件不变，如果regex匹配到行，则修改该行

\- hosts: node2

 tasks:

  \- name: test backrefs

   lineinfile:

​     path: /tmp/testfile

​     regexp: '^#bar'

​     backrefs: yes

​     line: 'bar'

 

blockinfile多行操作

1.追加多行

2.匹配到某行前面或者后面追加多行

 

\#参数说明：

block：要插入的文本内容

marker：指定块标记，"# {mark} ANSIBLE MANAGED BLOCK"

 

 

\- hosts: node1

 tasks:

  \- name: Insert/Update "Match User" configuration block in /tmp/sshd_config

   blockinfile:

​    path: /tmp/sshd_config

​    create: yes

​    block: |

​     Match User ansible-agent

​     PasswordAuthentication no

 

说明: blockinfile是通过mark标记来区分的

 

\- hosts: node2

 tasks:

  \- name: Insert/Update "Match User" configuration block in /tmp/sshd_config

   blockinfile:

​    path: /tmp/sshd_config

​    create: yes

​    marker: "#{mark} this is a mark" 

​    block: |

​     Match User ansible-agent

​     PasswordAuthentication no

 

删除多行，通过匹配标记来删除文件

\- hosts: node2

 tasks:

  \- name: Insert/Update "Match User" configuration block in /tmp/sshd_config

   blockinfile:

​    path: /tmp/sshd_config

​    marker: "#{mark} this is a mark"

​    state: absent

 

修改sshd

 

\- hosts: node2

 tasks:

  \- name: Insert/Update configuration using a local file and validate it

   blockinfile:

​    block: "{{ lookup('file', './local/sshd_config') }}"

​    path: /etc/ssh/sshd_config

​    backup: yes

​    validate: /usr/sbin/sshd -T -f %s

 

 

\- hosts: node2

 vars:

   package:

​     \- httpd

​     \- mysql

 tasks:

   \- name: install package

​    yum:

​     name:          

​     state: present

​    with_item: "{{ package }}"

 

 

\- hosts: node1

 gather_facts: no

 vars:

   mgroup:

​    \- group1

​    \- group2

​    \- group3

 tasks:

  \- name:

   group:

​    name: "{{ item }}"

​    state: present

   with_items:

​    "{{ mgroup }}"

 

循环列表

\- hosts: node1

 gather_facts: no

 vars:

  users:

   \- name: bjones

​    home_dirs: /home/bjones

​    uid: 2000

   \- name: acook

​    home_dirs: /home/acook

​    uid: 2001

​    

 tasks:

  \- name: create user 

   user:

​     name: "{{ item.name }}"

​     home: "{{ item.home_dirs}}"

​     uid:  "{{ item.uid }}"

   with_items:

​     "{{ users }}"   

 

 

循环字典：

\- hosts: test

 gather_facts: no

 vars:

  users:

   alice:

​    name: Alice Appleworth

​    telephone: 123-456-7890

   bob:

​    name: Bob Bananarama

​    telephone: 987-654-3210

  tasks:

   \- name:

​    debug:

​      msg: "{{ item.value.name }} telephone is {{ item.value.telephone }}"

​    with_dict: "{{users}}"

 

 

 

loop循环

 

1.loop默认是循环列表

\- hosts: node1

 tasks:

  \- name: postfix and httpd are running

   service:

​    name: "{{ item }}"

​    state: started

   loop:

​    \- postfix

​    \- httpd

 

\- name: registered variable usage as a loop list

 gather_facts: no

 hosts: node1

 tasks:

   \- name: ensure /mnt/bkspool exists

​    file:

​     path: /mnt/bkspool

​     state: directory

 

   \- name: retrieve the list of home directories

​    command: ls /home

​    register: home_dirs

   \- name: Show home_dirs results

​    debug:

​     var: home_dirs.stdout_lines

 

   \- name: add home dirs to the backup spooler

​    file:

​     path: /mnt/bkspool/{{ item }}

​     src: /home/{{ item }}

​     state: link

​     force: yes

​    loop: "{{ home_dirs.stdout_lines }}"

 

2.如果循环的是字典，那么将字典转换成列表，通过

loop: "{{ users | dict2items }}"

 

\- hosts: test

 gather_facts: no

 vars:

  users:

   alice: female

   bob: male

 tasks:

  \- debug:

​    msg: "{{ item.key }} is {{ item.value}}"

   loop: "{{ users | dict2items }}"

 

 

redis 内存数据库

key:value 

 

内存

监听的地址 

监听的端口号 6379

日志文件

file 

 

bind {{ ansible_ens160.ipv4.address }}

 

redis配置

 

{% if ansible_bond0 is defined %}

bind {{ ansible_bond0.ipv4.address }} 127.0.0.1

{% elif ansible_ens160 is defined %}

bind {{ ansible_ens160.ipv4.address }} 127.0.0.1

{% else %}

bind 0.0.0.0

{% endif %}

 

 

 

redis主从

slaveof 192.168.40.11 6379

 

{% if masterip is defined %}

 

slaveof {{ masterip }} {{ masterport|default(6379) }}

 

{% endif %}

 

 

 

循环 group.webservers

 

{% for host in group.webservers %}

server {{hostvars[host].ansible_ens160.ipv4.address}};

{% endfor %}

 

 

\#nginx.conf配置

  upstream web {

​     {% for host in groups.webservers %}

​     server {{ hostvars[host].ansible_ens160.ipv4.address }};

​     {% endfor %}

  }

 

 

  server {

​    listen    80 default_server;

​    listen    [::]:80 default_server;

​    server_name _;

​    root     /usr/share/nginx/html;

 

​    \# Load configuration files for the default server block.

​    include /etc/nginx/default.d/*.conf;

 

​    location / {

​       proxy_pass http://web;

​    }

 

\- hosts: webservers

 gather_facts: no

 tasks:

​     \- setup:

 

 

\#nginx.yml

\- hosts: proxy

 force_handlers: yes

 tasks:

   \- name: install nginx

​    yum:

​     name: nginx

​     state: present

   \- name: copy nginx.conf to remote hosts

​    template:

​     src: /root/ansible/nginx.conf

​     dest: /etc/nginx/nginx.conf

​    notify:

​     \- restarted nginx

   \- name: start nginx

​    service:

​     name: nginx

​     enabled: yes

​     state: started

   \- name: access 80/tcp

​    firewalld:

​     port: 80/tcp

​     immediate: yes

​     permanent: yes

​     state: enabled

   \- name: Set httpd_can_network_connect flag on and keep it persistent across reboots

​    seboolean:

​     name: httpd_can_network_connect

​     state: yes

​     persistent: yes

 handlers:

   \- name: restarted nginx

​    service:

​      name: nginx

​      state: restarted

 

 

 

\#webserver.yml

\- hosts: webservers

 force_handlers: yes

 tasks:

   \- name: install httpd

​    yum:

​     name: httpd

​     state: present

   \- name: copy httpd.conf to remote hosts

​    copy:

​     src: /root/ansible/httpd.conf

​     dest: /etc/httpd/conf/httpd.conf

​    notify:

​     \- restarted httpd

   \- name: start httpd

​    service:

​     name: httpd

​     enabled: yes

​     state: started

   \- name: copy index.html to remote hosts

​    template:

​     src: index.html

​     dest: /var/www/html/index.html

   \- name: access 80/tcp

​    firewalld:

​     port: 80/tcp

​     immediate: yes

​     permanent: yes

​     state: enabled

 handlers:

   \- name: restarted httpd

​    service:

​      name: httpd

​      state: restarted

 

 

休息20分钟，16：00分回来

 

 

 

\- hosts: test

 vars:

  paths:

   \- path: /tmp/test

​    mode: '0400'

   \- path: /tmp/foo

   \- path: /tmp/bar

 tasks:

  \- file:

​    path: "{{ item.path }}"

​    state: touch

​    mode: "{{ item.mode|default(omit)}}"

   with_items: "{{ paths }}"

 

 

jinja2过滤器

 mode: "{{ item.mode|default(omit)}}" item.mode没有定义，就忽略该参数

 port: "{{ masterport|default(6379)}}"  如果masterport没有定义，那么取默认值6379

"{{ (ansible_memtotal_mb / 3) | int }}"

 

 

[root@ansible ansible]# tree 

.

├── ansible.cfg

├── group_vars

│  └── db

├── host_vars

│  └── node1

├── inventory

├── webservers.yml

├── roles

│  ├── httpd

│  │  ├── files

│  │  │  └── httpd.conf

│  │  ├── handlers

│  │  │  └── main.yml

│  │  ├── tasks

│  │  │  └── main.yml

│  │  ├── templates

│  │  │  └── index.html

│  │  └── vars

│  │    └── main.yml

 

 

set paste