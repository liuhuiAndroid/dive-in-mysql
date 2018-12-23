# 如何搭建并使用数据强一致性的MySQL集群
# 一、课程摘要
## 1. 为什么要使用PXC
Replication集群采用异步复制，无法保证数据的一致性
数据同步弱一致性的痛点：MySQL同步失败，导致读写分离出问题。会导致向集群写入了数据，读不到记录

PXC的数据强一致性，保证同步不会失败，在购物不会花冤枉钱
PXC集群没有主节点和从节点，任何节点都可以读写

## 2. 开发环境要求
内存：8GB 硬盘：100G
客户端：Navicat or DataGrip
MySQL 官方原版 免费 未来可能会闭源 高负载性能不好 兼容性好
MariaDB 社区版 免费 继续开源 高负载性能较好 兼容性一般
Percona 企业版 免费 继续开源 高负载性能最好 兼容性好
Vmware CentOS7 分配1GB内存 CPU1核 桥接模式
SSH客户端：XShell、MobaXterm(很强)

# 二、创建PXC集群
## 1. CentOS安装PerconaServer数据库
Percona Server只支持Linux系统，不能在其他系统上安装

本地安装Percona数据库
在线安装Percona数据库

## 2. 安装PXC组建集群

Percona XtraDB Cluster

PXC是基于Galera的面向OLTP的多主同步复制插件

PXC主要用于解决MySQL集群中数据同步强一致性问题

PXC是MySQL集群方案中公认的优选方案之一

#### PXC的特点

同步复制，事务在所有集群节点要么同时提交，要么不提交

多主复制，可以在任意一个节点写入

数据同步的强一致性，所有节点数据保持一致

#### 尽可能的控制PXC集群的规模

PXC集群节点越多，数据同步的速度就越慢

#### 所有PXC节点的硬件配置要相同

PXC集群数据同步的速度取决于配置最低的节点

#### PXC集群只支持InnoDB引擎

只有InnoDB引擎的数据才会被同步

#### 卸载mariadb-libs

CentOS捆绑了mariadb-libs，所以必须先卸载

创建PXC集群步骤见课程脚本

## 3. PXC集群的常用管理-数据库集群使用

#### PXC集群信息可以分为以下几类：

[队列 复制 流控 事务 状态](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/wsrep-status-index.html)

```mysql
show status like 'wsrep%';
show status like '%queue%';
```

wsrep_replicated：被其他节点复制的总数

wsrep_replicated_bytes：被其他节点复制的数据总数

wsrep_received：从其他节点处收到的写入请求总数

wsrep_received_bytes：从其他节点处收到的同步数据总数

wsrep_last_applied：同步应用次数

wsrep_last_committed：事务提交次数

#### 队列的相关信息

wsrep_local_send_queue：发送队列的长度

wsrep_local_send_queue_max：发送队列的最大长度

wsrep_local_send_queue_min：发送队列的最小长度

wsrep_local_send_queue_avg：发送队列的平均长度

wsrep_local_recv_queue：接收队列的长度

wsrep_local_recv_queue_max：接收队列的最大长度

wsrep_local_recv_queue_min：接收队列的最小长度

wsrep_local_recv_queue_avg：接收队列的平均长度

#### 流量控制的相关信息

添加空节点，全量同步数据太多，导致集群限速，结果很严重

wsrep_flow_control_paused_ns：流控暂停状态下花费的总时间（纳秒）

wsrep_flow_control_paused：流量控制暂停时间的占比（0~1）

wsrep_flow_control_sent：发送的流控暂停事件的数量

wsrep_flow_control_recv：接收的流控暂停事件的数量

wsrep_flow_control_interval：流量控制的下限和上限，上限是队列中允许的最大请求数。如果队列达到上限，则拒绝新的请求。当处理现有请求时，队列会减少，一旦达到下限，将再次允许新的请求

wsrep_flow_control_interval_low：流量控制的下限

wsrep_flow_control_interval_high：流量控制的上限

wsrep_flow_control_status：流量控制状态

## 4. PXC集群的常用管理-状态参数

#### PXC节点状态

OPEN PRIMARY JOINER JOINED SYNCED DONER 

#### PXC集群状态

PRIMARY DISCONNECTED NON_PRIMARY

#### 节点与集群的相关信息

wsrep_local_state_comment：节点状态

wsrep_cluster_status：集群状态

wsrep_connected：节点是否连接到集群

wsrep_ready：集群是否正常工作

wsrep_cluster_size：节点数量

wsrep_desync_count：延时节点数量

wsrep_incoming_addresses：集群节点IP地址

#### 事务的相关信息

wsrep_cert_deps_distance：事务执行并发数

wsrep_apply_oooe：接收队列中事务的占比

wsrep_apply_oool：接收队列中事务乱序执行的频率

wsrep_apply_window：接收队列中事务的平均数量

wsrep_commit_oooe：发送队列中事务的占比

wsrep_commit_oool：无任何意义（不存在本地乱序提交）

wsrep_commit_window：发送队列中事务的平均数量

## 5. PXC节点的上线与关闭

#### PXC节点的安全下线操作

节点怎么启动的，就使用对应的命令去关闭

#### PXC节点的意外下线操作

宕机、挂机、关机、重启、断电、断网都会让节点意外下线

#### PXC节点上线造作

如果PXC节点都是安全退出的，先要重启最后退出的节点

如果PXC节点都是意外退出的，我们需要修改配置文件，挑选一个节点作为主节点

如果集群中还有可运行的节点，其他节点按照普通节点上线即可

## 6. MySQL集群中间件比较

#### PXC集群需要负载均衡、读写分离、数据切分等功能，需要一个软件能实现这些功能

负载均衡中间件：Haproxy、MySQL-Proxy

负载均衡提供了请求转发，降低了单节点的负载

数据切分中间件：**MyCat**、Atlas、OneProxy、ProxySQL

按照不同的路由算法分发SQL语句就形成了数据切分

可以把一个集群当作一个分片，通过数据切分让多个集群共同管理数据

热数据保存到分片中，冷数据从分片中移除，加入归档库

书籍：《MyCat权威指南》《分布式数据库架构及企业实践—基于MyCat中间件》

这里使用**MyCat**来管理PXC集群

## 7. 配置MyCat负载均衡

1.  JDK安装与配置
2. 创建数据表
3. MyCat安装与配置

## 8. 数据切分和父子表

| 切分算法         |                     适用场合 |     备注     |
| ---------------- | ---------------------------: | :----------: |
| **主键求模切分** | 数据增长速度慢，难于增加分片 | 有明确主键值 |
| **枚举值切分**   | 归类存储数据，适合大多数业务 |              |
| 主键范围切分     |   数据快速增长，容易增加分片 | 有明确主键值 |
| 日期切分         |   数据快速增长，容易增加分片 |              |

求模切分适合用在初始数据很大，但是数据增长不快的场景

求模切分的弊病在于扩展新分片难度大，迁移的数据太多

求模切分建议扩张后的分片数时原有分片的2n倍

枚举值切分按照某个字段的值来切分数据，在rule.xml 中 sharding-by-intfile 配置

#### 表连接的难题

在MyCat中是不允许跨分片做表连接查询的

MyCat提出了父子表这种解决方案

## 10. 组建双机热备的MyCat集群-构建高可用的MyCat集群

MySQL要大量集群，MyCat要少量集群，Haproxy不需要集群

创建双机热备的MyCat集群

利用keepalived抢占虚拟IP

## 12. Sysbench基准测试-安装使用Sysbench

基准测试时针对系统的一种压力测试，但基准测试不关心业务逻辑，更加简单、直接、易于测试，不要求真实

QPS：每秒钟处理完成请求的次数

TPS：每秒钟执行完的事务次数

响应时间：一次请求所需要的平均处理时间

并发量：系统能同时处理的请求数

常见的基准测试工具：Mysqlslap、Sysbench、Jmeter

Sysbench可以进行：线程测试、CPU测试、内存测试、磁盘测试、数据库测试

安装Sysbench

执行测试

测试数据表建议不低于10个，单表数据量不低于500万行。如果是配备了SSD或者PCIE SSD的话，则建议单表数据量最少不低于1亿行

## 14. tpcc-mysql压力测试

tpcc-mysql是percona基于tpcc规范衍生出来的产品，专用于mysql压力测试

# 三、PXC集群原理

## 1. binlog日志

两种文件：日志文件、索引文件

日志文件：mysql_bin.000001、mysql_bin.000002

索引文件：mysql_bin.index

```mysql
[mysqld]
port=3306
binlog_format=row
log_bin=mysql_bin
```

阅读binlog文件

```sql
show master logs;
show binlog events in 'localhost-bin.000009';
```

三种模式：statement、row、mixed

row模式：每条记录的变化都会写到日志中

row模式优点：清晰的记录了每条记录的细节、数据同步安全可靠、同步时出现行锁的更少

row模式缺点：日志体积太大，浪费存储空间、数据同步频繁速度慢

statement模式：每条会修改数据的sql语句会记录到binlog中

statement模式优点：日志文件体积小、节省I/O和存储资源、集群节点同步速度快

statement模式缺点：某些函数和主键自增长会出现同步数据不一致

mixed模式：普通操作使用statement模式，同步会出现问题的操作选择row模式

PXC节点默认的日志模式：binlog_format=row

## 2. PXC同步原理

GTID（全局事务ID）= server_uuid + transaction_id

```sql
show status like "%uuid%";
wsrep_local_state_uuid 集群uuid
```

```sql
show binlog events in 'localhost-bin.000004';
xid 就是 transaction_id
```

查询最后提交的事务ID

```sql
show status like "%wsrep_last_committed%";
```

这里应该有一张PXC同步原理图... 

锁冲突案例演示，需要使用MyCat全局主键

replication集群master数据写入直接写入日志，slave定期读取日志执行，容易数据不一致

# 四、业务需求与MySQL架构设计

## 1. MySQL的5种特殊架构设计

## 2. 数据库设计原则

# 五、数据库常见业务处理

## 1. 向集群导入大量数据-了解Xtend基本语法

