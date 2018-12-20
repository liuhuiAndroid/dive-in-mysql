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
PXC是基于Galera的面向OLTP的多主同步复制插件

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