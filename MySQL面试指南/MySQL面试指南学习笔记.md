# 二、MySQL版本类问题

## 1.版本类常见问题

问题1：你之前工作中使用的是什么版本的MySQL？为什么选择这个版本？

问题2：如何决定是否要为了MySQL进行升级？如何进行升级？

问题3：最新的MySQL版本是什么？它有什么特性比较吸引你？

## 2.为什么选择某一MySQL版本

#### 知识点

* MySQL常见的发行版

  MySQL官方版本

  Percona MySQL

  MariaDB

* 各个发行版本之间的区别和优缺点

  #### 基于服务器特性

  |           MySQL            |   Percona MySQL    |  MariaDB   |
  | :------------------------: | :----------------: | :--------: |
  |            开源            |        开源        |    开源    |
  |         支持分区表         |     支持分区表     | 支持分区表 |
  |           InnoDB           |       XtraDB       |   XtraDB   |
  | 企业版监控工具社区版不提供 | Percon Monitor工具 |   Monyog   |
  #### 基于高可用特性

  |     MySQL      | Percona MySQL  |             MariaDB              |
  | :------------: | :------------: | :------------------------------: |
  | 基于日志点复制 | 基于日志点复制 |          基于日志点复制          |
  |  基于Gtid复制  |  基于Gtid复制  | 基于Gtid复制,但Gtid同MySQL不兼容 |
  |      MGR       |    MGR&PXC     |          Galera Cluster          |
  |  MySQL Router  |   Proxy SQL    |             MaxScale             |
  #### 安全特性

  |                 MySQL                 |             Percona MySQL             |         MariaDB         |
  | :-----------------------------------: | :-----------------------------------: | :---------------------: |
  |             企业版防火墙              |           ProxySQL FireWall           |    MaxScale FireWall    |
  |            企业版用户审计             |               审计日志                |        审计日志         |
  |           用户密码生命周期            |           用户密码生命周期            |            -            |
  | sha256_password caching_sha2_password | sha256_password caching_sha2_password | ed25519 sha256_password |

  #### 开发及管理

  |      MySQL      |  Percona MySQL  |     MariaDB      |
  | :-------------: | :-------------: | :--------------: |
  | 窗口函数（8.0） | 窗口函数（8.0） | 窗口函数（10.2） |
  |        -        |        -        | 支持基于日志回滚 |
  |        -        |        -        |        -         |
  | Super read_only | Super read_only |        -         |

  问题1：你之前工作中使用的是什么版本的MySQL？为什么选择这个版本？

  当前公司是使用什么版本的MySQL，我们之前使用的是什么版本，版本之间有啥差异

## 3.如何对MySQL进行升级

#### 在对MySQL进行升级前要考虑什么？

1. 升级可以给业务带来的益处
   1. 是否可以解决业务上某一方面的痛点
      1. MySQL5.6升级到5.7，大幅度减小主从延迟时间
      2. MySQL8对Json部分复制的支持
   2. 是否可以解决运维上某一方面的痛点
2. 升级可能对业务带来的影响
   1. 对原业务程序的支持是否有影响
   2. 对原业务程序的性能是否有影响
3. 数据库升级方案的制定
   1. 评估受影响的业务系统
   2. 升级的详细步骤
   3. 升级后的数据库环境检查
   4. 升级后的业务检查
4. 升级失败的回滚方案
   1. 升级失败回滚的步骤
   2. 回滚后的业务检查

#### MySQL升级的步骤

1. 对待升级数据库进行备份
2. 升级Slave服务器版本
3. 手动进行主从切换
4. 升级Master服务器版本
5. 升级完成后进行业务检查

问题2：如何决定是否要为了MySQL进行升级？如何进行升级？

不能为了升级而升级，对数据库的升级一定要给我们的业务或者管理带来一些有利的好处

## 4.最新的MySQL版本及其新特性

#### MySQL8.0版本主要的新特性

|             服务器功能新特性             |
| :--------------------------------------: |
| 所有元数据使用InnoDB引擎存储，无frm文件  |
|   系统表采用InnoDB存储并采用独立表空间   |
| 支持定义资源管理组（目前仅支持CPU资源）  |
| 支持不可见索引和降序索引，支持直方图优化 |
|               支持窗口函数               |
|        支持在线修改全局参数持久化        |

|           InnoDB新特性            |
| :-------------------------------: |
|    InnoDB DDL语句支持原子操作     |
|      支持在线修改UNDO表空间       |
| 新增管理视图用于监控InnoDB表状态  |
| 新增Innodb_dedicated_server配置项 |

问题3：最新的MySQL版本是什么？它有什么特性比较吸引你？

# 第3章 用户管理类问题

## 1.用户管理常见问题

问题1：如何在给定场景下为某用户授权？

问题2：如何保证数据库账号的安全？

问题3：如何从一个实例迁移数据库账号到另一个实例？

## 2.给定场景下对用户授权

#### 知识点

* 如何定义MySQL数据库账号？

  用户名@可访问控制列表

  可访问控制列表：

   	1. %：代表可以从所有外部主机访问（默认）
   	2. 192.168.1.%：表示可以从192.168.1网段访问
   	3. localhost：DB服务器本地访问

  使用CREATE USER命令建立用户

* MySQL常用的用户权限

  1. Admin权限
     1. Create User 建立新的用户的权限
     2. Grant option 为其他用户授权的权限
     3. Super 管理服务器的权限
  2. DDL权限
     1. Create 新建数据库，表的权限
     2. Alter 修改表结构的权限
     3. Drop 删除数据库和表的权限
     4. Index 建立和删除索引的权限
  3. DML权限
     1. Select 查询表中数据的权限
     2. Insert 向表中插入数据的权限
     3. Update 更新表中数据的权限
     4. Delete 删除表中数据的权限
     5. Execute 执行存储过程的权限

* 如何为用户授权？

  * 遵循最小权限原则
  * 使用Grant命令对用户授权

  ```mysql
  show privileg; # 获取当前数据库所支持的数据库权限列表
  
  grant select,insert,update,delete on db.tb to user@ip; # 授权
  revoke delete on db.tb from user@ip; # 收回权限
  ```

## 3.保证数据库账号安全

#### 知识点

* 数据库用户管理流程规范

  * 最小权限原则

  * 密码强度策略

  * 密码过期原则

  * 限制历史密码重用原则

    ```mysql
    # mysql 8.0 密码过期原则 和 限制历史密码重用原则
    \h create user # 查看语法
    create user test@'localhost' identified by '123#qwe' password history 1; # 在设置新密码的时候不能和之前的密码相同
    select * from mysql.user where user='test';
        mysql -utest -p # （新连接）登录
        show databases; # （新连接）没有权限 看不到任务数据库
    alter user test@'localhost' password expire; # 设置密码过期
        mysql -utest -p # （新连接）登录
        show databases; # （新连接）提示需要重新设置密码
    	alter user user() identified by '123#qwe';  #（新连接）提示不能使用重复密码
    	alter user user() identified by '123#qwe*'; #（新连接）重置成功
    ```

* 密码管理策略

## 4.迁移数据库账号

#### 解决思路

数据库版本是否一致？

​	如果一致，则备份mysql库，然后在目的实例恢复

​	如果不一致，导出授权语句，在目的实例执行

#### 导出用户建立及授权语句

```mysql
pt-show-grants u=root,p=123456,h=localhost # Percona 的工具
```



