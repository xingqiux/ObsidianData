
Redis 读写是单线程的
# Redis 的数据类型

非关系的内存


# Redis 作为缓存
## 缓存穿透，缓存击穿，缓存雪崩都是什么如何解决？

### 缓存穿透
![[面试八股 - Redis-2.png|675]]
**定义**：缓存穿透是指查询到一个不存在的数据，同时也不会写入缓存，导致每次请求都会请求到数据库

#### **解决方法1 缓存空数据**：
当存储到一个不存在的数据时，通过缓存一个对应的空结果缓存。
缺点：对缓存的空间消耗高，可能出现数据不一致问题

#### **解决方案2 布隆过滤器**：
![[面试八股 - Redis-3.png|850]]
布隆过滤器用来检索一个数据是否存在 redis 中，如果不存在直接返回空，如果存在查询 redis 
原理：
- 存储：内部是使用一个位图存储，当存储 id 为 1 的数据时，通过多个 hash 值计算得出数组的下标，并置 1。
- 查询数据：使用同样的 hash 函数计算，如果下标位置都为1，说明缓存中存在，但也可能会出现误判

### 缓存击穿

**定义**：当一个热点 key 过期的同时，大量的数据访问这个 key ，但此时缓存没有就会去查找数据库，大量的请求打到数据库就会导致数据库宕机。

#### **解决方案1 添加互斥锁**

![[面试八股 - Redis-4.png|500]]

当第一个线程未查询到缓存时，获取互斥锁，然后开始重建缓存数据，在这期间其他所有的线程都只能休眠然后等待重建，当重建完成后释放锁，这个时候其他等待的线程即刻缓存命中。
优点：保证数据的强一致性
缺点：会导致性能较差

#### **解决方案2 逻辑过期**

![[面试八股 - Redis-5.png|700]]

不设置数据的过期时间，如果数据过期，则新开一个线程去重建缓存数据，写入缓存重置逻辑过期时间，并释放锁，同时其他线程 直接返回过期数据
优点：具有高可用性，性能优越

### 雪崩

**定义**：数据雪崩是指在同一时间所有的 key 同时失效或 redis 宕机导致请求去到数据库压垮数据库服务器
#### **解决方案1 随机过期时间**
通过将所有数据的过期时间设定一个 TTL 随机值，让其不会在同一时间过期

#### 解决方案1(Redis 宕机) Redis 哨兵和集群
可以通过设置 Redis 的集群模式和哨兵模式来预防提高可用性

#### 解决方案2(Redis 宕机) 添加降级限流策略
可以通过在 Nginx 和 Spring Cloud Gateway 设置降级限流策略预防

#### 解决方案3(Redis 宕机) 添加多级缓存
可以通过 Guava 本地缓存减缓数据冲击

---
## 双写一致性：mysql 的数据如何与 redis 的数据进行同步

### 设置前提

设置前提，介绍自己的业务背景需求 `一致性要求高`还是 `允许延迟一致`

### 一致性要求高（读写一致性）

`双写一致性` : 当修改了数据库的数据同时也要更新缓存的数据，缓存和数据库的数据需要保持一致


![[Pasted image 20250318163226.png|700]]
#### 采用 **延迟双删** 操作

1. 读操作： 缓存命中直接返回；缓存未命中查询数据库，写入缓存，设定超时时间

2. 写操作：延迟双删
![[Pasted image 20250318163356.png|775]]
	1. 先删除缓存还是先修改数据库**都**可能会导致缓存和数据库数据不一致
	2. 因为删除两次缓存可以有效的减少脏数据的风险，主要原因是第一次删除缓存，可以将旧数据删除，然后修改数据库，修改完成后再删除缓存，再将修改后的数据存入数据库即可一定程度减少脏数据风险
	3. 延迟是因为如果设置了数据库主从模式，需要从数据库同步完成，但是延迟的过程也可能会有脏数据风险


### 采用 **读写锁** 操作(强一致性)

**读写锁**
`共享锁` : 读锁 readLock,加锁之后，其他线程可以共享读操作
`排他锁` : 独占锁 writeLock ，加锁之后阻塞其他线程的读写操作
![[Pasted image 20250318164103.png]]
读数据的时候使用共享锁读数据，写数据的时候添加排他锁，这个方法可以保证强一致性，但是性能不高，主要是通过 redisson 实现的

### 最终一致性（允许延迟一致）

#### 采用 **异步通知** 保证数据的最终一致性
![[Pasted image 20250318164543.png|525]]
修改数据写入到 MySQL 以后就会发一条信息给 MQ ，缓存服务 cache-service 监听接收到 MQ 消息，然后更新缓存

虽然有延迟，但是最终还是会更新缓存

#### 采用 Canal 的异步通知
主要使用 MySQL 的主从同步实现的

![[Pasted image 20250318183115.png|625]]

监听 binlog 的二进制日志文件
优点：对代码几乎不进行侵入

---
## 持久化问题：redis 作为缓存其持久化是怎么做的

`RDB` 全称 Redis Database Backup file （Redis 数据备份文件） ,也成为 Redis 数据快照，简单来说就是把内存中的所有数据记录到磁盘中，当 Redis 实例故障重启后，从磁盘读取快照文件，恢复数据

一共由 RDB 和 AOF 两种方式
### 命令方式

**手动触发**

`save` 由 Redis 主进程来执行 RDB，会阻塞所有命令
`bgsave` 开启子进程执行 RDB，避免主进程受到影响

### RDB 执行原理

bgsave 开始会 fork 主进程的到子进程，子进程 共享 主进程的内存数据。完成 fork 后读取内存数据并写入 RDB 文件 ![[Pasted image 20250318191844.png|675]]
**流程：** 
1. Redis 将数据存储在物理内存，需要备份时物理地址创建一份数据拷贝， redis 主进程则是通过**页表**，即虚拟地址与物理地址的映射关系进行读写操作
2. 当使用 bgsave 命令开始时会 fork 主进程到子进程，也就相当于复制了一份页表给子进程
3. 子进程就可以通过页表读取物理内存的数据拷贝进行备份到磁盘替换旧 RDB 文件。

其中，如果子进程在备份的时候主进程仍然在写入则可能出现脏数据问题，所以当 fork 的同时会设置物理内存的数据为 read-only 模式，这也就是 fork 采用的 copy-on-write 技术

**copy-on-write 技术**
- 当主进程进行读操作时可以访问共享物理内存
- 当主进程进行写操作时，会拷贝一份数据，然后再数据的拷贝上执行 写操作

### AOF 方式

AOF 全称 Append Only File (追加文件)。Redis 处理的每一个写命令都会记录在 AOF 文件中，可以看作时命令日志文件

#### 开启 AOF

在 `redis.conf` 配置文件中开启 AOF
![[Pasted image 20250318192940.png]]

AOF 的命令记录的频率配置
![[Pasted image 20250318193034.png]]
一般在项目中都是设置 appendfsync everysec ，是默认方案

通过 `bgrewriteaof` 命令可以让 AOF 文件执行重写功能
![[Pasted image 20250318193312.png]]

### RDB 与 AOF 对比


|         | RDB                 | AOF                                   |
| ------- | ------------------- | ------------------------------------- |
| 持久化方式   | 定时对这个内存做快照          | 记录每一次执行的命令                            |
| 数据完整性   | 不完全，两次备份之间会有丢失      | 相对完整，取决于刷盘的策略                         |
| 文件大小    | 因为是二进制文件会进行压缩所以比较小  | 记录的是命令，文件见体积大                         |
| 宕机恢复速度  | 很快                  | 慢                                     |
| 数据恢复优先级 | 低，因为数据完整性不如AOF      | 高，数据完整性更高                             |
| 系统资源占用  | 较大，需要大量的 CPU 和内存消耗  | 低，主要吃磁盘IO资源<br>但 AOF 重写需要大量的CPU 和内存消耗 |
| 使用场景    | 可以忍受数分钟的数据丢失，追求启动速度 | 对数据安全性要求较高                            |
开发中往往结合两者一起使用

- **混合持久化**（Redis 4.0+）：开启`aof-use-rdb-preamble`，AOF文件包含RDB头+增量命令，兼顾恢复速度与数据安全。


---

## Redis 数据的过期策略

redis 中对数据设置有效期，过期就从内存中删除，而这些删除的规则就是数据的删除策略有 `惰性删除` 和 `定期删除`

### 惰性删除

设置该 key 过期时间后，不管它，当再次需要该 key 的时候再检查是否过去，如果过期就删掉否则返回该key

**优点**： 对 CPU 友好，用不到的 key 不会浪费时间进行过期检查
**缺点**：对内存不友好，如果一个 key 过期但一直没有用就会一直在内存中永不释放

### 定期删除

每隔一段时间对一些 key 进行检查，删除里面过期的 key

定期删除两种模式
- SLOW 模式是定时任务，执行频率默认为 10hz ，每次不超过 25ms, 通过修改配置文件 redis.conf 的 hz 选项来调整次数
- FAST 模式执行频率不固定，但两次间隔不低于 2ms ，每次耗时不超过 1ms

优点 ： 通过限制删除操作执行时长和频率来减少对 CPU 的影响。另外定期删除也能有效释放过期键占用的内存
缺点：难以确定删除执行的时长和频率

 **Redis 过期删除策略：惰性删除+定期删除两种策略配合使用**


---

## 数据淘汰策略 - 缓存过多，内存被占满了如何处理?

当 Redis 中的内存不够用，再添加新的 key，Redis 就会按照某种规则将内存中的数据删掉，这个规则就是数据淘汰策略

### 8种淘汰策略(不需要死记)

noeviction : 不淘汰任何 key，但内存满不会存储新数据，默认

有3个标签组成出6个策略，加上一个 ttl 策略

- **random** : 随机淘汰
- **lru** : 最少最近(Recently)使用，当前的时间减去最近访问的时间，数值越大淘汰优先级越高
- **lfu**：最少频率(Frequently)使用，统计每个key 的访问频率，值越小淘汰优先级越高
- **allkeys** ： 对全体 key
- **valatile** ： 对设置了 TTL 值的key
![[Pasted image 20250318204412.png|675]]

### 使用建议
1. 优先使用 allkeys-lru 策略。利用 LRU 的算法把最常访问的数据留在缓存中，如果业务有明显的冷热数据区分时建议使用
2. 如果业务访问频率不大，没有明显冷热数据区分，建议使用 allkeys-random 随机淘汰
3. 如果有置顶的数据，可以使用 volatile-lru 策略，然后不设置置顶数据的 ttl
4. 如果业务中有短时高频访问数据，可以使用带 lfu 的策略

### 其他面试题

#### 数据库有 1000万 条数据， Redis 只能缓存 20w 数据，如何保证 Redis 中的数据都是热点数据？

使用 allkeys-lru 最近最少使用的数据淘汰策略，以保证缓存中都是热点数据

#### Redis 的内存用完会发生什么？

主要看淘汰策略，不过如果是默认 noeviction 的策略会报错

---

# Redis 作为分布式锁

**分布式锁**：分布式锁的主要功能是跨进程跨设备的对需要共享的资源进行多线程并发控制，是解决分布式系统中并发访问同一份资源的一个机制，为了满足互斥性，安全性，可用性和避免死锁等特性

## 执行流程
![[面试八股 - Redis-6.png|875]]
在这个代码中，第一个线程想要加锁，然后操作 redis ，如果加锁成功，会**创建一个看门狗定时的去给锁续期**，也就是每过三分之一的锁过期时间进行一次续期，等运行完成后释放锁，通知看门狗结束，如果第二个线程想要加锁，但不成功的话，会进行一个等待循环等待时间，当等待到一定时间之后还没有成功加锁会结束，等待到了直接进行加锁

这个代码运行的过程主要是依靠 lua 脚本实现的
![[面试八股 - Redis-7.png|600]]

## 可重入

redisson 实现的分布式锁是 **可重入** 的

**可重入**：
![[面试八股 - Redis-8.png|600]]
在这个图片中，解释可重入最主要的来源是 redisson 是利用 hash 结构记录线程 id 和重入次数的，也就是说，如果在同一个线程中，可以多次加锁，加锁的时候每多加一次锁，都会判断是否是同一个线程，如果是则 value 值 +1 ，如果解锁则 value 值 -1 ，知道 value 值为 0 时释放锁。

## 分布式锁是如何实现的？

分布式锁是解决分布式系统中并发访问同一份资源的一个机制，为了满足互斥性，安全性，可用性和避免死锁等特性，有几种方法可以实现，

第一种是通过 **setnx** 命令实现的，然后使用 DEL lock_key 释放锁，接着可以设定参数 set nx ex 30 这种命令来实现更具有互斥性和避免死锁的方法。

第二种是通过 Reids 在 Java 中的 **Redisson** 实现，他的底层是依靠 setnx 命令与 lua 脚本。 他的工作流程就是
- 线程上锁之后，redisson 会用当前的线程 id 和重入次数作为锁的value 值，key 可以自己设定为锁的名称，
- 这时线程获取锁了之后，redisson 会创建一个看门狗来检测任务执行时间，每当锁的过期时间进过三分之一时就会重置锁的过期时间，
- 如果这时有一个新的线程想要获取锁，就会获取失败，然后进入一段时间的循环等待任务完成获取锁，从而保证原子性和唯一性

分布式锁是解决分布式系统中并发访问共享资源的一种机制，它需要满足互斥性、安全性、可用性以及避免死锁等特性。主要有以下几种实现方式：

### SETNX
最基础的实现是使用Redis的`SETNX`(SET if Not eXists)命令：
```
SETNX lock_key unique_value
```
再通过`DEL lock_key`释放锁。但这种方式存在问题：如果客户端崩溃，锁无法自动释放。

### 改进版：SET NX EX
```
SET lock_key unique_value NX EX 30
```
这种方式设置锁的同时指定过期时间，防止死锁。释放时需要校验value确保锁被正确释放：
```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

### Redisson实现

Redisson是Java中最完善的Redis分布式锁实现：

1. **可重入机制**：使用哈希结构存储`{threadId: 重入次数}`
2. **自动延期（看门狗）**：后台线程定时检查，当锁持有时间超过过期时间的1/3时，自动续期
3. **阻塞等待**：支持公平锁和非公平锁模式的等待获取
4. **Lua脚本**：保证获取锁和释放锁的原子性

## Redis分布式锁的问题及解决方案

### 主从切换问题

单实例Redis在故障转移时可能导致锁丢失。解决方案是使用Redis Redlock算法：

1. 在多个独立Redis实例上获取锁
2. 当超过半数实例获取成功则认为锁获取成功
3. 所有实例使用相同的过期时间
4. 客户端需要维护一个有效时间计算，确保操作在锁有效期内完成

### 锁竞争问题

高并发下锁竞争激烈时，可采用以下优化：

1. 使用分段锁减少锁粒度
2. 实现退避算法，避免立即重试
3. 设置合理超时时间，避免长时间阻塞

--- 
# Redis 集群方案

## 主从复制
![[面试八股 - Redis-9.png|825]]
Redis 的主从复制主要是一个 redis 服务器写，其他的服务器读，因为 redis 主要还是读多写少，所以这样的设定可以提高并发能力

### 主从复制全量同步

![[面试八股 - Redis-10.png]]
全量同步主要包含两个部分 **第一次同步** 和 **RDB期间增量同步** 

#### 第一次同步

第一次同步的关键点是 replid ,这个 id 是数据集标记， id 一致说明是同一数据集，如果不一致说明是第一次同步，则返回 master 的版本信息和 offset,先保存版本信息然后主节点发送 RDB 文件，从节点加载 RDB 文件

#### RDB 期间增量同步

如果在 RDB 期间的所有命令，都会存储到主节点的 repl_baklog 中，这时 主节点的 offset 值会增加，如果与 从节点的保存的 offset 值不一样，则会根据这个增量的区间同步过来。

### 主从复制增量同步
 ![[面试八股 - Redis-11.png|625]]
当从节点重启或者后期数据变化时，回向主节点发送一个请求，主节点判断 replid 是否一致，一致去 repl_baklog 中获取 offset 值之后的数据，然后同步给从节点

---

## 哨兵模式

集群模式如果主节点出现故障会丧失写操作，不具备高可用性，所以 redis 还有哨兵模式

![[面试八股 - Redis-12.png|850]]
Redis 的哨兵 Sentinel 机制可以实现主从节点的自动故障恢复。
### 监控

Sentinel 会不断检查主节点和从节点是否按预期工作
#### 服务状态监控

是采用心跳机制检测服务状态，每隔一秒向 redis 集群发送一次 Ping 命令
- **主观下线**：如果某一个 Sentinel 节点的 ping 命令在一段时间没有被响应，则认为这个节点主观下线
- **客观下线**：如果超过指定数量的 Sentinel 节点都认为这个节点主观下线，则这个实例为客观下线

#### 哨兵选主规则

- 首先判断从节点的优先级值，越小优先级越高
- 如果优先级一样，判断 offset 值，越大说明与主节点的值越完整，优先级越高
- 最后判断从节点的 id 大小。

#### 哨兵模式脑裂

当网络出现问题时，这个主节点与从节点和哨兵处于不同的网络分区，但此哨兵监控中没有感觉到主节点心跳，所以这时就会出现从节点会自动提升一个主节点，但客户端还是一致在往老的主节点中写入数据

![[面试八股 - Redis-13.png|550]]

这时当网络恢复后，曾经的老主节点就会被强制降为从节点，但此时这个从节点会从主节点同步数据，这个时候之前客户端写入的数据就会被清空丢失。

![[面试八股 - Redis-14.png|550]]

**解决方法**：
修改 Redis 的配置参数
- 设置最小的从节点数量为一个 `min-replicas-to-write 1` ，就是这个主节点必须要有**至少一个从节点**才能接受数据，否则直接拒绝请求即可

- 配置主从复制的延迟不可超过五秒 `min-replicas-max-lag 5` 
### 自动故障恢复

如果主节点故障，哨兵会将一个从节点升级为主节点，故障恢复后还是以新主节点为主
### 通知

哨兵充当 Redis 客户端的服务发现源，当集群出现故障转移，将最新信息推送给客户端

--- 

## 分片集群

主要解决 **高并发写** 和 **海量数据存储** 问题

![[面试八股 - Redis 1.png|500]]

### 分片集群结构特征

- 集群中多个 master
- 每个 master 都有对应的 从节点
- 没有哨兵，每个 master 互为检测彼此健康状态
- 有实现路由机制，请求集群的任意节点都会被转发到正确的节点

### 数据读写

#### 哈希槽 

Redis 中有 16384 个插槽，会将这些插槽分配给不同的实例，读写时会根据 key 的有效部分（{大括号内是有效部分}，如果没有 key 名称为有小部分分）对 16384 取余的余数得到插槽，寻找插槽所在实例。

# Redis 单线程为什么快

- Redis 是纯内存操作
- 采用单线程，避免了不必要的上下文切换
- 使用的 I/O 多路复用模型，非阻塞 IO

## IO 多路复用模型
主要实现的是高效的网络请求

## 用户空间和内核空间

## 常见 IO 模型
### 阻塞 IO
### 非阻塞 IO
### IO 多路复用

## Redis 网络模型

