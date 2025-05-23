# 1. 架构和历史
## 1.1 逻辑架构
![[读书 - 高性能 MySQL.png]]

第一层是大多数基于网络层的客户端服务器工具
第二层是 MySQL 的核心服务功能
第三层是存储引擎

## 1.2 并发控制
### 1.2.1 读写锁

概念：
**读锁**是共享的，相互不阻塞，同一时刻可以同时读取同一个资源互不干扰
**写锁**是排他的，写锁会阻塞其他的写锁和读锁

### 1.2.2 锁粒度

如果能做到修改哪部分锁定哪部分，会提高并发性

加锁粒度消耗资源，所以要找到锁粒度与安全性的平衡。MySQL有两个重要锁策略：

- **表锁**
最基本锁策略，锁定整张表，特定场景也会有良好性能

- **行级锁**
最大程度支持并发处理。

## 1.3 事务

如果数据库引擎能够成功地对数据库应用该组查询的全部语句，那么就执行该组查询。如果其中有任何一条语句因为崩溃或其他原因无法执行，那么所有的语句都不会执行。也就是说，事务内的语句，要么全部执行成功，要么全部执行失败。

ACID 表示原子性(atomicity)、一致性(consistency)、隔离性(isolation)和持久性(durability)

**原子性**
一个事务一定是最小的不可分割工作单元，整个事务要么全部成功要么全部回滚

**一致性**
数据从一个状态切换到另一个状态，不会因为系统崩溃等愿意导致数据被修改

**隔离性**
当一个事务的修改完成之前，他的修改对其他的事务通常是不可见的

**持久性**
当事务提交时，所有的修改就会保存在数据库中

### 1.3.1 隔离级别

四种隔离级别，低级别通常可以执行高并发

**读未提交** READ UNCOMMITTED
在此级别，事务的修改，即使没有提交，对其他事务也是可见的。
事务可以读取未提交的事务，称为**脏读**
实际中很少使用，因为其性能并没有比其他级别好很多

**提交读** READ COMMITTED 
大多数数据库默认的隔离级别（MySQL 不是）
满足隔离性的简单定义，一个事务开始时，只能看见已提交事务的修改。

**可重复读** REPEATABLE READ 
解决了脏读问题，保证同一个事务多次读取记录的结果一致
无法解决**幻读**问题：当某个事物在读取某个范围记录时，另一个事务又在该范围内插入了新的记录，当之前的事务再读取该范围的纪录时，会产生 **幻行**

**可串行化** SERIALIZABLE
最高隔离级别。
强制事务串行执行，避免幻读问题
就是会在读取的每一行数据都加上锁，可能会导致超时和锁争用问题
很少使用这个隔离级别，通常在非常需要一致性以及可以介绍没有并发的情况下使用

### 1.3.2 死锁

两个事务或多个事务在同一资源上相互占用，并且请求锁定对方占用的资源，造成恶性循环。

例子
![[读书 - 高性能 MySQL-1.png]]

InnoDB 目前处理思索的方法就是将持有最少行级排它锁的事务进行回滚。

### 1.3.3 事务日志

主要目的是为提高事务效率。

修改表数据实际上是在修改表数据在内存的拷贝，然后再将修改行为记录到硬盘上的事务日志中，不用每次修改数据都记录在物理磁盘中。

事务日志采用 **追加方式** 

事务日志持久化之后，内存中被修改的数据可以慢慢写入磁盘。

通常称之为预写试日志

### 1.3.4　MySQL中的事务

MySQL 有两种事务性存储引擎 ： InnoDB 和 NDB Cluster

#### **自动提交** AUTOCOMMIT
MySQL 默认是自动提交模式，即如果不显式开启一个事物，则每个查询都当做一个事务操作
可以通过 
![[读书 - 高性能 MySQL-2.png]]

#### **混合使用存储引擎**
在事务中混合使用 事务型 和 非事务型表 (例如InnoDB和MyISAM表）​，在正常提交的情况下不会有什么问题。但如果该事务需要回滚，非事务型的表上的变更就无法撤销，这会导致数据库处于不一致的状态，这种情况很难修复，事务的最终结果将无法确定。

## 多版本并发控制

MySQL 大多数事务型存储引擎都同时实现了多版本并发控制(MVCC)

多版本并发控制(MVCC)：是行级锁的一个变种，很多情况下避免了加锁操作，开销更低

实现：通过保存数据在某个时间点的快照来实现的。不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务开始的时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。

#### InnoDB 的 MVCC
通过每行记录保存两个隐藏的行实现，一个保存行的创建时间，一个保存行的过期时间。
存储系统版本号(system version number)。每开始一个新的事务，系统版本号都会自动递增。
事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较

##### 在REPEATABLE READ隔离级别下的 MVCC 操作

**SELECT**

根据以下两个条件查询每行记录

1. InnoDB 只查找版本号早于当前事务版本数据行（行的系统版本号小于等于事务的系统版本号）确保事务读取的行，要么是事务开始就存在，要么是事务自身插入的

2. 行的删除版本要么未定义，要么大于当前事务版本号。确保读取到的行在事务开始之前未被删除

符合上述两者才能作为返回结果

**INSERT**

InnoDB 为新插入的每一行保存当前系统版本号作为行版本号。

**DELETE**
InnoDB 为删除的每一行保存当前系统版本号作为删除标识

**UPDATE**
InnoDB 插入一行新纪录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来行作为行的删除标识。

保存这两个额外的系统版本号，可以使大多数读操作都不需要加锁。

MVCC 只在 REPEATABLE READ 和 READ COMMITTED 两个隔离级别下工作。


# 2. MySQL 基准测试

是针对系统设计的一种压力测试