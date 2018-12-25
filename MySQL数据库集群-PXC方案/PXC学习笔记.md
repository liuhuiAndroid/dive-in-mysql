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

1. MySQL+分布式Proxy扩展

   pxc集群

   replication集群

   pxc+replication集群

2. 数据归档，冷热数据分离

   mysql数据归档到MongoDB（会丢）或者TokuDB（不会丢，带事务 ）

3. MySQL+缓存（Redis）高并发架构

4. MySQL+小文件系统

   用数据库保存二进制数据

   用数据库保存文件路径

   hdfs文件系统

   mongodb gridfs文件系统

5. MySQL+Inforbright统计分析架构

## 2. 数据库设计原则

依据业务系统对数据的操作分类：只读业务系统、可读写系统

常见业务表分类：

1. 配置表：存放配置信息和字典信息
2. 状态表：业务实体的状态信息
3. 日志表：业务实体的状态变化记录
4. 归档表：业务实体的冷数据
5. 统计数据表：OLAP中的业务数据
6. 统计结果表：OLAP中的统计结果

如何避免“过度设计”和“过早优化”

单体单节点阶段 -> 单体集群阶段 -> 拆分业务表 -> 微服务分布式架构

# 五、数据库常见业务处理

## 1. 向集群导入大量数据

#### 了解Xtend基本语法

- 单节点导入

source命令导入，比较慢

```sql
source test.sql
source all.sql
```

 load data命令导入，单线程，最佳方式

```sql
load data local infile 'data.txt' info table orders fields terminated by ',' optionally enclosed by '' lines terminated by '\n'
```

- MySQL集群数据导入

  利用程序生成数据

  安装 Xtend插件，导入Xtend运行库 

#### 准备向集群导入的数据

生成1000万条数据

数据切分

```shell
split -l 1000000 -d data.txt
```

#### 将数据导入到PXC集群

修改PXC配置文件

两个PXC集群都只开启一个节点，防止数据同步导致限流

创建t_test数据表

配置MyCat

执行Java程序，多线程导入数据

#### 执行与总结

Java程序打包为runnable jar file

```shell
java -jar demo.jar
```

```mysql
use test;
select count(*) from t_test;
```

数据导入之后的后续工作：

1. 关闭PXC节点（还原配置文件）

   ```shell
   systemctl stop mysql@bootstrap.service
   vi /etc/my.cnf
   ```

2. 拷贝数据文件到其他PXC节点

3. 关闭MyCat（还原配置文件）

4. 启动PXC和MyCat

总结：

1. 使用load data命令导入数据
2. 多线程，速度快
3. 单节点PXC，避免限流

## 5. 分页查询优化

常用的分页查询SQL

```mysql
select * from t_test limit 100,100;
select * from t_test limit 10000,100;
select * from t_test limit 1000000,100;
select * from t_test limit 5000000,100;
```

分析结论：全表扫描，速度极慢

1. limit语句的查询时间与起始记录的位置成正比
2. mysql的limit语句是很方便，但是对记录很多的表并不适合直接使用

优化办法：

1. 利用主键索引来加速分页查询

   explain 关键字可以分析sql执行

   ```mysql
   select * from t_test where id > 5000000 limit 100;
   select * from t_test where id > 5000000 and id <= 5000000 + 100;
   
   explain select * from t_test limit 5000000,100;
   explain select * from t_test where id > 5000000 limit 100;
   explain select * from t_test where id > 5000000 and id <= 5000000 + 100;
   ```

2. 如果主键值不连续，怎么分页

   1. 使用逻辑删除，不会造成主键不连续

   2. 利用主键索引加速，再做表连接查询

      ```mysql
      select t.* from t_test t inner join (select id from t_test limit 5000000,100) tmp on t.id = tmp.id
      ```

3. 其他解决办法

   业务上限定不可以查询早期数据

## 6. 高并发引起的重复写入

产生重复写入的案例：提交 按钮重复点击，数据库插入重复数据

前端解决方案：按钮禁用

Redis缓存计数器方案：5秒内只能执行一次，注意计数器加锁

Redis缓存计数器的方案没有彻底解决了重复写入的问题，只解决了单位时间内的重复写入，单位时间外的重复写入还是不能阻止

Token方案

## 7. 高并发访问优化

数据库设计核心原则：

1. 不在数据库做运算
2. CPU计算务必转移至业务层
3. 控制字段数量（少而精，注意拆分）
4. 平衡范式与冗余（效率优先，可以牺牲范式）
5. 拒绝大sql语句、拒绝大事务、拒绝大批量

字段设计原则：

1. 用恰当的数据类型
2. 字符转化为数字（节约空间，提高查询性能）
3. 避免使用NULL字段（NULL很难查询优化、索引需要额外空间，而且复合索引无效）
4. 少用text类型（尽量使用varchar代替text字段）

索引设计原则：

1. 合理使用索引（改善查询，减慢更新）
2. 长字符串字段必须建前缀索引
3. 不在索引做列运算
4. 不用外键（使用逻辑外键）

SQL设计原则：

1. SQL语句尽可能简单
2. 简单的事务
3. 避免使用触发器和存储过程
4. OR改写成IN
5. OR改写成UNION

减少不必要的SQL

```mysql
insert ignore into t_user set username = ... and password = hex(aes_encrypt(...,...));
#如果插入的数据违反了主键约束或者唯一性约束，则放弃数据写入，返回值为0，不会报错
```

防止MySQL集群主键自增长的弊病

1. MyCat全局主键，使用 zookeeper集群保存状态信息
2. uuid主键不适合MyCat求模切分

性能不行，缓存来凑

1. 最好把缓存业务独立出来
2. 可以使用阿里巴巴Canal中间件

## 8. 大数据归档-冷热数据分离

InnoDB写入速度比较慢，数据归档瞬时写入压力对于InnoDB引擎比较大，可以使用TokuDB引擎

TokuDB引擎：

1. 高压缩比，高写入性能
2. 在线创建索引和字段
3. 支持事务
4. 支持主从同步

安装TokuDB

使用TokuDB引擎建表

```mysql
create table student(
	...
)engine = TokuDB;
```

归档库的双机热备

节省硬件资源PXC集群都只启动一个节点，

MyCat一个节点向虚拟IP发送归档数据，注意修改MyCat配置

###### docker虚拟机方案！！！虚拟机实例共享使用硬件资源

## 9. 大数据归档-搭建Replication集群

A节点和B节点互为主从关系，实现双向同步

在两个TokuDB数据库上创建用户

修改两个TokuDB的配置文件，开启bin_log日志和relay_log日志

重新启动两个TokuDB节点

配置主从同步：分别在两个TokuDB上执行4句SQL

创建归档表

配置Haproxy+Keepalived双机热备

## 10. 大数据归档-执行大数据归档

启动一个MyCat节点，两个归档库节点，以及双机热配程序

#### 准备归档数据

在两个PXC分片上创建进货表，采用InnoDB引擎

配置MyCat的schema.xml文件，并重启MyCat，让MyCat接管进货表

#### 执行数据归档

编写Java程序向MyCat写入十万条数据

安装pt-archiver，数据归档工具

pt-archiver的用途：

1. 导入线上数据，到线下数据作处理
2. 清理过期数据，并把数据归档到本地归档表中，或者远程归档服务器

执行数据归档，自带事务，归档失败数据不会删除

#### 总结

1. 使用TokuDB引擎保存归档数据，拥有高速写入特性
2. 使用双机热备方案搭建归档库，具备高可用性
3. 使用pt-archiver执行数据归档，简单易行
4. 编写Java或Python自动化定时归档程序，归档，发送邮件通知管理员

## 11. 数据分区-认识表分区

#### 什么是表分区

#### 表分区有什么好处？

1. 表分区的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备
2. 单表可以存储更多的数据
3. 数据写入和读取的效率提升了，汇总函数计算速度变快了
4. 不会出现表锁，只会锁住相关分区
5. 数据分区不是无限的，每张数据表最多可以有1024个分区

#### 表分区有什么缺点？

1. 不支持存储过程、存储函数和某些特殊函数
2. 不支持按位运算符
3. 分区键不能子查询
4. 创建分区后，尽量不要修改数据库模式

#### 为什么要在集群中引入表分区？

表分区比分片的成本低很多，当热数据太多的情况下，优先考虑使用表分区

#### 挂载硬盘

可以在A PXC节点和B PXC节点都挂载3块2G SCSI硬盘

```shell
fdisk -l # 查看主机硬盘信息
fdisk /dev/sdb # 输入n创建新分区
n：创建新分区
d：删除分区
p：列出分区表
w：把分区表写入硬盘并退出
q：退出而不保存

mkfs -t ext4 /dev/sdb1 # 格式化分区
vi /etc/fstab # 永久挂载
/dev/sdb1 /mnt/p0 ext4 defaults 0 0

reboot # 重启生效
cd /mnt/p0
mkdir data
chown -R mysql:mysql /mnt/p0/data # 给数据目录分配用户
```

#### PXC节点使用表分区

PXC同步模式只能是PERMISSIVE或者DISABLED，修改my.cnf

```mysql
pxc_strict_mode = PERMISSIVE | DISABLED
```

#### 表分区类型

RANGE分区：根据连续区间值切分数据

LIST分区：根据枚举值切分数据

HASH分区：对整数求模切分数据

KEY分区：对任何数据类型求模切分数据

## 12. 数据分区-Range分区

主键范围切分，很少使用

```mysql
create table t_range_1(
	id int unsigned primary key,
	name varchar(200) not null
)
pritition by range(id)(
	partition p0 values less than(10000000),
	partition p1 values less than(20000000),
	partition p2 values less than(30000000),
	partition p3 values less than(40000000)
)
```

按日期范围切分

```mysql
create table t_range_2(
	id int unsigned ,
	name varchar(200) not null,
    birthday date not null,
    primary key(id,birthday) # 复合主键
)
pritition by range(month(birthday))(
	partition p0 values less than(3),
	partition p1 values less than(6),
	partition p2 values less than(9),
	partition p3 values less than(12)
)
```

把分区映射到特定的硬盘

```mysql
create table t_range_2(
	id int unsigned ,
	name varchar(200) not null,
    birthday date not null,
    primary key(id,birthday) # 复合主键
)
pritition by range(month(birthday))(
	partition p0 values less than(6) data directory = "/mnt/p0/data",
	partition p1 values less than(12) data directory = "/mnt/p1/data"
)
```

```shell
cd /mnt/p0/data
cd test
ls
cd /mnt/p1/data
cd test
ls
```

查看每个分区保存了哪些数据

```mysql
select partition_name,partition_method,partition_expression,partition_description,
table_rows,subpartition_name,subpartition_method,subpartition_expression
from infomation_schema.partitions
where table_schema = schema()
and table_name = 't_range_2';
```

配置mycat的schema.xml中的t_range_2的配置

测试mycat和表分区完美结合在一起

## 13. 数据分区-LIST分区

```mysql
create table t_list_1(
	id int unsigned,
	name varchar(200) not null,
	province_id int unsigned not null,
	primary key(id,province_id) # 复合主键
)
pritition by list(province_id)(
	partition p0 values in(1,2,3,4) data directory = "/mnt/p0/data",
	partition p1 values in(5,6,7,8) data directory = "/mnt/p1/data"
)
```

配置mycat的schema.xml中的t_list_1的配置

```mysql
<function name="provice-hash-int"
	class="io.mycat.route.function.PartitionByFileMap">
	<property name="mapFile">province-hash-int.txt</property>
</function>

<tableRule name="sharding-province">
	<rule>
		<columns>province_id</columns>
		<algorithm>province_hash-int</algorithm>
	</rule>
</tableRule>

<table name="t_list_1" dataNode="dn1,dn2" rule="sharding-province"/>

vi province_hash-int.txt
1=0
2=0
...
8=0
9=1
...
16=1
```

表分区的切分规则与MyCat切分规则可以没有关系

## 14. 数据分区-Hash分区

hash分区：整数、或者日期、时间戳、字符串、二进制类型运算结果是整数

macat主键求模：整数

```mysql
create table t_hash_1(
	id int unsigned primary key,
	name varchar(200) not null,
	province_id int unsigned not null
)
pritition by hash(id) partitions 2(
	partition p0 data directory = "/mnt/p0/data",
	partition p1 data directory = "/mnt/p1/data"
)
```

```mysql
<table name="t_hash_1" dataNode="dn1,dn2" rule="sharding-province"/>
```

查询分区数据

```mysql
select partition_name,table_rows
from information_schema.partitions
where table_schema=schema() and table_name="t_hash_1";
```

还可以试验一下非整数字段hash分区

```mysql
create table t_hash_2(
	id int not null,
	name varchar(200) not null,
    hiredate date not null,
	primary key(id,hiredate)
)
pritition by hash(year(hiredate)) partitions 2(
	partition p0 data directory = "/mnt/p0/data",
	partition p1 data directory = "/mnt/p1/data"
)
```

```mysql
<table name="t_hash_2" dataNode="dn1,dn2" rule="sharding-province"/>
```

hash分区可以对整数，或者函数运算结果是整数的字段做数据切分，保证数据均匀分布

## 15. 数据分区-Key分区

hash分区：整数、或者日期、时间戳、字符串、二进制类型运算结果是整数

Key分区：整数、或者日期、时间戳、字符串、二进制，是hash分区的加强版

```mysql
create table t_key_1(
	id int not null,
	name varchar(200) not null,
    job varchar(200) not null,
	primary key(id,job)
)
pritition by key(job) partitions 2(
	partition p0 data directory = "/mnt/p0/data",
	partition p1 data directory = "/mnt/p1/data"
)
```

```mysql
<table name="t_key_1" dataNode="dn1,dn2" rule="mod-long"/>
```

查询分区数据

```mysql
select partition_name,table_rows
from information_schema.partitions
where table_schema=schema() and table_name="t_key_1";
```

Key分区支持任何数据类型

Key分区创建的时候可以不指定字段，mysql会默认用主键字段（单一主键）

## 16. 数据分区-管理Range表分区

```mysql
pritition by range(id)(
	partition p0 values less than(10000000),
	partition p1 values less than(20000000),
	partition p2 values less than(30000000),
	partition p3 values less than(40000000)
)

# 添加新分区
alter table t_range_1 add partition (
	partition p4 values less than(50000000)
)
```

```mysql
# 在已挂载的硬盘创建2个目录充当2块硬盘
mkdir -p /home/p0/data
mkdir -p /home/p1/data
mkdir -p /home/p2/data
mkdir -p /home/p3/data
mkdir -p /home/p4/data
# 给数据目录分配用户
chown -R mysql:mysql /home/p0/data 
chown -R mysql:mysql /home/p1/data
chown -R mysql:mysql /home/p2/data
chown -R mysql:mysql /home/p3/data
chown -R mysql:mysql /home/p4/data
```

```mysql
create table t_range_1(
	id int unsigned primary key,
	name varchar(200) not null,
    job varchar(200) not null
)
pritition by range(id)(
	partition p0 values less than(10000000) data directory = "/home/p0/data",
	partition p1 values less than(20000000) data directory = "/home/p1/data",
	partition p2 values less than(30000000) data directory = "/home/p2/data",
	partition p3 values less than(40000000) data directory = "/home/p3/data"
)

# 添加新分区
alter table t_range_1 add partition (
	partition p4 values less than(50000000) data directory = "/home/p4/data"
)

# 删除表分区,数据会丢失
alter table t_range_1 drop partition p3,p4;

# 表分区拆分,数据不会丢失
alter table t_range_1 reorganize partition p0 into (
	partition s0 values less than(5000000) data directory = "/home/p3/data",
	partition s1 values less than(10000000) data directory = "/home/p4/data",
);

# 表分区合并,数据不会丢失
alter table t_range_1 reorganize partition s0,s1 into (
	partition p0 values less than(10000000) data directory = "/home/p0/data"
);

# 移除所有表分区,数据不会丢失
alter table t_range_1 remove partitioning;
```

## 17. 数据分区总结

#### 子分区

子分区就是在已有的分区上再创建分区切分数据

目前只有RANGE和LIST分区可以创建自分区，而且自分区只能是HASH或者KEY分区

```mysql
create table t_range_3(
	id int unsigned not null,
	name varchar(200) not null,
    province_id int unsigned not null,
	primary key(id,province_id,name)
)
pritition by range(province_id) subpartition by key(name) subpartitions 4(
	partition p0 values less than(10) data directory = "/home/p0/data",
	partition p1 values less than(20) data directory = "/home/p1/data",
    partition p2 values less than maxvalue data directory = "/home/p2/data"
)
```

#### 合理的利用表分区与集群分片，降低数据库集群的使用成本

# 六、数据备份与恢复

## 1. 数据库的冷备份与热备份

#### 数据导出不等于数据备份

#### 何时用导出，何时用备份？

1. 数据导出用于把数据从一个系统迁移到另一个系统
2. 数据备份用于保存一个数据库实例的全部信息

#### 什么是冷备份？

在数据库已经关闭的情况下，对数据的备份称作冷备份

把数据文件、结构文件、索引文件、日志文件压缩成zip文件保存

#### 冷备份存在的问题

1. 数据库必须停机备份
2. 备份的文件非常占用存储空间的，不支持增量备份
3. 冷备份的是所有的数据文件和日志文件，所以无法按照逻辑库和数据表恢复数据

#### 联机冷备份

#### 什么是热备份？

在数据库节点不停机的状态下执行的备份被称作热备份

#### 热备份的缺点

数据库备份的时候会全局加读锁，备份期间节点只能读取数据不能写入数据

#### 联机热备份

#### 联机热备份与联机冷备份该如何选择？

既然在联机状态下备份节点数据不影响业务系统，两种备份的内容都差不多，应该如何选择用哪种备份呢？

## 2. 联机冷备份-数据表碎片整理

```mysql
cd /var/lib/mysql
ls
cd test 
ls 
```

#### auto.cnf文件

每个MySQL都有唯一的UUID，这个值被保存到了auto.cnf文件

数据还原的时候，注意auto.cnf文件

#### grastate.dat文件

grastate.dat文件里保存的是PXC的同步信息

#### gvwstate.dat文件

gvwstate.dat文件里保存的是PXC集群节点的信息

#### 其他文件

err				错误日志文件

pid				进程id文件

ib_buffer_pool	InnoDB缓存文件

ib_logfile		InnoDB事务文件

ibdata			InnoDB共享表空间文件

logbin			日志文件

index			日志索引文件

ibtmp			临时表空间文件

#### 数据文件中的碎片是什么？

向数据表写入数据，数据文件的体积会增大，但是删除数据的时候数据文件体积并不会减小，数据被删除后留下的空白，被称作碎片

数据备份前需要执行碎片整理

#### 整理碎片文件

```mysql
alter table student engine=InnoDB; # 整理碎片，会锁表
```

```shell
vi /etc/my.cnf # 防止碎片整理记录到日志，导致整个集群都进行碎片整理
#log_bin
#log_slave_updates
```

## 3. 联机冷备份-冷备份PXC节点

```shell
systemctl stop mysql@bootstrap.service
vi /etc/my.cnf
#log_bin
#log_slave_updates
#wsrep_... 同步参数也注释掉
service start mysql
# 执行碎片整理的Java程序
service stop mysql # 冷备份需要停止数据库
```

```shell
# 冷备份数据目录
tar -cvf mysql.tar /var/lib/mysql
# 冷备份表分区
tar -cvf p0.tar /mnt/p0/data
tar -cvf p1.tar /mnt/p1/data
...
```

#### 冷备份后，节点上线

恢复my.cnf文件中，有关binlog日志，以及PXC同步的相关配置

重新启动PXC节点，加入到PXC集群中

```shell
vi /etc/my.cnf
systemctl start mysql@bootstrap.service
```

## 4. 联机冷备份-冷还原

#### 还原前的准备工作

如果备份节点存在表分区，那么还原节点必须创建相同的表分区存储空间，并删除MySQL数据目录

停止MySQL服务

```shell
rm -rf /var/lib/mysql
```

#### 执行还原

```shell
tar -xvf mysql.tar
mv -f var/lib/mysql /var/lib
rm -rf var
...

rm /var/lib/mysql/auto.cnf # 删除 mysql uuid 
vi /var/lib/mysql/grastate.dat
safe_to_bootstrap: 0 # 设置为0
service start mysql
```

#### 冷备份的实际用途

利用冷备份的备份文档，还原到新上线的PXC节点，让节点具有初始数据，避免上线后出现全量同步

可以定期执行冷备份

## 5. XtraBackup热备份原理

## 6. 全量热备份-常见命令

## 7. 全量热备份-编写shell脚本

## 8. 全量冷恢复

## 9. 增量热备份-注意事项

## 10. 增量热备份-Cron表达式语法

## 11. java程序定时增量热备份数据库

## 12. 增量冷还原

## 13. 误操作恢复_延时节点解决方案

## 14. 误操作恢复_恢复主节点误删除故障

## 15. 误操作恢复_日志闪回方案

# 备注

Typora 工具很方便编辑MD格式的文件
