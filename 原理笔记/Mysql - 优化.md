# 如何定位慢查询

## 开源工具

Arthas、Prometheus、Skywalking
## MySQL 自带慢日志查询

慢日志查询记录了所有执行时间超过指定参数(long_query_time)的 SQL 语句
需要在 MySQL 的配置文件中进行配置 (/etc/my.cnf)

```json
slow_query_log=1 #1为开0为关
long_query_time=2 #秒
```
配置完成后重新启动，检测到慢查询就会记录到`/var/lib/mysql/localhost-slow.log`中

# 分析 SQL 语句执行慢的原因

## 获取 MySQL 执行 SELECT 语句的信息

可以使用 EXPLAIN 或者 DESC 命令获取 MySQL 执行 SELECT 语句的信息



# 未整理
## 事务四大特性
    
### 原子性(Atomicity)
 
事务是不可分割的最小操作单元要么全部成功/要么全部失败。
            
### 一致性(Consistency)

事务完成时，必须使所有的数据都保持一致状态。
### 隔离性(Isolation)

数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。

### 持久性(Durability)
 
 事务一旦提交或回滚/它对数据库中的数据的改变就是永久的。

## MySQL 中的分页

MySQL 中实现分页查询使用的是 `LIMIT` 和 `OFFSET` 关键字，而不是 `GROUP BY`。

**基本语法：**
```sql
-- 方式一
SELECT * FROM table_name LIMIT [每页记录数] OFFSET [跳过的记录数];

-- 方式二（简写形式）
SELECT * FROM table_name LIMIT [跳过的记录数], [每页记录数];
```

**示例：**
```sql
-- 获取第3页数据，每页10条（跳过前20条，取10条）
-- 方式一
SELECT * FROM users LIMIT 10 OFFSET 20;

-- 方式二
SELECT * FROM users LIMIT 20, 10;
```

`GROUP BY` 是用于分组聚合操作的关键字，与分页无关。

## 事务隔离级别

MySQL 支持的四种标准事务隔离级别：

1. **READ UNCOMMITTED（读未提交）**
   - 最低隔离级别
   - 问题：脏读、不可重复读、幻读都可能发生
   - 一个事务可以读取另一个未提交事务的数据

2. **READ COMMITTED（读已提交）**
   - 问题：不可重复读、幻读可能发生
   - 一个事务只能读取另一个已提交事务的数据
   - 解决了脏读问题

3. **REPEATABLE READ（可重复读）**
   - MySQL 的默认隔离级别
   - 问题：幻读可能发生
   - 在同一事务中多次读取同样的数据结果一致
   - 解决了脏读、不可重复读问题

4. **SERIALIZABLE（串行化）**
   - 最高隔离级别
   - 完全解决了脏读、不可重复读、幻读问题
   - 通过强制事务串行执行，性能最低

**相关问题解释：**
- **脏读**：读取了未提交的数据，对方回滚后数据变无效
- **不可重复读**：同一事务内多次读取，数据被其他事务更改
- **幻读**：同一事务内多次查询，返回的记录数不一致（其他事务插入新记录）

# 读书
# [[读书 - 《高性能 MySQL》]]
