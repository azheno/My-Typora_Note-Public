# [MySQL 主从复制原理不再难](https://www.cnblogs.com/rickiyang/p/13856388.html)

[一文带你了解 Binlog 日志](https://www.cnblogs.com/rickiyang/p/13841811.html)

Binlog 日志主要作用是数据恢复和主从复制。本身就是二进制格式的日志文件，网络传输无需进行协议转换。MySQL 集群的高可用，负载均衡，读写分离等功能都是基于Binlog 来实现的。

#### MySQL 主从复制主流架构模型[#](https://www.cnblogs.com/rickiyang/p/13856388.html#1841806314)

我们基于 Binlog 可以复制出一台 MySQL 服务器，也可以复制出多台，取决于我们想实现什么功能。主流的系统架构有如下几种方式：

##### 1. 一主一从 / 一主多从

![1](./.图片存放/007S8ZIlgy1gjx1mx9n5wj30lu0mwgne.jpg)

一主一从和一主多从是最常见的主从架构方式，一般实现主从配置或者读写分离都可以采用这种架构。

如果是一主多从的模式，当 Slave 增加到一定数量时，Slave 对 Master 的负载以及网络带宽都会成为一个严重的问题。

##### 2. 多主一从

![1](./.图片存放/007S8ZIlgy1gjx1mt1km7j30m80l4407.jpg)

MySQL 5.7 开始支持多主一从的模式，将多个库的数据备份到一个库中存储。

##### 3. 双主复制

理论上跟主从一样，但是两个MySQL服务器互做对方的从，任何一方有变更，都会复制对方的数据到自己的数据库。双主适用于写压力比较大的业务场景，或者 DBA 做维护需要主从切换的场景，通过双主架构避免了重复搭建从库的麻烦。（主从相互授权连接，读取对方binlog日志并更新到本地数据库的过程；只要对方数据改变，自己就跟着改变）

##### 4. 级联复制

![1](./.图片存放/007S8ZIlgy1gjx1mr6mxzj30z60hg40u.jpg)

级联模式下因为涉及到的 slave 节点很多，所以如果都连在 master 上对主服务器的压力肯定是不小的。所以部分 slave 节点连接到它上一级的从节点上。这样就缓解了主服务器的压力。

级联复制解决了一主多从场景下多个从库复制对主库的压力，带来的弊端就是数据同步延迟比较大。

#### MySQL 主从复制原理[#](https://www.cnblogs.com/rickiyang/p/13856388.html#1197480470)

MySQL 主从复制涉及到三个线程：

一个在主节点的线程：`log dump thread`

从库会生成两个线程：一个 I/O 线程，一个 SQL 线程

如下图所示:

![4](./.图片存放/007S8ZIlgy1gjx1ms5ezhj312s0imtb5.jpg)

主库会生成一个 log dump 线程,用来给从库 I/O 线程传 Binlog 数据。

从库的 I/O 线程会去请求主库的 Binlog，并将得到的 Binlog 写到本地的 relay log (中继日志)文件中。

SQL 线程,会读取 relay log 文件中的日志，并解析成 SQL 语句逐一执行。

##### 主节点 log dump 线程

当从节点连接主节点时，主节点会为其创建一个 log dump 线程，用于发送和读取 Binlog 的内容。在读取 Binlog 中的操作时，log dump 线程会对主节点上的 Binlog 加锁；当读取完成发送给从节点之前，锁会被释放。**主节点会为自己的每一个从节点创建一个 log dump 线程**。

##### 从节点I/O线程

当从节点上执行`start slave`命令之后，从节点会创建一个 I/O 线程用来连接主节点，请求主库中更新的Binlog。I/O 线程接收到主节点的 log dump 进程发来的更新之后，保存在本地 relay-log（中继日志）中。

##### relay log

这里又引申出一个新的日志概念。MySQL 进行主主复制或主从复制的时候会在要复制的服务器下面产生相应的 relay log。

relay log 是怎么产生的呢？

从服务器 I/O 线程将主服务器的 Binlog 日志读取过来，解析到各类 Events 之后记录到从服务器本地文件，这个文件就被称为 relay log。然后 SQL 线程会读取 relay log 日志的内容并应用到从服务器，从而使从服务器和主服务器的数据保持一致。中继日志充当缓冲区，这样 master 就不必等待 slave 执行完成才发送下一个事件。

relay log 相关参数查询：

```sql
Copymysql>  show variables like '%relay%';
+---------------------------+------------------------------------------------------------+
| Variable_name             | Value                                                      |
+---------------------------+------------------------------------------------------------+
| max_relay_log_size        | 0                                                          |
| relay_log                 | yangyuedeMacBook-Pro-relay-bin                             |
| relay_log_basename        | /usr/local/mysql/data/yangyuedeMacBook-Pro-relay-bin       |
| relay_log_index           | /usr/local/mysql/data/yangyuedeMacBook-Pro-relay-bin.index |
| relay_log_info_file       | relay-log.info                                             |
| relay_log_info_repository | TABLE                                                      |
| relay_log_purge           | ON                                                         |
| relay_log_recovery        | OFF                                                        |
| relay_log_space_limit     | 0                                                          |
| sync_relay_log            | 10000                                                      |
| sync_relay_log_info       | 10000                                                      |
+---------------------------+------------------------------------------------------------+
11 rows in set (0.03 sec)
```

**max_relay_log_size**

标记 relay log 允许的最大值，如果该值为 0，则默认值为 max_binlog_size(1G)；如果不为 0，则max_relay_log_size 则为最大的 relay_log 文件大小。

**relay_log_purge**

是否自动清空不再需要中继日志时。默认值为1(启用)。

**relay_log_recovery**

当 slave 从库宕机后，假如 relay log 损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的 relay log，并且重新从 master 上获取日志，这样就保证了 relay log 的完整性。默认情况下该功能是关闭的，将 relay_log_recovery 的值设置为 1 时，可在 slave 从库上开启该功能，建议开启。

**relay_log_space_limit**

防止中继日志写满磁盘，这里设置中继日志最大限额。但此设置存在主库崩溃，从库中继日志不全的情况，不到万不得已，**不推荐使用。**

**sync_relay_log**

这个参数和 Binlog 中的 `sync_binlog`作用相同。当设置为 1 时，slave 的 I/O 线程每次接收到 master 发送过来的 Binlog 日志都要写入系统缓冲区，然后刷入 relay log 中继日志里，这样是最安全的，因为在崩溃的时候，你最多会丢失一个事务，但会造成磁盘的大量 I/O。

当设置为 0 时，并不是马上就刷入中继日志里，而是由操作系统决定何时来写入，虽然安全性降低了，但减少了大量的磁盘 I/O 操作。这个值默认是 0，可动态修改，建议采用默认值。

**sync_relay_log_info**

当设置为 1 时，slave 的 I/O 线程每次接收到 master 发送过来的 Binlog 日志都要写入系统缓冲区，然后刷入 relay-log.info 里，这样是最安全的，因为在崩溃的时候，你最多会丢失一个事务，但会造成磁盘的大量 I/O。当设置为 0 时，并不是马上就刷入 relay-log.info 里，而是由操作系统决定何时来写入，虽然安全性降低了，但减少了大量的磁盘 I/O 操作。这个值默认是0，可动态修改，建议采用默认值。

##### 从节点 SQL 线程

SQL 线程负责读取 relay log 中的内容，解析成具体的操作并执行，最终保证主从数据的一致性。

对于每一个主从连接，都需要这三个进程来完成。当主节点有多个从节点时，主节点会为每一个当前连接的从节点建一个 log dump 进程，而每个从节点都有自己的 I/O 进程，SQL 进程。

从节点用两个线程将从主库拉取更新和执行分成独立的任务，这样在执行同步数据任务的时候，不会降低读操作的性能。比如，如果从节点没有运行，此时 I/O 进程可以很快从主节点获取更新，尽管 SQL 进程还没有执行。如果在 SQL 进程执行之前从节点服务停止，至少 I/O 进程已经从主节点拉取到了最新的变更并且保存在本地 relay log 中，当服务再次起来之后就可以完成数据的同步。

要实施复制，首先必须打开 Master 端的 Binlog 功能，否则无法实现。

因为整个复制过程实际上就是 Slave 从 Master 端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。如下图所示：

![5](./.图片存放/007S8ZIlgy1gjx1mvsnp5j31040iyad2.jpg)

##### 复制的基本过程

1.  在从节点上执行 `sart slave` 命令开启主从复制开关，开始进行主从复制。从节点上的 I/O 进程连接主节点，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容。
2.  主节点接收到来自从节点的 I/O 请求后，通过负责复制的 I/O 进程（log Dump Thread）根据请求信息读取指定日志指定位置之后的日志信息，返回给从节点。返回信息中除了日志所包含的信息之外，还包括本次返回的信息的 Binlog file 以及 Binlog position（Binlog 下一个数据读取位置）。
3.  从节点的 I/O 进程接收到主节点发送过来的日志内容、日志文件及位置点后，将接收到的日志内容更新到本机的 relay log 文件（Mysql-relay-bin.xxx）的最末端，并将读取到的 Binlog文件名和位置保存到`master-info` 文件中，以便在下一次读取的时候能够清楚的告诉 Master ：“ 我需要从哪个 Binlog 的哪个位置开始往后的日志内容，请发给我”。
4.  Slave 的 SQL 线程检测到relay log 中新增加了内容后，会将 relay log 的内容解析成在能够执行 SQL 语句，然后在本数据库中按照解析出来的顺序执行，并在 `relay log.info` 中记录当前应用中继日志的文件名和位置点。

#### MySQL 基于 Binlog 主从复制的模式介绍[#](https://www.cnblogs.com/rickiyang/p/13856388.html#3226159864)

MySQL 主从复制默认是 **异步的模式**。MySQL增删改操作会全部记录在 Binlog 中，当 slave 节点连接 master 时，会主动从 master 处获取最新的 Binlog 文件。并把 Binlog 存储到本地的 relay log 中，然后去执行 relay log 的更新内容。

##### 异步模式 (async-mode)

异步模式如下图所示：

![6](./.图片存放/007S8ZIlgy1gjx1mu02wbj31160kk0vj.jpg)

这种模式下，主节点不会主动推送数据到从节点，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主节点如果崩溃掉了，此时主节点上已经提交的事务可能并没有传到从节点上，如果此时，强行将从提升为主，可能导致新主节点上的数据不完整。

##### 半同步模式(semi-sync)

介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到 relay log 中才返回成功信息给客户端（只能保证主库的 Binlog 至少传输到了一个从节点上），否则需要等待直到超时时间然后切换成异步模式再提交。

![7](./.图片存放/007S8ZIlgy1gjx1muypwdj31420jg41q.jpg)

相对于异步复制，半同步复制提高了数据的安全性，一定程度的保证了数据能成功备份到从库，同时它也造成了一定程度的延迟，但是比全同步模式延迟要低，这个延迟最少是一个 TCP/IP 往返的时间。所以，半同步复制最好在低延时的网络中使用。

半同步模式不是 MySQL 内置的，从 MySQL 5.5 开始集成，需要 master 和 slave 安装插件开启半同步模式。

##### 全同步模式

指当主库执行完一个事务，然后所有的从库都复制了该事务并成功执行完才返回成功信息给客户端。因为需要等待所有从库执行完该事务才能返回成功信息，所以全同步复制的性能必然会收到严重的影响。

#### Binlog 复制实战[#](https://www.cnblogs.com/rickiyang/p/13856388.html#3393628896)

配置 my.cnf

```sql
Copy[mysqld]
log-bin
server-id
gtid_mode=off #禁掉 gtid
```

添加主从复制用户：

```sql
Copygrant replication slave on *.* to 'repl'@'%' identified by 'gtidUser';
flush privileges;
```

然后我们新增一个从库。

接着我们用命令行的方式来加载主库的 Binlog 到从库，这里可以设置指定的 binlog 文件和位移值。在从库执行以下命令：

```sql
Copymysql>change master to
master_host='192.168.199.117',
master_user='slave',
master_port=7000,
master_password='slavepass',
master_log_file='mysql-bin.000008',
master_log_pos=0;

mysql>start slave;
mysql>show slave status\G;
```

复制过程中如果出现代码性错误，个人根据错误日志来决定是否要跳过错误继续执行：

```sql
Copymysql>stop slave;
mysql>set global sql_slave_skip_counter=1;
```

#### 主从复制可能会出现的问题[#](https://www.cnblogs.com/rickiyang/p/13856388.html#1737714396)

**Slave 同步延迟**

因为 Slave 端是通过 I/O thread 单线程来实现数据解析入库；而 Master 端写 Binlog 由于是顺序写效率很高，当主库的 TPS 很高的时候，必然 Master 端的写效率要高过 Slave 端的读效率，这时候就有同步延迟的问题。

I/O Thread 的同步是基于库的，即同步几个库就会开启几个 I/O Thread。

可以通过 `show slave status` 命令查看 `Seconds_Behind_Master` 的值来看是否出现同步延迟，这个值代表主从同步延迟的时间，值越大说明延迟越严重。值为 0 为正常情况，正值表示已经出现延迟，数字越大从库落后主库越多。

基于 Binlog 的复制方式肯定有这种问题，MySQL 官方也意识到，单线程不如多线程强，所以在 MySQL 5.7 版本引入了基于组提交的并行复制（官方称为Enhanced Multi-threaded Slaves，即MTS），设置参数：

`slave_parallel_workers>0` 即可，并且 `global.slave_parallel_type＝‘LOGICAL_CLOCK’`，

即可支持一个 schema(库) 下，`slave_parallel_workers`个 worker 线程并发执行 relay log 中主库提交的事务。

**其核心思想：**

一个组提交的事务都是可以并行回放（配合binary log group commit）；

slave 机器的 relay log 中 last_committed 相同的事务（sequence_num不同）可以并发执行。其中，变量 `slave-parallel-type` 可以有两个值：

1.  DATABASE 默认值，基于库的并行复制方式
2.  LOGICAL_CLOCK，基于组提交的并行复制方式

MySQL 5.7 开启 MTS 很简单，只需要在 Slave 从数据库的 my.cnf 文件中如下配置即可:

```sql
Copy# slave
 slave-parallel-type=LOGICAL_CLOCK
 slave-parallel-workers=8        #一般建议设置4-8，太多的线程会增加线程之间的同步开销
 master_info_repository=TABLE
 relay_log_info_repository=TABLE
 relay_log_recovery=ON
```

当然多线程带来的并行复制方案也有很多实现难点，比如事务都是有序执行的，如果并行回放会不会存在执行数据错乱的问题。这些问题就不在本节解释，大家有兴趣可以继续深究。

#### 新一代主从复制模式 - GTID 复制模式[#](https://www.cnblogs.com/rickiyang/p/13856388.html#1206294522)

在传统的复制里面，当发生故障，需要**主从切换**，需要找到 Binlog 和 位点信息，恢复完成数据之后将主节点指向新的主节点。在 MySQL 5.6 里面，提供了新的数据恢复思路，只需要知道主节点的 IP、端口以及账号密码就行，因为复制是自动的，MySQL 会通过内部机制 **GTID** 自动找点同步。

基于 GTID 的复制是 MySQL 5.6.5 后新增的复制方式。

**GTID (global transaction identifier)** 即全局事务 ID，一个事务对应一个 GTID，保证了在每个在主库上提交的事务在集群中有一个唯一的 ID。

##### GTID复制原理

在原来基于日志的复制中，从库需要告知主库要从哪个偏移量进行增量同步， 如果指定错误会造成数据的遗漏，从而造成数据的不一致。

而基于 GTID 的复制中，从库会告知主库已经执行的事务的 GTID 的值，然后主库会将所有未执行的事务的 GTID 的列表返回给从库，并且可以保证同一个事务只在指定的从库执行一次，**通过全局的事务 ID 确定从库要执行的事务的方式代替了以前需要用 Binlog 和 位点确定从库要执行的事务的方式**。

基于 GTID 的复制过程如下：

1.  master 更新数据时，会在事务前产生 GTID，一同记录到 Binlog 日志中。
2.  slave 端的 I/O 线程将变更的 Binlog，写入到本地的 relay log 中,读取值是根据`gitd_next变量`，告诉我们 slave 下一个执行哪个 GTID。
3.  SQL 线程从 relay log 中获取 GTID，然后对比 slave 端的 Binlog 是否有记录。如果有记录，说明该 GTID 的事务已经执行，slave 会忽略。
4.  如果没有记录，slave 就会从 relay log 中执行该 GTID 的事务，并记录到 Binlog。
5.  在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有二级索引就用全部扫描。

##### GTID 组成

 GTID = source_id:transaction_id

`source_id` 正常即是 `server_uuid`，在第一次启动时生成(函数 `generate_server_uuid`)，并持久化到 `DATADIR/auto.cnf` 文件里。

`transaction_id` 是`顺序化的序列号`(sequence number)，在每台 MySQL 服务器上都是从 1 开始自增长的序列，是事务的唯一标识。

##### GTID 生成

GTID 的生成受 `gtid_next` 控制。

在 Master 上，`gtid_next` 是默认的 `AUTOMATIC`,即 GTID 在每次事务提交时自动生成。它从当前已执行的 GTID 集合(即 gtid_executed)中，找一个大于 0 的未使用的最小值作为下个事务 GTID。在实际的更新事务记录之前将 GTID 写入到 Binlog。

在 Slave 上，从 Binlog 先读取到主库的 GTID(即 set gtid_next 记录)，而后执行的事务采用该 GTID。

##### GTID 的好处

1.  GTID 使用 `master_auto_position=1` 代替了 Binlog 的主从复制方案，相比 Binlog 方式更容易搭建主从复制。
2.  GTID 方便实现主从之间的 failover（主从切换），不用一步一步的去定位 Binlog日志文件和查找 Binlog 的位点信息。

##### GTID 模式复制局限性

1.  在一个事务里面混合使用引擎，如 Innodb(支持事务)、MyISAM(不支持事务)， 造成多个 GTIDs 和同一个事务相关联出错。
2.  `CREATE TABLE…..SELECT` 不能使用，该语句产生的两个 Event。 在某一情况会使用同一个 GTID(同一个 GTID 在 slave 只能被使用一次)：
    -   event one：创建表语句 create table
    -   event two ：插入数据语句 insert
3.  `CREATE TEMPORARY TABLE and DROP TEMPORARY TABLE` 不能在事务内使用 (启用了 `–enforce-gtid-consistency` 参数)。
4.  使用 GTID 复制从库跳过错误时，不支持 `sql_slave_skip_counter` 参数的语法。

##### GTID 主从复制实战

1.Master 主数据库上的操作

在 my.cnf 文件中配置 GTID 主从复制

```sql
Copy[root@mysql-master ~]# cp /etc/my.cnf /etc/my.cnf.bak
[root@mysql-master ~]# >/etc/my.cnf
[root@mysql-master ~]# cat /etc/my.cnf
[mysqld]
datadir = /var/lib/mysql
socket = /var/lib/mysql/mysql.sock
  
symbolic-links = 0
  
log-error = /var/log/mysqld.log
pid-file = /var/run/mysqld/mysqld.pid
  
#GTID:
server_id = 1
gtid_mode = on
enforce_gtid_consistency = on
    
#binlog
log_bin = mysql-bin
log-slave-updates = 1
binlog_format = row
sync-master-info = 1
sync_binlog = 1
   
#relay log
skip_slave_start = 1
```

配置后，重启 MySQL 服务：

```sql
Copy[root@mysql-master ~]# systemctl restart mysqld
```

登录 MySQL，并查看 Master 状态， 发现多了一项 `Executed_Gtid_Set`：

```sql
Copymysql> show master status;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
  
mysql> show global variables like '%uuid%';
+---------------+--------------------------------------+
| Variable_name | Value                                |
+---------------+--------------------------------------+
| server_uuid   | 317e2aad-1565-11e9-9c2e-005056ac6820 |
+---------------+--------------------------------------+
1 row in set (0.00 sec)
```

查看确认 GTID 功能打开:

```sql
Copymysql> show global variables like '%gtid%';
+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| binlog_gtid_simple_recovery      | ON    |
| enforce_gtid_consistency         | ON    |
| gtid_executed                    |       |
| gtid_executed_compression_period | 1000  |
| gtid_mode                        | ON    |
| gtid_owned                       |       |
| gtid_purged                      |       |
| session_track_gtids              | OFF   |
+----------------------------------+-------+
8 rows in set (0.00 sec)	
```

查看确认 Binlog 日志功能打开:

```sql
Copymysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

授权 slave 复制用户，并刷新权限:

```sql
Copy
mysql> flush privileges;
Query OK, 0 rows affected (0.04 sec)
  
mysql> show grants for slave@'172.23.3.66';
+-------------------------------------------------------------------------------+
| Grants for slave@172.23.3.66                                                |
+-------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'172.23.3.66' |
+-------------------------------------------------------------------------------+
1 row in set (0.00 sec)
  
```

再次查看 master 状态:

```sql
Copymysql> show master status;
+-------------------+----------+--------------+------------------+------------------------------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+-------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000001 |      622 |              |                  | 317e2aad-1565-11e9-9c2e-005056ac6820:1-2 |
+-------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
```

这里需要注意一下：
启动配置之前，同样需要对从服务器进行初始化。对从服务器初始化的方法基本和基于日志点是相同的，只不过在启动了 GTID 模式后，在备份中所记录的就不是备份时的二进制日志文件名和偏移量了，而是记录的是备份时最后的 GTID 值。
需要先在主数据库机器上把目标库备份一下，假设这里目标库是 slave_test：

```sql
Copymysql> CREATE DATABASE slave_test CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected (0.02 sec)
  
mysql> use slave_test;
Database changed
mysql> create table user (id int(10) PRIMARY KEY AUTO_INCREMENT,name varchar(50) NOT NULL);
Query OK, 0 rows affected (0.27 sec)
  
mysql> insert into slave_test.user values(1,"xiaoming"),(2,"xiaohong"),(3,"xiaolv");   
Query OK, 3 rows affected (0.06 sec)
Records: 3  Duplicates: 0  Warnings: 0
  
mysql> select * from slave_test.user;
+----+----------+
| id | name     |
+----+----------+
|  1 | xiaoming |
|  2 | xiaohong |
|  3 | xiaolv   |
+----+----------+
3 rows in set (0.00 sec)
```

把 slave_test 库备份出来:

```sql
Copy[root@mysql-master ~]# mysqldump --single-transaction --master-data=2 --triggers --routines --databases slave_test -uroot -p123456 > /root/user.sql
```

这里有个版本上的问题：

MySQL 5.6 使用 `mysqldump` 备份时，指定备份的具体库，使用 `--database`。

MySQL 5.7 使用 `mysqldump` 备份时，指定备份的具体库，使用`--databases。`

然后把备份的`/root/user.sql` 文件拷贝到 slave 从数据库服务器上。

```sql
Copy[root@mysql-master ~]# rsync -e "ssh -p20" -avpgolr /root/user.sql 
```

到这里主库的操作结束，包含 GTID 的备份数据已经 copy 到从库，下面来进行从库的操作。

2.从库操作

在 my.cnf 文件中配置 GTID 主从复制

与主服务器配置大概一致，除了 server_id 不一致外，从服务器还可以在配置文件里面添加`read_only＝on` ，使从服务器只能进行读取操作，此参数对超级用户无效，并且不会影响从服务器的复制。

```sql
Copy[root@mysql-slave1 ~]# >/etc/my.cnf
[root@mysql-slave1 ~]# vim /etc/my.cnf
[mysqld]
datadir = /var/lib/mysql
socket = /var/lib/mysql/mysql.sock
  
symbolic-links = 0
  
log-error = /var/log/mysqld.log
pid-file = /var/run/mysqld/mysqld.pid
  
#GTID:
server_id = 2
gtid_mode = on
enforce_gtid_consistency = on
    
#binlog
log_bin = mysql-bin
log-slave-updates = 1
binlog_format = row
sync-master-info = 1
sync_binlog = 1
   
#relay log
skip_slave_start = 1
read_only = on
```

配置完成后，重启mysql服务。

```sql
Copy[root@mysql-slave1 ~]# systemctl restart mysql
```

接着将主数据库目标库的备份数据 `user.sql`导入到从数据库里。

```sql
Copy[root@mysql-slave1 ~]# ls /root/user.sql
/root/user.sql
[root@mysql-slave1 ~]# mysql -p123456
.........
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
  
mysql> source /root/user.sql;
  
mysql> select * from slave.test;
+----+----------+
| id | name     |
+----+----------+
|  1 | xiaoming |
|  2 | xiaohong |
|  3 | xiaolv   |
+----+----------+
3 rows in set (0.00 sec)
```

在从数据库里，使用 `change master` 配置主从复制:

```sql
Copymysql> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)
  
mysql> change master to master_host='172.23.3.66',master_user='slave1',master_password='123456',master_auto_position=1;
Query OK, 0 rows affected, 2 warnings (0.26 sec)
  
mysql> start slave;
Query OK, 0 rows affected (0.02 sec)
  
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.23.3.66
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 1357
               Relay_Log_File: mysql-slave1-relay-bin.000002
                Relay_Log_Pos: 417
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
................
................
            Executed_Gtid_Set: 317e2aad-1565-11e9-9c2e-005056ac6820:1-5
                Auto_Position: 1
  
```

由此，Master 和 Slave 节点已经配置了主从同步关系。接下来你可以自行在主库插入一条数据观察从库是否同步过来。

##### 使用 GTID 添加从库有两种方式

**直接同步主库所有GTID**

如果主库一开始就开启了 GTID，那么可以直接获取主库的所有GTID来同步至从库。但是如果主库 Binlog 日志太多，那么相应同步的时间也会变长。这种方式适用于小数据量的同步。

使用这种方式来同步相应的命令为：

```sql
Copymysql>change master to master_host='xxxxxxx',master_user='xxxxxx',master_password='xxxxx',MASTER_AUTO_POSITION=1;
mysql> start slave;
mysql> stop slave io_thread; #重启 io 线程,刷新状态
mysql> start slave io_thread;
```

**当使用 `MASTER_AUTO_POSITION` 参数的时候，`MASTER_LOG_FILE`，`MASTER_LOG_POS` 参数不能使用。**
**如果想要从 `GTID 配置回 pos`，再次执行这条语句，不过把 MASTER_AUTO_POSITION 置为 0。**

**通过设定范围进行同步**

通过指定 GTID 的范围，然后通过在 slave 设置 `@@GLOBAL.GTID_PURGED` 从而跳过备份包含的 GTID 。

这种方案适用于数据量比较大一次同步需要耗费巨量时间的数据。但同时也有操作复杂的问题，需要你记住每次同步的范围。

用这种方式来同步相应的命令为：

```sql
Copymysql>change master to master_host='xxxxxxx',master_user='xxxxxx',master_password='xxxxx',MASTER_LOG_POS='xxxx';
mysql> start slave;
mysql> stop slave io_thread; #重启 io 线程,刷新状态
mysql> start slave io_thread;
```

这里注意到我们的参数换了：`MASTER_LOG_POS`，该参数表示当前需要同步的 GTID 事务的起点值。

#### 总结[#](https://www.cnblogs.com/rickiyang/p/13856388.html#2860264476)

本篇介绍主从复制的两种形式：基于 Binlog 和 位点信息的传统复制方式；基于 Binlog 和 GTID 的新式复制方式。现在很多公司可能还在使用 MySQL 5.6 的版本，所以 GTID不一定可以使用。

本文篇幅较长，大家可以简要查看原理。