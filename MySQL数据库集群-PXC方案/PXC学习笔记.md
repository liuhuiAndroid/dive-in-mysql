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
## 4. PXC集群的常用管理-状态参数
## 5. PXC节点的上线与关闭
## 6. MySQL集群中间件比较
## 7. 配置MyCat负载均衡
## 8. 数据切分
## 9. 父子表
## 10. 组建双机热备的MyCat集群-构建高可用的MyCat集群
## 11. 组建双机热备的MyCat集群-利用keepalived抢占虚拟IP
## 12. Sysbench基准测试-安装Sysbench
## 13. Sysbench基准测试-使用Sysbench
## 14. tpcc-mysql压力测试