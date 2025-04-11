
> 以下内容属于 AI 生成，主要是个人用于学习和了解，请客观了解理性看待，模型使用 claude-sonnet-3-7-thinking

## 第一章：RedisTemplate基础

### 定义解释：什么是RedisTemplate？

RedisTemplate 是 Spring Data Redis 提供的一个高级抽象类，封装了对 Redis 数据库的各种操作，使开发者可以更便捷地与 Redis 进行交互，而无需直接处理底层的连接和命令执行细节。它是 Spring 应用中操作 Redis 的主要入口点。

### 核心组件

#### 1. ConnectionFactory

ConnectionFactory 是 RedisTemplate 的核心依赖，负责管理与 Redis 服务器的连接：

- **RedisConnectionFactory** 接口定义了创建和管理 Redis 连接的基本方法
- 主要实现包括：
  - **JedisConnectionFactory**：基于 Jedis 客户端的连接工厂
  - **LettuceConnectionFactory**：基于 Lettuce 客户端的连接工厂（Spring Boot 2.0+ 默认使用）

#### 2. Serializers（序列化器）

Redis 是键值数据库，存储的是字节数组。序列化器负责在 Java 对象和 Redis 存储格式间进行转换：

| 序列化器                                   | 适用场景     | 优点           | 缺点                 |
| -------------------------------------- | -------- | ------------ | ------------------ |
| **StringRedisSerializer**              | 字符串类型的键值 | 可读性高，兼容性好    | 仅支持String类型        |
| **GenericJackson2JsonRedisSerializer** | 复杂对象存储   | 可读性好，支持大多数类型 | 包含类信息，占用空间大        |
| **Jackson2JsonRedisSerializer**        | 特定类型对象   | 高效，不存储类型信息   | 需要指定类型，灵活性低        |
| **JdkSerializationRedisSerializer**    | Java特有对象 | 使用Java原生序列化  | 只能在Java系统间使用，存储空间大 |
| **OxmSerializer**                      | XML数据    | 支持XML映射      | 较少使用，性能较低          |

### 配置示例

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(factory);
        
        // 设置key的序列化方式
        template.setKeySerializer(new StringRedisSerializer());
        // 设置value的序列化方式
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        
        // Hash类型的key序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        // Hash类型的value序列化方式
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        
        // 初始化RedisTemplate
        template.afterPropertiesSet();
        return template;
    }
}
```

# 第二章：常见操作指南

## 2.1 字符串操作（opsForValue）

字符串是Redis最基本的数据类型，可存储字符串、整数、浮点数或二进制数据。在RedisTemplate中，通过`opsForValue()`方法访问字符串操作。

### 2.1.1 基本操作

```java
// 设置字符串值
redisTemplate.opsForValue().set("user:name", "张三");

// 获取字符串值
String name = redisTemplate.opsForValue().get("user:name");

// 删除键
Boolean deleted = redisTemplate.delete("user:name");

// 检查键是否存在
Boolean exists = redisTemplate.hasKey("user:name");

// 设置新值并返回旧值
String oldValue = redisTemplate.opsForValue().getAndSet("user:name", "李四");

// 追加字符串
Integer newLength = redisTemplate.opsForValue().append("user:name", "-后缀");

// 获取字符串长度
Long length = redisTemplate.opsForValue().size("user:name");

// 设置值的一部分（从offset开始）
redisTemplate.opsForValue().set("user:name", "王", 1);
// 结果: "张王"
```

### 2.1.2 过期时间操作

```java
// 设置带过期时间的值
redisTemplate.opsForValue().set("verification:code", "123456", 30, TimeUnit.SECONDS);

// 仅当键不存在时设置值（带过期时间）
Boolean setSuccess = redisTemplate.opsForValue().setIfAbsent(
    "login:token", UUID.randomUUID().toString(), 2, TimeUnit.HOURS);

// 仅当键存在时设置值（带过期时间）
Boolean updateSuccess = redisTemplate.opsForValue().setIfPresent(
    "session:token", newToken, 1, TimeUnit.HOURS);

// 单独设置过期时间
redisTemplate.expire("user:session", 30, TimeUnit.MINUTES);

// 获取剩余过期时间
Long ttl = redisTemplate.getExpire("user:session", TimeUnit.SECONDS);

// 移除过期时间设置（使键永久存在）
Boolean persist = redisTemplate.persist("important:config");
```

### 2.1.3 原子性操作

```java
// 原子性递增（整数）
Long newCount = redisTemplate.opsForValue().increment("visitor:count");

// 原子性递增指定数值
Long newStock = redisTemplate.opsForValue().increment("product:stock", 5);

// 原子性递减
Long remainingAttempts = redisTemplate.opsForValue().decrement("login:attempts");

// 原子性递减指定数值
Long newValue = redisTemplate.opsForValue().decrement("countdown", 2);

// 浮点数递增
Double newScore = redisTemplate.opsForValue().increment("player:score", 1.5);
```

### 2.1.4 批量操作

```java
// 批量获取值
List<String> keys = Arrays.asList("user:1:name", "user:2:name", "user:3:name");
List<Object> values = redisTemplate.opsForValue().multiGet(keys);

// 批量设置值
Map<String, String> map = new HashMap<>();
map.put("user:1:name", "张三");
map.put("user:2:name", "李四");
map.put("user:3:name", "王五");
redisTemplate.opsForValue().multiSet(map);

// 批量设置（当所有键都不存在时）
Boolean setSuccess = redisTemplate.opsForValue().multiSetIfAbsent(map);
```

### 2.1.5 位图操作

```java
// 设置位图的特定位
redisTemplate.opsForValue().setBit("user:online:bitmap", 100, true);

// 获取位图的特定位
Boolean isOnline = redisTemplate.opsForValue().getBit("user:online:bitmap", 100);

// 统计位图中设置为1的位数
Long onlineCount = redisTemplate.execute((RedisCallback<Long>) conn -> 
    conn.bitCount("user:online:bitmap".getBytes()));
```

### 2.1.6 应用场景

#### 缓存对象
```java
// 缓存用户对象（序列化为JSON）
User user = userService.findById(1001);
redisTemplate.opsForValue().set("user:1001", user, 1, TimeUnit.HOURS);

// 获取缓存的用户对象
User cachedUser = (User) redisTemplate.opsForValue().get("user:1001");
```

#### 分布式计数器
```java
// 网页访问计数器
Long viewCount = redisTemplate.opsForValue().increment("article:1001:views");
System.out.println("文章访问次数：" + viewCount);
```

#### 限流器实现
```java
// 简单的IP限流（1分钟内最多10次请求）
String ipKey = "rate:limit:" + ipAddress;
Long requestCount = redisTemplate.opsForValue().increment(ipKey);
if (requestCount == 1) {
    // 第一次访问，设置60秒过期
    redisTemplate.expire(ipKey, 60, TimeUnit.SECONDS);
}
if (requestCount > 10) {
    throw new RateLimitException("请求频率过高，请稍后再试");
}
```

#### 分布式锁
```java
// 简单的分布式锁实现
String lockKey = "lock:order:" + orderId;
Boolean acquired = redisTemplate.opsForValue().setIfAbsent(lockKey, "LOCKED", 10, TimeUnit.SECONDS);
if (Boolean.TRUE.equals(acquired)) {
    try {
        // 执行需要加锁的操作
        processOrder(orderId);
    } finally {
        // 释放锁
        redisTemplate.delete(lockKey);
    }
} else {
    // 未获取到锁，可以重试或返回繁忙提示
}
```

#### 验证码存储
```java
// 生成并存储短信验证码
String code = generateRandomCode();
String codeKey = "sms:code:" + phoneNumber;
redisTemplate.opsForValue().set(codeKey, code, 5, TimeUnit.MINUTES);

// 验证码校验
String inputCode = request.getParameter("code");
String storedCode = redisTemplate.opsForValue().get(codeKey);
if (storedCode != null && storedCode.equals(inputCode)) {
    // 验证通过，删除验证码防止重用
    redisTemplate.delete(codeKey);
    return "验证成功";
} else {
    return "验证码错误或已过期";
}
```

## 2.2 哈希操作（opsForHash）

Redis哈希是字段-值对的集合，类似于Java中的Map结构。适合存储对象和关联数据。通过`opsForHash()`方法访问哈希操作。

### 2.2.1 基本操作

```java
// 设置哈希字段
redisTemplate.opsForHash().put("user:profile:1001", "name", "李四");
redisTemplate.opsForHash().put("user:profile:1001", "age", "28");
redisTemplate.opsForHash().put("user:profile:1001", "email", "lisi@example.com");

// 获取哈希字段
String userName = (String) redisTemplate.opsForHash().get("user:profile:1001", "name");

// 获取所有哈希字段和值
Map<Object, Object> entries = redisTemplate.opsForHash().entries("user:profile:1001");

// 获取所有字段名
Set<Object> fields = redisTemplate.opsForHash().keys("user:profile:1001");

// 获取所有字段值
List<Object> values = redisTemplate.opsForHash().values("user:profile:1001");

// 判断字段是否存在
Boolean hasEmail = redisTemplate.opsForHash().hasKey("user:profile:1001", "email");

// 获取哈希大小（字段数量）
Long size = redisTemplate.opsForHash().size("user:profile:1001");

// 删除一个或多个哈希字段
Long deletedFields = redisTemplate.opsForHash().delete("user:profile:1001", "temporary_field", "another_field");

// 删除整个哈希
redisTemplate.delete("user:profile:1001");
```

### 2.2.2 批量操作

```java
// 批量设置哈希字段
Map<String, String> fields = new HashMap<>();
fields.put("name", "王五");
fields.put("age", "35");
fields.put("city", "上海");
redisTemplate.opsForHash().putAll("user:profile:1002", fields);

// 批量获取哈希字段
List<Object> fieldValues = redisTemplate.opsForHash().multiGet(
    "user:profile:1002", 
    Arrays.asList("name", "age", "city")
);
```

### 2.2.3 原子性操作

```java
// 哈希字段值递增（整数）
Long newLoginCount = redisTemplate.opsForHash().increment("user:stats:1001", "login_count", 1);

// 哈希字段值递增（浮点数）
Double newBalance = redisTemplate.opsForHash().increment("user:balance:1001", "amount", 15.5);

// 当字段不存在时设置（模拟putIfAbsent）
Boolean valueWasSet = redisTemplate.opsForHash().putIfAbsent("user:profile:1001", "vip", "false");
```

### 2.2.4 应用场景

#### 用户资料存储
```java
// 存储用户资料（避免序列化整个对象，更新单个字段更高效）
String userKey = "user:" + userId;
redisTemplate.opsForHash().put(userKey, "name", user.getName());
redisTemplate.opsForHash().put(userKey, "email", user.getEmail());
redisTemplate.opsForHash().put(userKey, "phone", user.getPhone());
redisTemplate.opsForHash().put(userKey, "age", String.valueOf(user.getAge()));
redisTemplate.opsForHash().put(userKey, "last_login", new Date().toString());

// 更新单个字段
redisTemplate.opsForHash().put(userKey, "email", "new_email@example.com");

// 读取用户资料
Map<Object, Object> userInfo = redisTemplate.opsForHash().entries(userKey);
```

#### 购物车实现
```java
// 添加商品到购物车
String cartKey = "cart:" + userId;
// 添加/更新商品数量
redisTemplate.opsForHash().put(cartKey, productId, "2");

// 增加商品数量
redisTemplate.opsForHash().increment(cartKey, productId, 1);

// 获取购物车商品数量
String quantity = (String) redisTemplate.opsForHash().get(cartKey, productId);

// 移除购物车中的商品
redisTemplate.opsForHash().delete(cartKey, productId);

// 获取整个购物车
Map<Object, Object> cartItems = redisTemplate.opsForHash().entries(cartKey);
```

#### 网站计数器聚合
```java
// 记录不同页面的访问量
String counterKey = "page:counters:" + dateStr;
redisTemplate.opsForHash().increment(counterKey, "/home", 1);
redisTemplate.opsForHash().increment(counterKey, "/products", 1);
redisTemplate.opsForHash().increment(counterKey, "/about", 1);

// 获取特定页面的计数
Long homeViews = Long.valueOf(redisTemplate.opsForHash().get(counterKey, "/home").toString());

// 获取所有页面访问量
Map<Object, Object> allPageViews = redisTemplate.opsForHash().entries(counterKey);
```

#### 配置中心
```java
// 存储应用配置
String configKey = "app:config";
redisTemplate.opsForHash().putAll(configKey, Map.of(
    "max_connections", "100", 
    "timeout", "3000",
    "cache_enabled", "true",
    "default_language", "zh-CN"
));

// 读取配置
String timeout = (String) redisTemplate.opsForHash().get(configKey, "timeout");
boolean cacheEnabled = "true".equals(redisTemplate.opsForHash().get(configKey, "cache_enabled"));

// 动态更新配置
redisTemplate.opsForHash().put(configKey, "max_connections", "200");
```

## 2.3 列表操作（opsForList）

Redis列表是简单的字符串列表，按照插入顺序排序，支持头尾操作。通过`opsForList()`方法访问列表操作。

### 2.3.1 基本操作

```java
// 从左侧添加元素（头部）
redisTemplate.opsForList().leftPush("messages", "新消息1");

// 从右侧添加元素（尾部）
redisTemplate.opsForList().rightPush("messages", "新消息2");

// 批量添加元素（左侧）
redisTemplate.opsForList().leftPushAll("messages", "消息A", "消息B", "消息C");

// 批量添加元素（右侧）
List<String> newMessages = Arrays.asList("消息D", "消息E", "消息F");
redisTemplate.opsForList().rightPushAll("messages", newMessages);

// 在指定元素前/后插入
redisTemplate.opsForList().leftPush("messages", "消息B", "消息B前面");
redisTemplate.opsForList().rightPush("messages", "消息B", "消息B后面");

// 设置指定索引处的值
redisTemplate.opsForList().set("messages", 1, "替换的消息");

// 获取列表长度
Long size = redisTemplate.opsForList().size("messages");

// 获取指定索引处的元素
String message = redisTemplate.opsForList().index("messages", 0);

// 获取指定范围的元素
List<String> rangeResult = redisTemplate.opsForList().range("messages", 0, 2);
// 获取全部元素
List<String> allMessages = redisTemplate.opsForList().range("messages", 0, -1);

// 从左侧弹出元素（头部）
String firstMessage = redisTemplate.opsForList().leftPop("messages");

// 从右侧弹出元素（尾部）
String lastMessage = redisTemplate.opsForList().rightPop("messages");

// 移除指定元素
redisTemplate.opsForList().remove("messages", 1, "消息B前面"); // 从头部开始删除1个"消息B前面"

// 裁剪列表（仅保留指定范围的元素）
redisTemplate.opsForList().trim("messages", 0, 9);  // 仅保留前10个元素
```

### 2.3.2 阻塞操作

```java
// 阻塞弹出（最多等待10秒）
String waitingMessage = redisTemplate.opsForList().leftPop("queue:tasks", 10, TimeUnit.SECONDS);

// 弹出src列表最右元素并推入dst列表左侧（常用于任务转移）
String movedItem = redisTemplate.opsForList().rightPopAndLeftPush("queue:pending", "queue:processing");

// 阻塞版本的rightPopAndLeftPush
String waitAndMove = redisTemplate.opsForList().rightPopAndLeftPush(
    "queue:pending", "queue:processing", 5, TimeUnit.SECONDS);
```

### 2.3.3 应用场景

#### 消息队列
```java
// 生产者：添加任务到队列
String taskId = UUID.randomUUID().toString();
Task task = new Task(taskId, "处理数据", TaskPriority.HIGH);
redisTemplate.opsForList().rightPush("queue:tasks", objectMapper.writeValueAsString(task));

// 消费者：从队列中取出任务
String taskJson = redisTemplate.opsForList().leftPop("queue:tasks");
if (taskJson != null) {
    Task task = objectMapper.readValue(taskJson, Task.class);
    // 处理任务...
    System.out.println("处理任务: " + task.getName());
}

// 阻塞式消费者（适合长轮询）
public void startTaskConsumer() {
    while (true) {
        String taskJson = redisTemplate.opsForList().leftPop("queue:tasks", 30, TimeUnit.SECONDS);
        if (taskJson != null) {
            // 处理任务...
        }
    }
}
```

#### 最新活动列表
```java
// 记录用户活动（最多保留100条）
String timeline = "user:activity:" + userId;
redisTemplate.opsForList().leftPush(timeline, new ActivityLog("登录系统").toJson());
redisTemplate.opsForList().leftPush(timeline, new ActivityLog("更新资料").toJson());
redisTemplate.opsForList().leftPush(timeline, new ActivityLog("发布评论").toJson());
// 修剪列表
redisTemplate.opsForList().trim(timeline, 0, 99);

// 获取最近10条活动
List<String> recentActivities = redisTemplate.opsForList().range(timeline, 0, 9);
```

#### 分页数据存储
```java
// 存储文章评论（按时间顺序）
String commentsKey = "article:comments:" + articleId;
redisTemplate.opsForList().rightPush(commentsKey, comment.toJson());

// 分页获取评论
int pageSize = 10;
int pageNumber = 2; // 从0开始
int startIndex = pageNumber * pageSize;
int endIndex = startIndex + pageSize - 1;
List<String> comments = redisTemplate.opsForList().range(commentsKey, startIndex, endIndex);
```

#### 栈与队列实现
```java
// 队列操作（FIFO - 先进先出）
String queueKey = "queue:print";
// 入队
redisTemplate.opsForList().rightPush(queueKey, "文档1");
redisTemplate.opsForList().rightPush(queueKey, "文档2");
// 出队
String nextDocument = redisTemplate.opsForList().leftPop(queueKey);

// 栈操作（LIFO - 后进先出）
String stackKey = "stack:undo";
// 入栈
redisTemplate.opsForList().leftPush(stackKey, "操作1");
redisTemplate.opsForList().leftPush(stackKey, "操作2");
// 出栈
String lastOperation = redisTemplate.opsForList().leftPop(stackKey);
```

## 2.4 集合操作（opsForSet）

Redis集合是无序字符串集合，支持添加、删除和集合运算操作。通过`opsForSet()`方法访问集合操作。

### 2.4.1 基本操作

```java
// 添加元素到集合
Long addedCount = redisTemplate.opsForSet().add("user:tags:1001", "技术", "Java", "Redis", "Spring");

// 获取集合所有成员
Set<String> tags = redisTemplate.opsForSet().members("user:tags:1001");

// 判断元素是否存在
Boolean hasTag = redisTemplate.opsForSet().isMember("user:tags:1001", "Java");

// 随机获取集合中的元素
String randomTag = redisTemplate.opsForSet().randomMember("user:tags:1001");

// 随机获取多个不重复元素
List<String> distinctTags = redisTemplate.opsForSet().distinctRandomMembers("user:tags:1001", 2);

// 随机获取多个元素（可能重复）
List<String> randomTags = redisTemplate.opsForSet().randomMembers("user:tags:1001", 3);

// 移除元素
Long removedCount = redisTemplate.opsForSet().remove("user:tags:1001", "临时标签", "旧标签");

// 从源集合移动元素到目标集合
Boolean moved = redisTemplate.opsForSet().move("user:tags:1001", "Java", "popular:tags");

// 获取集合大小
Long size = redisTemplate.opsForSet().size("user:tags:1001");

// 弹出元素（随机移除并返回一个元素）
String poppedTag = redisTemplate.opsForSet().pop("user:tags:1001");

// 弹出多个元素
List<String> poppedTags = redisTemplate.opsForSet().pop("user:tags:1001", 2);
```

### 2.4.2 集合运算操作

```java
// 交集
Set<String> intersection = redisTemplate.opsForSet().intersect("user:tags:1001", "user:tags:1002");
// 多个集合交集
Set<String> multiIntersection = redisTemplate.opsForSet().intersect(
    Arrays.asList("user:tags:1001", "user:tags:1002", "user:tags:1003"));
// 交集并存储结果
Long intersectionCount = redisTemplate.opsForSet().intersectAndStore(
    "user:tags:1001", "user:tags:1002", "common:tags");

// 并集
Set<String> union = redisTemplate.opsForSet().union("user:tags:1001", "user:tags:1002");
// 多个集合并集
Set<String> multiUnion = redisTemplate.opsForSet().union(
    Arrays.asList("user:tags:1001", "user:tags:1002", "user:tags:1003"));
// 并集并存储结果
Long unionCount = redisTemplate.opsForSet().unionAndStore(
    "user:tags:1001", "user:tags:1002", "all:tags");

// 差集（在第一个集合但不在第二个集合的元素）
Set<String> difference = redisTemplate.opsForSet().difference("user:tags:1001", "user:tags:1002");
// 多个集合差集
Set<String> multiDifference = redisTemplate.opsForSet().difference(
    Arrays.asList("user:tags:1001", "user:tags:1002", "user:tags:1003"));
// 差集并存储结果
Long differenceCount = redisTemplate.opsForSet().differenceAndStore(
    "user:tags:1001", "user:tags:1002", "unique:tags");
```

### 2.4.3 应用场景

#### 标签系统
```java
// 为用户添加兴趣标签
String userTagsKey = "user:tags:" + userId;
redisTemplate.opsForSet().add(userTagsKey, "电影", "旅行", "美食", "科技");

// 为内容添加标签
String contentTagsKey = "content:tags:" + contentId;
redisTemplate.opsForSet().add(contentTagsKey, "科技", "编程", "人工智能");

// 查找有相同兴趣的内容
Set<String> matchingTags = redisTemplate.opsForSet().intersect(userTagsKey, contentTagsKey);
Double matchScore = (double) matchingTags.size() / redisTemplate.opsForSet().size(contentTagsKey);

// 查找喜欢特定标签的所有用户
Set<String> userKeys = redisTemplate.keys("user:tags:*");
List<Object> userIdsLikingTag = new ArrayList<>();
for (String key : userKeys) {
    if (redisTemplate.opsForSet().isMember(key, "人工智能")) {
        // 从key中提取userId
        String userId = key.substring("user:tags:".length());
        userIdsLikingTag.add(userId);
    }
}
```

#### 好友关系
```java
// 添加好友关系
redisTemplate.opsForSet().add("user:friends:" + userId, friendId1, friendId2, friendId3);

// 检查是否为好友
Boolean isFriend = redisTemplate.opsForSet().isMember("user:friends:" + userId, targetUserId);

// 获取共同好友
Set<String> mutualFriends = redisTemplate.opsForSet().intersect(
    "user:friends:" + userId, 
    "user:friends:" + targetUserId
);

// 好友推荐（我的好友的好友，但不是我的好友）
Set<String> myFriends = redisTemplate.opsForSet().members("user:friends:" + userId);
Set<String> friendsOfFriends = new HashSet<>();

for (String friendId : myFriends) {
    Set<String> friendsFriends = redisTemplate.opsForSet().members("user:friends:" + friendId);
    friendsOfFriends.addAll(friendsFriends);
}

// 移除已经是我的好友的用户
friendsOfFriends.removeAll(myFriends);
// 移除自己
friendsOfFriends.remove(userId);
```

#### 唯一访问统计
```java
// 记录网站每日独立访问用户
String dailyVisitorsKey = "stats:daily:visitors:" + LocalDate.now().toString();
redisTemplate.opsForSet().add(dailyVisitorsKey, userId);
// 设置过期时间（保留90天）
redisTemplate.expire(dailyVisitorsKey, 90, TimeUnit.DAYS);

// 获取当日独立访问用户数
Long uniqueVisitors = redisTemplate.opsForSet().size(dailyVisitorsKey);

// 连续多日都活跃的用户
String yesterdayKey = "stats:daily:visitors:" + LocalDate.now().minusDays(1).toString();
Set<String> loyalUsers = redisTemplate.opsForSet().intersect(dailyVisitorsKey, yesterdayKey);
```

#### 抽奖系统
```java
// 用户参与抽奖
redisTemplate.opsForSet().add("raffle:participants", userId1, userId2, userId3, userId4);

// 随机抽取一名获奖者
String winner = redisTemplate.opsForSet().randomMember("raffle:participants");

// 随机抽取多名获奖者（不重复）
List<String> winners = redisTemplate.opsForSet().distinctRandomMembers("raffle:participants", 3);

// 移除已获奖用户
redisTemplate.opsForSet().remove("raffle:participants", winners.toArray());
```

## 2.5 有序集合操作（opsForZSet）

Redis有序集合类似集合，但每个元素关联一个分数，通过分数排序。通过`opsForZSet()`方法访问有序集合操作。

### 2.5.1 基本操作

```java
// 添加元素和分数
Boolean added = redisTemplate.opsForZSet().add("leaderboard", "player1", 100.0);
// 批量添加
Set<ZSetOperations.TypedTuple<String>> tuples = new HashSet<>();
tuples.add(new DefaultTypedTuple<>("player2", 85.5));
tuples.add(new DefaultTypedTuple<>("player3", 66.0));
tuples.add(new DefaultTypedTuple<>("player4", 95.0));
Long addedCount = redisTemplate.opsForZSet().add("leaderboard", tuples);

// 获取元素分数
Double score = redisTemplate.opsForZSet().score("leaderboard", "player1");

// 增加元素分数
Double newScore = redisTemplate.opsForZSet().incrementScore("leaderboard", "player1", 10.5);

// 获取元素排名（从低到高，0为最低）
Long rank = redisTemplate.opsForZSet().rank("leaderboard", "player1");

// 获取元素排名（从高到低，0为最高）
Long reverseRank = redisTemplate.opsForZSet().reverseRank("leaderboard", "player1");

// 删除元素
Long removed = redisTemplate.opsForZSet().remove("leaderboard", "player3", "player4");

// 获取有序集合大小
Long size = redisTemplate.opsForZSet().size("leaderboard");
// 或者使用zCard获取大小
Long count = redisTemplate.opsForZSet().zCard("leaderboard");

// 统计指定分数范围内的元素数量
Long countInRange = redisTemplate.opsForZSet().count("leaderboard", 80, 100);

// 获取集合中的所有元素（按分数从低到高）
Set<String> allPlayers = redisTemplate.opsForZSet().range("leaderboard", 0, -1);

// 获取集合中的所有元素及其分数（按分数从低到高）
Set<ZSetOperations.TypedTuple<String>> playersWithScores = 
    redisTemplate.opsForZSet().rangeWithScores("leaderboard", 0, -1);

// 获取集合中的所有元素（按分数从高到低）
Set<String> topPlayers = redisTemplate.opsForZSet().reverseRange("leaderboard", 0, -1);

// 按排名范围获取元素（获取前三名）
Set<String> top3 = redisTemplate.opsForZSet().reverseRange("leaderboard", 0, 2);

// 按排名范围获取元素及其分数
Set<ZSetOperations.TypedTuple<String>> top3WithScores = 
    redisTemplate.opsForZSet().reverseRangeWithScores("leaderboard", 0, 2);
```

### 2.5.2 分数范围操作

```java
// 按分数范围获取元素（获取80-100分数段的玩家，从低到高）
Set<String> highScorePlayers = redisTemplate.opsForZSet().rangeByScore("leaderboard", 80, 100);

// 按分数范围获取元素及其分数
Set<ZSetOperations.TypedTuple<String>> playersWithScoresInRange = 
    redisTemplate.opsForZSet().rangeByScoreWithScores("leaderboard", 80, 100);

// 按分数范围获取元素（从高到低）
Set<String> highScorePlayersDesc = redisTemplate.opsForZSet().reverseRangeByScore("leaderboard", 80, 100);

// 按分数范围获取元素，带偏移和限制（分页）
Set<String> pagedPlayers = redisTemplate.opsForZSet().rangeByScore("leaderboard", 0, 100, 5, 10);
// 从索引5开始获取10个元素

// 移除指定排名范围内的元素（移除排名前3的玩家）
Long removedCount = redisTemplate.opsForZSet().removeRange("leaderboard", 0, 2);

// 移除指定分数范围内的元素（移除分数<60的玩家）
Long removedByScore = redisTemplate.opsForZSet().removeRangeByScore("leaderboard", 0, 59.99);
```

### 2.5.3 集合操作

```java
// 计算并集并存储到新集合（默认取最大分数）
Long unionCount = redisTemplate.opsForZSet().unionAndStore(
    "leaderboard:jan", "leaderboard:feb", "leaderboard:q1");

// 多个集合的并集
Long unionMultiCount = redisTemplate.opsForZSet().unionAndStore(
    "leaderboard:jan", 
    Arrays.asList("leaderboard:feb", "leaderboard:mar"), 
    "leaderboard:q1");

// 自定义聚合规则的并集
RedisZSetCommands.Aggregate aggregate = RedisZSetCommands.Aggregate.SUM;
RedisZSetCommands.Weights weights = RedisZSetCommands.Weights.of(1, 2);
Long unionWithWeights = redisTemplate.opsForZSet().unionAndStore(
    "scores:math", "scores:physics", "scores:total", 
    aggregate, weights);

// 计算交集并存储
Long intersectCount = redisTemplate.opsForZSet().intersectAndStore(
    "active:users:today", "premium:users", "active:premium:users");

// 多个集合的交集
Long intersectMultiCount = redisTemplate.opsForZSet().intersectAndStore(
    "users:group1", 
    Arrays.asList("users:active", "users:vip"), 
    "users:active:vip:group1");

// 自定义聚合规则的交集
RedisZSetCommands.Aggregate minAggregate = RedisZSetCommands.Aggregate.MIN;
Long intersectWithMin = redisTemplate.opsForZSet().intersectAndStore(
    "server1:response_time", "server2:response_time", "servers:best_response",
    minAggregate);
```

### 2.5.4 应用场景

#### 排行榜系统
```java
// 更新玩家分数
redisTemplate.opsForZSet().add("game:leaderboard", "player123", 1000);

// 玩家得分增加
redisTemplate.opsForZSet().incrementScore("game:leaderboard", "player123", 50);

// 获取玩家当前排名（从1开始）
Long rank = redisTemplate.opsForZSet().reverseRank("game:leaderboard", "player123");
long userRank = rank != null ? rank + 1 : 0;
System.out.println("玩家排名: " + userRank);

// 获取排行榜前10名玩家
Set<ZSetOperations.TypedTuple<String>> topPlayers = 
    redisTemplate.opsForZSet().reverseRangeWithScores("game:leaderboard", 0, 9);

// 展示排行榜
int ranking = 1;
for (ZSetOperations.TypedTuple<String> player : topPlayers) {
    System.out.println(
        ranking++ + ". 玩家: " + player.getValue() + 
        ", 分数: " + player.getScore()
    );
}

// 获取某玩家附近的排名（比如前后5名）
Long playerRank = redisTemplate.opsForZSet().reverseRank("game:leaderboard", "player123");
if (playerRank != null) {
    long start = Math.max(0, playerRank - 5);
    long end = playerRank + 5;
    Set<String> nearbyPlayers = redisTemplate.opsForZSet().reverseRange("game:leaderboard", start, end);
    System.out.println("附近玩家: " + nearbyPlayers);
}
```

#### 时间序列数据
```java
// 使用时间戳作为分数记录温度传感器数据
String sensorKey = "sensor:temperature:device001";
double timestamp = System.currentTimeMillis() / 1000.0;
double temperature = 22.5;
redisTemplate.opsForZSet().add(sensorKey, String.valueOf(temperature), timestamp);

// 获取最近30分钟的传感器数据
double now = System.currentTimeMillis() / 1000.0;
double thirtyMinutesAgo = now - (30 * 60);
Set<ZSetOperations.TypedTuple<String>> recentReadings = 
    redisTemplate.opsForZSet().rangeByScoreWithScores(sensorKey, thirtyMinutesAgo, now);

// 处理和分析数据
for (ZSetOperations.TypedTuple<String> reading : recentReadings) {
    double readingTime = reading.getScore();
    double value = Double.parseDouble(reading.getValue());
    LocalDateTime readingDateTime = 
        LocalDateTime.ofInstant(Instant.ofEpochSecond((long)readingTime), ZoneId.systemDefault());
    System.out.println("时间: " + readingDateTime + ", 温度: " + value);
}

// 数据清理（只保留最近24小时的数据）
double oneDayAgo = now - (24 * 60 * 60);
redisTemplate.opsForZSet().removeRangeByScore(sensorKey, 0, oneDayAgo);
```

#### 带权重的搜索结果
```java
// 为搜索结果添加多个权重因素
String searchResultKey = "search:java:redis";

// 根据相关性、点击率和发布时间计算最终分数
Map<String, Double> documentScores = new HashMap<>();
documentScores.put("doc1", calculateScore(0.8, 120, 7)); // 相关性，点击数，天数
documentScores.put("doc2", calculateScore(0.9, 50, 2));
documentScores.put("doc3", calculateScore(0.7, 200, 30));

// 批量添加到排序集合
Set<ZSetOperations.TypedTuple<String>> scoredDocs = new HashSet<>();
for (Map.Entry<String, Double> entry : documentScores.entrySet()) {
    scoredDocs.add(new DefaultTypedTuple<>(entry.getKey(), entry.getValue()));
}
redisTemplate.opsForZSet().add(searchResultKey, scoredDocs);
redisTemplate.expire(searchResultKey, 10, TimeUnit.MINUTES); // 搜索结果缓存10分钟

// 获取分页搜索结果（第2页，每页10条）
int page = 1;
int pageSize = 10;
Set<String> results = redisTemplate.opsForZSet().reverseRange(
    searchResultKey, 
    page * pageSize, 
    (page + 1) * pageSize - 1
);

// 计算搜索结果分数的辅助方法
private double calculateScore(double relevance, int clicks, int daysOld) {
    // 自定义打分公式：相关性 * 点击率 * 时间衰减因子
    double timeDecay = Math.exp(-0.05 * daysOld);
    return relevance * Math.log(1 + clicks) * timeDecay;
}
```

#### 优先级队列
```java
// 任务优先级队列
String taskQueueKey = "tasks:priority:queue";

// 添加任务（分数越低优先级越高）
String task1 = "{\"id\":\"t1\",\"name\":\"紧急任务\",\"data\":\"...\"}"
String task2 = "{\"id\":\"t2\",\"name\":\"普通任务\",\"data\":\"...\"}"
String task3 = "{\"id\":\"t3\",\"name\":\"低优先级任务\",\"data\":\"...\"}"

redisTemplate.opsForZSet().add(taskQueueKey, task1, 1); // 高优先级
redisTemplate.opsForZSet().add(taskQueueKey, task2, 5); // 中优先级
redisTemplate.opsForZSet().add(taskQueueKey, task3, 10); // 低优先级

// 工作线程：获取优先级最高的任务
Set<String> highestPriorityTasks = redisTemplate.opsForZSet().range(taskQueueKey, 0, 0);
if (!highestPriorityTasks.isEmpty()) {
    String taskToProcess = highestPriorityTasks.iterator().next();
    // 立即移除任务，避免其他工作线程重复处理
    redisTemplate.opsForZSet().remove(taskQueueKey, taskToProcess);
    
    // 处理任务...
    System.out.println("处理任务: " + taskToProcess);
}

// 延迟任务：使用时间戳作为分数
long executeAt = System.currentTimeMillis() + 3600000; // 一小时后执行
String delayedTask = "{\"id\":\"t4\",\"name\":\"延迟执行任务\",\"data\":\"...\"}"
redisTemplate.opsForZSet().add("tasks:delayed", delayedTask, executeAt);

// 调度器：获取当前应执行的所有任务
long currentTime = System.currentTimeMillis();
Set<String> tasksToExecute = redisTemplate.opsForZSet().rangeByScore(
    "tasks:delayed", 0, currentTime);

// 移除这些任务并处理
redisTemplate.opsForZSet().removeRangeByScore("tasks:delayed", 0, currentTime);
```

## 2.6 事务管理

Redis事务允许将多个命令打包，然后一次性、按顺序执行，中间不会被其他客户端命令打断。RedisTemplate提供了事务支持，通过SessionCallback接口实现。

### 2.6.1 基本事务操作

```java
// 执行Redis事务
List<Object> txResults = redisTemplate.execute(new SessionCallback<>() {
    @Override
    public <K, V> Object execute(RedisOperations<K, V> operations) {
        operations.multi(); // 开始事务
        
        // 事务中的所有命令都会被加入队列，不会立即执行
        operations.opsForValue().set("tx:key1", "value1");
        operations.opsForHash().put("tx:hash1", "field1", "value2");
        operations.opsForList().leftPush("tx:list1", "item1");
        operations.opsForSet().add("tx:set1", "member1");
        operations.opsForZSet().add("tx:zset1", "member1", 1.0);
        
        // 在事务中获取值总是返回null，因为命令尚未执行
        Object val = operations.opsForValue().get("tx:key1");  // 将返回null
        
        // 执行事务并返回结果
        return operations.exec();
    }
});

// 处理事务结果
if (txResults != null) {
    for (int i = 0; i < txResults.size(); i++) {
        System.out.println("命令" + i + "结果: " + txResults.get(i));
    }
}
```

Kotlin版本：

```kotlin
// 执行Redis事务
val txResults = redisTemplate.execute { operations ->
    operations.multi() // 开始事务
    
    operations.opsForValue().set("tx:key1", "value1")
    operations.opsForHash().put("tx:hash1", "field1", "value2")
    operations.opsForList().leftPush("tx:list1", "item1")
    
    operations.exec() // 执行事务并返回结果
}
```

### 2.6.2 使用乐观锁（Watch）实现CAS操作

Redis通过WATCH命令提供乐观锁机制，当事务执行前被监视的键发生变化时，事务会被取消。

```java
// 使用Watch实现乐观锁
String key = "inventory:product:10001";
Boolean txSuccess = redisTemplate.execute(new SessionCallback<Boolean>() {
    @Override
    public <K, V> Boolean execute(RedisOperations<K, V> operations) {
        // 监视key，如果它被其他客户端修改，事务将失败
        operations.watch(key);
        
        // 检查库存
        Integer currentStock = (Integer) operations.opsForValue().get(key);
        if (currentStock == null || currentStock < 10) {
            operations.unwatch();
            return false; // 库存不足，取消事务
        }
        
        // 开始事务
        operations.multi();
        
        // 减少库存
        operations.opsForValue().set(key, currentStock - 10);
        
        // 执行事务
        List<Object> result = operations.exec();
        
        // 如果事务被打断，exec会返回空列表
        return result != null && !result.isEmpty();
    }
});

if (Boolean.TRUE.equals(txSuccess)) {
    System.out.println("库存更新成功");
} else {
    System.out.println("库存更新失败，可能是并发修改导致");
}
```

### 2.6.3 管道与事务对比

```java
// 使用事务确保原子性
List<Object> txResults = redisTemplate.execute(new SessionCallback<>() {
    @Override
    public <K, V> Object execute(RedisOperations<K, V> operations) {
        operations.multi();
        // 所有操作作为一个原子单元执行，要么全部成功，要么全部失败
        for (int i = 0; i < 5; i++) {
            operations.opsForValue().set("tx:key" + i, "value" + i);
        }
        return operations.exec();
    }
});

// 使用管道提高吞吐量（但不保证原子性）
List<Object> pipelineResults = redisTemplate.executePipelined(new SessionCallback<>() {
    @Override
    public <K, V> Object execute(RedisOperations<K, V> operations) {
        // 所有操作批量发送，但不保证原子性
        for (int i = 0; i < 1000; i++) {
            operations.opsForValue().set("pipeline:key" + i, "value" + i);
        }
        return null; // 管道操作结果由List<Object>接收
    }
});
```

### 2.6.4 事务应用场景

#### 库存扣减与订单创建
```java
// 使用事务同时完成库存扣减和订单创建
Boolean success = redisTemplate.execute(new SessionCallback<Boolean>() {
    @Override
    public <K, V> Boolean execute(RedisOperations<K, V> operations) {
        String stockKey = "product:stock:" + productId;
        String orderKey = "orders:" + orderId;
        
        // 监视库存键
        operations.watch(stockKey);
        
        // 检查库存
        Integer stock = (Integer) operations.opsForValue().get(stockKey);
        if (stock == null || stock < quantity) {
            operations.unwatch();
            return false; // 库存不足
        }
        
        // 开始事务
        operations.multi();
        
        // 扣减库存
        operations.opsForValue().set(stockKey, stock - quantity);
        
        // 创建订单
        Map<String, String> orderData = new HashMap<>();
        orderData.put("productId", productId);
        orderData.put("userId", userId);
        orderData.put("quantity", String.valueOf(quantity));
        orderData.put("status", "CREATED");
        orderData.put("createTime", String.valueOf(System.currentTimeMillis()));
        operations.opsForHash().putAll(orderKey, orderData);
        
        // 执行事务
        List<Object> results = operations.exec();
        
        return results != null && !results.isEmpty();
    }
});

if (Boolean.TRUE.equals(success)) {
    System.out.println("订单创建成功");
} else {
    System.out.println("订单创建失败，可能是库存不足或并发更新");
}
```

#### 转账操作
```java
// 使用事务实现账户间转账
Boolean transferred = redisTemplate.execute(new SessionCallback<Boolean>() {
    @Override
    public <K, V> Boolean execute(RedisOperations<K, V> operations) {
        String fromAccountKey = "account:balance:" + fromAccountId;
        String toAccountKey = "account:balance:" + toAccountId;
        
        // 监视两个账户
        operations.watch(fromAccountKey);
        operations.watch(toAccountKey);
        
        // 检查余额
        Double fromBalance = (Double) operations.opsForValue().get(fromAccountKey);
        if (fromBalance == null || fromBalance < amount) {
            operations.unwatch();
            return false; // 余额不足
        }
        
        // 开始事务
        operations.multi();
        
        // 从付款方减少余额
        operations.opsForValue().set(fromAccountKey, fromBalance - amount);
        
        // 给收款方增加余额
        Double toBalance = (Double) operations.opsForValue().get(toAccountKey);
        toBalance = (toBalance == null) ? amount : toBalance + amount;
        operations.opsForValue().set(toAccountKey, toBalance);
        
        // 记录交易日志
        String txLogKey = "transactions:" + UUID.randomUUID().toString();
        Map<String, String> txData = new HashMap<>();
        txData.put("from", fromAccountId);
        txData.put("to", toAccountId);
        txData.put("amount", String.valueOf(amount));
        txData.put("time", String.valueOf(System.currentTimeMillis()));
        operations.opsForHash().putAll(txLogKey, txData);
        
        // 执行事务
        List<Object> results = operations.exec();
        
        return results != null && !results.isEmpty();
    }
});

System.out.println("转账" + (transferred ? "成功" : "失败"));
```

## 2.7 高级功能

### 2.7.1 发布/订阅模式

Redis的发布/订阅模式允许消息发送者（发布者）发送消息，而不直接指定消息接收者（订阅者）。订阅者通过订阅感兴趣的频道来接收消息。

#### 消息监听器配置

```java
// 1. 创建消息监听器
@Component
public class RedisMessageListener implements MessageListener {
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String channel = new String(message.getChannel(), StandardCharsets.UTF_8);
        String content = new String(message.getBody(), StandardCharsets.UTF_8);
        
        System.out.println("接收到频道 [" + channel + "] 的消息: " + content);
        
        // 处理消息...
        if ("notification".equals(channel)) {
            try {
                NotificationMessage notificationMsg = new ObjectMapper()
                    .readValue(content, NotificationMessage.class);
                // 处理通知...
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

// 2. 配置消息监听容器
@Configuration
public class RedisConfig {
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
                                                  RedisMessageListener messageListener) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        
        // 添加订阅的频道
        container.addMessageListener(
            messageListener, 
            new PatternTopic("notification")
        );
        
        // 添加多个频道
        container.addMessageListener(
            messageListener,
            Arrays.asList(
                new PatternTopic("system:alerts"),
                new PatternTopic("user:events")
            )
        );
        
        // 添加模式匹配订阅（使用通配符）
        container.addMessageListener(
            messageListener,
            new PatternTopic("user:*:updates")
        );
        
        return container;
    }
}
```

#### 发送消息

```java
// 基本消息发送
redisTemplate.convertAndSend("notification", "系统通知：服务即将升级");

// 发送结构化消息
NotificationMessage notification = new NotificationMessage(
    "SYSTEM_UPGRADE",
    "系统升级通知", 
    "系统将于明晚8点进行升级，预计停机30分钟。",
    "CRITICAL",
    System.currentTimeMillis()
);

String jsonMsg = new ObjectMapper().writeValueAsString(notification);
redisTemplate.convertAndSend("system:alerts", jsonMsg);

// 发送带有用户ID的消息
String userChannel = "user:" + userId + ":updates";
redisTemplate.convertAndSend(userChannel, "用户配置已更新");
```

#### 使用RedisTemplate手动订阅

除了使用RedisMessageListenerContainer，也可以使用RedisTemplate直接订阅频道：

```java
@Autowired
private RedisConnectionFactory connectionFactory;

public void subscribeToChannel(String channel) {
    new Thread(() -> {
        RedisConnection connection = connectionFactory.getConnection();
        connection.subscribe((message, pattern) -> {
            String content = new String(message.getBody(), StandardCharsets.UTF_8);
            System.out.println("接收到消息: " + content);
        }, channel.getBytes());
    }).start();
}
```

### 2.7.2 管道操作（executePipelined）

管道允许发送多个命令到服务器而无需等待响应，显著提高吞吐量：

```java
// 使用管道批量设置值
List<Object> results = redisTemplate.executePipelined(new RedisCallback<Object>() {
    @Override
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
        connection.openPipeline();
        
        // 直接使用底层连接
        for (int i = 0; i < 1000; i++) {
            connection.stringCommands().set(
                ("pipelined:key" + i).getBytes(),
                ("value" + i).getBytes()
            );
        }
        
        return null; // 结果由List<Object>接收
    }
});

// 使用管道执行批量读取
List<Object> values = redisTemplate.executePipelined(new RedisCallback<Object>() {
    @Override
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
        connection.openPipeline();
        
        for (int i = 0; i < 50; i++) {
            connection.stringCommands().get(("pipelined:key" + i).getBytes());
        }
        
        return null;
    }
});

// 管道配合操作模板
List<Object> mixedResults = redisTemplate.executePipelined(new SessionCallback<>() {
    @Override
    public <K, V> Object execute(RedisOperations<K, V> operations) {
        // 使用高级API进行管道操作
        for (int i = 0; i < 100; i++) {
            operations.opsForValue().set("pipeline:key" + i, "value" + i);
            operations.opsForValue().get("pipeline:key" + i);
            operations.opsForList().rightPush("pipeline:list", "item" + i);
        }
        return null;
    }
});
```

#### 优化数据批量导入

```java
// 批量导入用户数据
public void bulkImportUsers(List<User> users) {
    int batchSize = 500;
    
    for (int i = 0; i < users.size(); i += batchSize) {
        int endIndex = Math.min(i + batchSize, users.size());
        List<User> batch = users.subList(i, endIndex);
        
        redisTemplate.executePipelined(new SessionCallback<>() {
            @Override
            public <K, V> Object execute(RedisOperations<K, V> operations) {
                for (User user : batch) {
                    String key = "user:" + user.getId();
                    Map<String, String> userData = new HashMap<>();
                    userData.put("name", user.getName());
                    userData.put("email", user.getEmail());
                    userData.put("createdAt", String.valueOf(user.getCreatedAt().getTime()));
                    
                    operations.opsForHash().putAll(key, userData);
                    
                    // 添加到索引
                    operations.opsForSet().add("users:by:city:" + user.getCity(), user.getId());
                    
                    // 过期时间（可选）
                    operations.expire(key, 30, TimeUnit.DAYS);
                }
                return null;
            }
        });
        
        System.out.println("已导入 " + endIndex + " 个用户");
    }
}
```

### 2.7.3 过期时间设置

Redis可以为键设置生存时间（TTL），当键过期时会被自动删除。

```java
// 设置键的过期时间（具体时间值）
// 30分钟
redisTemplate.expire("session:token", 30, TimeUnit.MINUTES);
// 7天
redisTemplate.expire("user:profile", 7, TimeUnit.DAYS);
// 1小时
redisTemplate.expire("cache:data", 3600, TimeUnit.SECONDS);

// 设置键在指定的日期过期
Date expiryDate = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2023-12-31 23:59:59");
redisTemplate.expireAt("promotion:christmas", expiryDate);

// 设置Unix时间戳过期
long timestamp = System.currentTimeMillis() / 1000 + 86400; // 明天此时
redisTemplate.expireAt("temp:data", new Date(timestamp * 1000));

// 获取键的剩余过期时间
Long remainingSeconds = redisTemplate.getExpire("session:token");
Long remainingMinutes = redisTemplate.getExpire("session:token", TimeUnit.MINUTES);

// 移除过期时间设置（使键永久存在）
redisTemplate.persist("important:config");

// 判断键是否存在过期时间
Boolean hasExpiry = redisTemplate.expire("session:token", 0, TimeUnit.SECONDS) == false;

// 只有当键不存在时才设置值和过期时间（原子操作）
redisTemplate.opsForValue().setIfAbsent("lock:resource", "locked", 10, TimeUnit.SECONDS);

// 替换现有键的值并重新设置过期时间
redisTemplate.opsForValue().getAndSet("session:user:1001", newSessionData);
redisTemplate.expire("session:user:1001", 30, TimeUnit.MINUTES);

// 原子性地设置值和过期时间（推荐方式）
redisTemplate.opsForValue().set("cache:product:1001", productData, 1, TimeUnit.HOURS);
```

#### 过期时间应用场景

```java
// 会话管理
String sessionToken = UUID.randomUUID().toString();
String sessionKey = "session:" + sessionToken;
redisTemplate.opsForHash().putAll(sessionKey, sessionData);
redisTemplate.expire(sessionKey, 30, TimeUnit.MINUTES);

// 限时优惠
String promotionKey = "promo:summer2023";
redisTemplate.opsForHash().putAll(promotionKey, promoDetails);
// 设置到指定日期结束
LocalDateTime endDate = LocalDateTime.of(2023, 8, 31, 23, 59, 59);
ZonedDateTime zdt = endDate.atZone(ZoneId.systemDefault());
redisTemplate.expireAt(promotionKey, Date.from(zdt.toInstant()));

// 验证码有效期
String verificationKey = "verify:" + email;
redisTemplate.opsForValue().set(verificationKey, code, 10, TimeUnit.MINUTES);

// 缓存刷新策略（使用随机过期时间避免缓存雪崩）
Random random = new Random();
int baseExpiry = 3600; // 基础过期时间（1小时）
int delta = random.nextInt(600); // 随机增加0-10分钟
redisTemplate.opsForValue().set(cacheKey, cacheData, baseExpiry + delta, TimeUnit.SECONDS);

// 自动删除临时数据
String uploadKey = "temp:upload:" + userId;
redisTemplate.opsForList().rightPushAll(uploadKey, fileIds);
redisTemplate.expire(uploadKey, 24, TimeUnit.HOURS);
```

### 2.7.4 Lua脚本执行

Redis支持使用Lua脚本进行服务器端编程，可以实现更复杂的原子操作。RedisTemplate提供了脚本执行的支持：

```java
// 定义一个简单的Lua脚本（检查键是否存在，不存在则设置）
String script = "if redis.call('exists', KEYS[1]) == 0 then " +
                "  redis.call('set', KEYS[1], ARGV[1]) " +
                "  return 1 " +
                "else " +
                "  return 0 " +
                "end";

// 创建脚本对象
RedisScript<Long> redisScript = RedisScript.of(script, Long.class);

// 执行脚本
Long result = redisTemplate.execute(
    redisScript,
    Collections.singletonList("script:test:key"), // KEYS参数
    "hello world"                                // ARGV参数
);

System.out.println("脚本执行结果: " + result);
```

#### 多键和多参数脚本

```java
// 定义一个复杂的Lua脚本（原子性地增加多个计数器）
String multiIncScript = 
    "local results = {} " +
    "for i=1,#KEYS do " +
    "  results[i] = redis.call('incrby', KEYS[i], ARGV[i]) " +
    "end " +
    "return results";

RedisScript<List> multiIncRedisScript = RedisScript.of(multiIncScript, List.class);

// 准备键和参数
List<String> keys = Arrays.asList("counter:a", "counter:b", "counter:c");
Object[] args = {"1", "5", "10"}; // 增加的值

// 执行脚本
List<Long> results = (List<Long>) redisTemplate.execute(
    multiIncRedisScript,
    keys,
    args
);

for (int i = 0; i < keys.size(); i++) {
    System.out.println(keys.get(i) + " 新值: " + results.get(i));
}
```

#### 加载脚本文件

对于复杂的脚本，最好从外部文件加载：

```java
@Configuration
public class RedisScriptConfig {
    @Bean
    public RedisScript<Boolean> lockScript() {
        Resource scriptResource = new ClassPathResource("scripts/lock.lua");
        return RedisScript.of(scriptResource, Boolean.class);
    }
    
    @Bean
    public RedisScript<Long> rateLimiterScript() {
        Resource scriptResource = new ClassPathResource("scripts/rate_limiter.lua");
        return RedisScript.of(scriptResource, Long.class);
    }
}

// 使用注入的脚本
@Autowired
private RedisScript<Boolean> lockScript;

public boolean acquireLock(String lockKey, String lockValue, int expireSeconds) {
    return Boolean.TRUE.equals(redisTemplate.execute(
        lockScript,
        Collections.singletonList(lockKey),
        lockValue, String.valueOf(expireSeconds)
    ));
}
```

#### Lua脚本应用场景

##### 1. 分布式锁的获取与释放

```java
// 获取锁的Lua脚本
String acquireLockScript = 
    "if redis.call('setnx', KEYS[1], ARGV[1]) == 1 then " +
    "  redis.call('expire', KEYS[1], ARGV[2]) " +
    "  return 1 " +
    "else " +
    "  return 0 " +
    "end";

// 释放锁的Lua脚本（确保只释放自己的锁）
String releaseLockScript = 
    "if redis.call('get', KEYS[1]) == ARGV[1] then " +
    "  return redis.call('del', KEYS[1]) " +
    "else " +
    "  return 0 " +
    "end";

// 创建脚本对象
RedisScript<Long> acquireScript = RedisScript.of(acquireLockScript, Long.class);
RedisScript<Long> releaseScript = RedisScript.of(releaseLockScript, Long.class);

// 使用分布式锁
String lockKey = "lock:resource:12345";
String lockValue = UUID.randomUUID().toString(); // 唯一标识
int expireSeconds = 30;

try {
    // 尝试获取锁
    Long acquired = redisTemplate.execute(
        acquireScript, 
        Collections.singletonList(lockKey),
        lockValue, String.valueOf(expireSeconds)
    );
    
    if (acquired == 1) {
        // 获取锁成功，执行受保护的代码
        System.out.println("获取锁成功，执行业务逻辑");
        // ... 执行业务逻辑 ...
    } else {
        System.out.println("获取锁失败");
    }
} finally {
    // 释放锁
    Long released = redisTemplate.execute(
        releaseScript,
        Collections.singletonList(lockKey),
        lockValue
    );
    
    System.out.println("锁释放" + (released == 1 ? "成功" : "失败，可能已过期或被其他线程释放"));
}
```

##### 2. 限流器实现

```java
// 令牌桶限流算法Lua实现
String rateLimiterScript = 
    "local key = KEYS[1] " +
    "local max_tokens = tonumber(ARGV[1]) " +
    "local tokens_per_sec = tonumber(ARGV[2]) " +
    "local now = tonumber(ARGV[3]) " +
    "local requested = tonumber(ARGV[4]) " +
    
    -- 获取当前桶信息或创建新的
    "local info = redis.call('HMGET', key, 'last_time', 'tokens') " +
    "local last_time = info[1] " +
    "local available_tokens = tonumber(info[2]) " +
    
    "if last_time == false then " +
    "  available_tokens = max_tokens " +
    "  last_time = 0 " +
    "end " +
    
    -- 计算新增令牌数
    "local time_passed = math.max(0, now - last_time) " +
    "local new_tokens = math.min(max_tokens, available_tokens + (time_passed * tokens_per_sec)) " +
    
    -- 是否有足够令牌
    "local allowed = new_tokens >= requested " +
    "local new_available_tokens = new_tokens " +
    "if allowed then " +
    "  new_available_tokens = new_tokens - requested " +
    "end " +
    
    -- 更新桶信息
    "redis.call('HMSET', key, 'last_time', now, 'tokens', new_available_tokens) " +
    "redis.call('EXPIRE', key, 10) " +
    
    "return allowed and 1 or 0";

RedisScript<Long> rateLimiterRedisScript = RedisScript.of(rateLimiterScript, Long.class);

// 使用限流器
public boolean isAllowed(String userId, int tokensRequested) {
    String rateLimiterKey = "rate:limiter:" + userId;
    
    // 参数: 最大令牌数, 每秒生成的令牌数, 当前时间戳(秒), 请求的令牌数
    Long result = redisTemplate.execute(
        rateLimiterRedisScript,
        Collections.singletonList(rateLimiterKey),
        "100",                                   // 最大100个令牌
        "10",                                    // 每秒生成10个令牌
        String.valueOf(System.currentTimeMillis() / 1000), // 当前时间
        String.valueOf(tokensRequested)          // 请求的令牌数量
    );
    
    return result == 1;
}
```

##### 3. 延迟队列实现

```java
// 延迟队列出队Lua脚本
String popDelayedMsgScript = 
    "local queue_key = KEYS[1] " +
    "local processing_queue_key = KEYS[2] " +
    "local now = tonumber(ARGV[1]) " +
    
    -- 获取到期的消息
    "local messages = redis.call('ZRANGEBYSCORE', queue_key, 0, now, 'LIMIT', 0, 1) " +
    "if #messages == 0 then " +
    "  return nil " +
    "end " +
    
    "local message = messages[1] " +
    
    -- 从延迟队列移除消息
    "redis.call('ZREM', queue_key, message) " +
    
    -- 移到处理队列
    "redis.call('ZADD', processing_queue_key, now + 30, message) " + -- 30秒处理超时
    
    "return message";

RedisScript<String> popDelayedMsgScript = RedisScript.of(popDelayedMsgScript, String.class);

// 添加延迟消息
public void addDelayedMessage(String message, long delayInMillis) {
    long executeAtMs = System.currentTimeMillis() + delayInMillis;
    redisTemplate.opsForZSet().add("delayed:queue", message, executeAtMs / 1000.0);
}

// 轮询处理到期消息
@Scheduled(fixedRate = 1000) // 每秒执行一次
public void processDelayedMessages() {
    while (true) {
        String message = redisTemplate.execute(
            popDelayedMsgScript,
            Arrays.asList("delayed:queue", "processing:queue"),
            String.valueOf(System.currentTimeMillis() / 1000)
        );
        
        if (message == null) {
            break; // 没有到期消息
        }
        
        try {
            // 处理消息...
            System.out.println("处理延迟消息: " + message);
            
            // 处理成功后从处理队列中移除
            redisTemplate.opsForZSet().remove("processing:queue", message);
        } catch (Exception e) {
            // 处理失败，消息将在超时后重新处理
            e.printStackTrace();
        }
    }
}

// 重新处理超时的消息
@Scheduled(fixedRate = 30000) // 每30秒执行一次
public void reprocessTimeoutMessages() {
    long now = System.currentTimeMillis() / 1000;
    Set<String> timeoutMessages = redisTemplate.opsForZSet()
        .rangeByScore("processing:queue", 0, now);
    
    for (String message : timeoutMessages) {
        // 从处理队列移回延迟队列
        redisTemplate.opsForZSet().remove("processing:queue", message);
        redisTemplate.opsForZSet().add("delayed:queue", message, now);
        System.out.println("重新入队超时消息: " + message);
    }
}
```

##### 4. 计数器并检查限制

```java
// 递增计数并检查是否超过限制的Lua脚本
String incrementAndCheckScript = 
    "local key = KEYS[1] " +
    "local limit = tonumber(ARGV[1]) " +
    "local expiry = tonumber(ARGV[2]) " +
    
    "local current = redis.call('incr', key) " +
    "if current == 1 then " +
    "  redis.call('expire', key, expiry) " +
    "end " +
    
    "return current <= limit and 1 or 0";

RedisScript<Long> incrementAndCheckScript = RedisScript.of(incrementAndCheckScript, Long.class);

// 使用场景：API请求限制
public boolean isRequestAllowed(String ip, int limit, int expirySeconds) {
    String key = "request:limit:" + ip;
    
    Long result = redisTemplate.execute(
        incrementAndCheckScript,
        Collections.singletonList(key),
        String.valueOf(limit),
        String.valueOf(expirySeconds)
    );
    
    return result == 1;
}

// 使用例子
public String handleApiRequest(String ip, String requestData) {
    // 限制每个IP每分钟最多60个请求
    if (isRequestAllowed(ip, 60, 60)) {
        // 处理请求...
        return "请求处理成功";
    } else {
        return "请求频率过高，请稍后再试";
    }
}
```

##### 5. 原子操作多个键的计数器

```java
// 在一次操作中更新多个计数器
String multiCounterScript = 
    "local result = {} " +
    "for i = 1, #KEYS do " +
    "  local value = redis.call('incrby', KEYS[i], ARGV[i]) " +
    "  if tonumber(ARGV[i+#KEYS]) > 0 then " +
    "    redis.call('expire', KEYS[i], ARGV[i+#KEYS]) " +
    "  end " +
    "  table.insert(result, value) " +
    "end " +
    "return result";

RedisScript<List> multiCounterScript = RedisScript.of(multiCounterScript, List.class);

// 更新多种统计数据
public void trackUserActivity(String userId, String action) {
    String dayKey = "stats:day:" + LocalDate.now() + ":" + action;
    String userKey = "user:" + userId + ":activity:" + action;
    String totalKey = "stats:total:" + action;
    
    List<String> keys = Arrays.asList(dayKey, userKey, totalKey);
    
    // 增加计数的值
    Object[] increments = {"1", "1", "1"};
    
    // 设置过期时间（天统计保留7天，用户统计保留30天，总计数不过期）
    Object[] expiries = {"604800", "2592000", "0"};
    
    // 合并参数
    Object[] args = new Object[increments.length + expiries.length];
    System.arraycopy(increments, 0, args, 0, increments.length);
    System.arraycopy(expiries, 0, args, increments.length, expiries.length);
    
    List<Long> result = (List<Long>) redisTemplate.execute(
        multiCounterScript,
        keys,
        args
    );
    
    System.out.println("日统计: " + result.get(0) + 
                      ", 用户统计: " + result.get(1) + 
                      ", 总计数: " + result.get(2));
}
```

通过这些详细的操作指南和示例，您应该能够掌握RedisTemplate的各种常见操作，并能够根据具体的业务场景选择合适的数据结构和操作方式。Redis的灵活性和强大功能使其在缓存、会话管理、计数器、队列、限流等各种场景下都能发挥重要作用。


## 第三章：最佳实践示例

### 缓存穿透解决方案

缓存穿透是指查询一个不存在的数据，导致每次都会穿透到数据库。

```java
public User getUserById(String userId) {
    String cacheKey = "user:" + userId;
    
    // 1. 尝试从缓存获取
    User user = (User) redisTemplate.opsForValue().get(cacheKey);
    
    if (user != null) {
        return user; // 缓存命中直接返回
    }
    
    // 2. 判断缓存中是否存储了空值（防止缓存穿透）
    if (redisTemplate.hasKey(cacheKey)) {
        return null; // 之前查询过，确认是空值
    }
    
    // 3. 查询数据库
    user = userRepository.findById(userId);
    
    // 4. 将结果存入缓存（即使是null也缓存，但设置短期过期时间）
    if (user == null) {
        // 存储空值并设置较短的过期时间
        redisTemplate.opsForValue().set(cacheKey, null, 5, TimeUnit.MINUTES);
    } else {
        // 存储实际值并设置较长的过期时间
        redisTemplate.opsForValue().set(cacheKey, user, 1, TimeUnit.HOURS);
    }
    
    return user;
}
```

### 分布式锁实现

使用RedisTemplate实现一个简单但可靠的分布式锁：

```java
public class RedisDistributedLock {
    private final RedisTemplate<String, String> redisTemplate;
    private final String lockKey;
    private final String lockValue;
    private final long expireTime;
    
    public RedisDistributedLock(RedisTemplate<String, String> redisTemplate, 
                               String lockKey, 
                               long expireTime) {
        this.redisTemplate = redisTemplate;
        this.lockKey = lockKey;
        this.lockValue = UUID.randomUUID().toString(); // 唯一标识
        this.expireTime = expireTime;
    }
    
    /**
     * 尝试获取锁
     */
    public boolean tryLock() {
        Boolean result = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, expireTime, TimeUnit.MILLISECONDS);
        return Boolean.TRUE.equals(result);
    }
    
    /**
     * 释放锁（确保只释放自己的锁）
     */
public boolean releaseLock() {
    // 使用Lua脚本确保操作的原子性
    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                    "return redis.call('del', KEYS[1]) " +
                    "else return 0 end";
    
    Long result = redisTemplate.execute(
        RedisScript.of(script, Long.class),
        Collections.singletonList(lockKey),
        lockValue
    );
    
    return Long.valueOf(1).equals(result);
}
```

使用示例：

```java
// 使用分布式锁确保分布式环境下的操作原子性
RedisDistributedLock lock = new RedisDistributedLock(redisTemplate, "lock:order:12345", 30000);
try {
    if (lock.tryLock()) {
        // 执行需要加锁的业务逻辑
        processOrder(orderId);
    } else {
        throw new RuntimeException("获取锁失败，请稍后重试");
    }
} finally {
    // 确保锁释放
    lock.releaseLock();
}
```

### 热点数据缓存策略

针对高频访问的热点数据，可以采用多级缓存策略：

```java
@Service
public class ProductService {
    private final RedisTemplate<String, Product> redisTemplate;
    private final ProductRepository productRepository;
    
    // 本地缓存（适用于单机部署）
    private final LoadingCache<String, Product> localCache;
    
    @Autowired
    public ProductService(RedisTemplate<String, Product> redisTemplate, 
                         ProductRepository productRepository) {
        this.redisTemplate = redisTemplate;
        this.productRepository = productRepository;
        
        // 初始化本地缓存
        this.localCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(1, TimeUnit.MINUTES)
            .build(key -> loadFromRedis(key));
    }
    
    public Product getProduct(String productId) {
        // 1. 尝试从本地缓存获取
        try {
            return localCache.get(productId);
        } catch (Exception e) {
            // 本地缓存未命中，继续查询Redis
            return loadFromRedis(productId);
        }
    }
    
    private Product loadFromRedis(String productId) {
        String key = "product:" + productId;
        
        // 2. 从Redis获取
        Product product = redisTemplate.opsForValue().get(key);
        if (product != null) {
            return product;
        }
        
        // 3. 从数据库加载
        product = productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
        
        // 4. 存入Redis，设置过期时间和随机延迟（防止缓存雪崩）
        int randomExpiry = 3600 + new Random().nextInt(300);
        redisTemplate.opsForValue().set(key, product, randomExpiry, TimeUnit.SECONDS);
        
        return product;
    }
    
    /**
     * 更新产品信息时，同步更新缓存
     */
    public void updateProduct(Product product) {
        // 更新数据库
        productRepository.save(product);
        
        // 更新Redis缓存
        String key = "product:" + product.getId();
        redisTemplate.opsForValue().set(key, product, 1, TimeUnit.HOURS);
        
        // 清除本地缓存
        localCache.invalidate(product.getId());
    }
}
```

### 性能优化建议

#### 1. 优化键设计

良好的键命名模式对Redis性能有很大影响：

```java
// 推荐的键命名模式：对象类型:ID:属性
redisTemplate.opsForValue().set("user:1001:profile", userProfile);
redisTemplate.opsForHash().putAll("user:1001:data", userData);
redisTemplate.opsForList().leftPushAll("user:1001:logs", userLogs);
```

#### 2. 批量操作优化

尽可能使用批量操作，减少网络开销：

```java
// 不推荐：多次单独操作
for (String key : keys) {
    redisTemplate.opsForValue().get(key);
}

// 推荐：使用批量操作
List<Object> values = redisTemplate.opsForValue().multiGet(keys);

// 使用管道批量处理
List<Object> results = redisTemplate.executePipelined(new SessionCallback<>() {
    @Override
    public <K, V> Object execute(RedisOperations<K, V> operations) {
        for (String key : keys) {
            operations.opsForValue().get(key);
        }
        return null;
    }
});
```

#### 3. 使用合适的数据结构

```java
// 场景：用户点赞记录
// 不推荐：使用单独的键存储每个用户的点赞状态
redisTemplate.opsForValue().set("post:1001:liked_by_user:2001", "1");

// 推荐：使用集合存储所有点赞用户
redisTemplate.opsForSet().add("post:1001:liked_by_users", "2001");
// 判断用户是否点赞
Boolean hasLiked = redisTemplate.opsForSet().isMember("post:1001:liked_by_users", "2001");
```

#### 4. 合理设置过期时间

```java
// 给不同类型的数据设置不同的过期时间
// 会话信息 - 短期
redisTemplate.opsForValue().set("session:"+token, sessionData, 30, TimeUnit.MINUTES);

// 用户基本信息 - 中期
redisTemplate.opsForValue().set("user:"+userId, userData, 24, TimeUnit.HOURS);

// 系统配置 - 长期
redisTemplate.opsForValue().set("config:"+key, configValue, 7, TimeUnit.DAYS);

// 添加随机过期时间，防止缓存雪崩
int baseExpiry = 3600; // 基础过期时间1小时
int randomAdditional = new Random().nextInt(300); // 随机添加0-5分钟
redisTemplate.opsForValue().set(key, value, baseExpiry + randomAdditional, TimeUnit.SECONDS);
```

## 第四章：常见问题排查

### 连接池配置异常

#### 问题表现
- 应用启动缓慢
- 连接超时错误
- 连接池资源耗尽

#### 排查和解决方案

**配置检查**：

```java
@Configuration
public class RedisConnectionPoolConfig {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        // 创建Lettuce连接工厂的配置
        LettucePoolingClientConfiguration poolConfig = LettucePoolingClientConfiguration.builder()
            .commandTimeout(Duration.ofSeconds(5))  // 命令超时时间
            .poolConfig(getPoolConfig())           // 连接池配置
            .build();
        
        // Redis服务器配置
        RedisStandaloneConfiguration redisConfig = new RedisStandaloneConfiguration();
        redisConfig.setHostName("localhost");
        redisConfig.setPort(6379);
        
        return new LettuceConnectionFactory(redisConfig, poolConfig);
    }
    
    private GenericObjectPoolConfig getPoolConfig() {
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMaxTotal(50);               // 最大连接数
        poolConfig.setMaxIdle(20);                // 最大空闲连接数
        poolConfig.setMinIdle(5);                 // 最小空闲连接数
        poolConfig.setTestOnBorrow(true);         // 获取连接时测试连接有效性
        poolConfig.setTestOnReturn(false);        // 归还连接时不测试连接有效性
        poolConfig.setTestWhileIdle(true);        // 空闲时测试连接有效性
        poolConfig.setTimeBetweenEvictionRunsMillis(60000); // 空闲连接检测间隔
        poolConfig.setMaxWaitMillis(3000);        // 获取连接最大等待时间
        return poolConfig;
    }
}
```

**常见问题和修复**：

1. **连接超时**：检查网络连通性，增加命令超时时间
   ```java
   .commandTimeout(Duration.ofSeconds(10))  // 增加超时时间
   ```

2. **连接数不足**：增加最大连接数
   ```java
   poolConfig.setMaxTotal(100);  // 增加最大连接数
   ```

3. **连接泄漏**：开启连接池监控和日志
   ```java
   poolConfig.setJmxEnabled(true);  // 启用JMX监控
   ```

### 序列化不一致问题

#### 问题表现
- 存取数据时出现类型转换异常
- 跨应用交互时无法读取对方存储的数据
- 数据格式在Redis中显示错乱

#### 排查和解决方案

**确保序列化器统一**：

```java
@Configuration
public class ConsistentRedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建RedisTemplate
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // 使用Jackson2JsonRedisSerializer作为默认序列化器
        Jackson2JsonRedisSerializer<Object> jacksonSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        
        // 配置Jackson ObjectMapper
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(om.getPolymorphicTypeValidator(), 
                                ObjectMapper.DefaultTyping.NON_FINAL);
        jacksonSerializer.setObjectMapper(om);
        
        // 设置所有序列化器保持一致
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jacksonSerializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jacksonSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

**跨应用兼容性方案**：

```java
// 为不同类型的数据创建专用RedisTemplate
@Bean
public RedisTemplate<String, User> userRedisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, User> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);
    
    // 使用特定类型的序列化器，避免包含类型信息
    Jackson2JsonRedisSerializer<User> serializer = new Jackson2JsonRedisSerializer<>(User.class);
    
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(serializer);
    template.afterPropertiesSet();
    return template;
}
```

### 事务执行失效场景

#### 问题表现
- 事务中的命令似乎被执行，但没有生效
- `exec()` 返回的结果列表与预期不符
- 抛出事务相关的异常

#### 排查和解决方案

**事务执行正确方式**：

```java
// 正确的事务执行方式
List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
    @Override
    public <K, V> List<Object> execute(RedisOperations<K, V> operations) {
        operations.multi();
        
        // 事务中的操作仅被排队，不会立即执行
        operations.opsForValue().set("tx:key1", "value1");
        operations.opsForValue().get("tx:key1"); // 注意：此操作不会返回实际值，而是在exec后
        
        // 执行事务并获取结果
        return operations.exec();
    }
});

// 事务结果处理
if (txResults != null) {
    // txResults.get(0) 对应第一个操作的结果
    // txResults.get(1) 对应第二个操作的结果
}
```

**事务失效常见原因**：

1. **Redis集群不支持跨分片事务**：
   ```java
   // 确保事务中操作的键在同一个分片上
   // 例如：使用相同的哈希标签
   redisTemplate.opsForValue().set("{user:1001}:name", "张三");
   redisTemplate.opsForValue().set("{user:1001}:age", "30");
   ```

2. **命令使用不当**：
   ```java
   // 错误：事务内查询的结果在事务外不可用
   operations.multi();
   String value = operations.opsForValue().get("key"); // 在事务内，这里实际不会执行查询
   operations.opsForValue().set("newKey", value.toUpperCase()); // 可能出现NPE，因为value为null
   operations.exec();
   
   // 正确：先查询，再执行事务
   String value = operations.opsForValue().get("key");
   operations.multi();
   operations.opsForValue().set("newKey", value.toUpperCase());
   operations.exec();
   ```

### 集群环境特殊处理

#### 问题表现
- 跨槽位命令执行失败
- 批量操作部分失败
- 某些高级功能在集群下不可用

#### 排查和解决方案

**集群配置示例**：

```java
@Configuration
public class RedisClusterConfig {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        // 集群节点配置
        RedisClusterConfiguration clusterConfig = new RedisClusterConfiguration();
        clusterConfig.setClusterNodes(Arrays.asList(
            new RedisNode("redis1", 6379),
            new RedisNode("redis2", 6379),
            new RedisNode("redis3", 6379)
        ));
        clusterConfig.setMaxRedirects(3); // 请求重定向次数
        
        // 集群连接池配置
        LettuceClientConfiguration clientConfig = LettuceClientConfiguration.builder()
            .commandTimeout(Duration.ofSeconds(5))
            .readFrom(ReadFrom.REPLICA_PREFERRED) // 优先从从节点读取
            .build();
        
        return new LettuceConnectionFactory(clusterConfig, clientConfig);
    }
}
```

**集群环境下特殊处理**：

1. **关键字哈希标签**：确保相关键在同一分片上
   ```java
   // 使用花括号内的部分作为哈希标签
   redisTemplate.opsForValue().set("{user:1001}:profile", profile);
   redisTemplate.opsForValue().set("{user:1001}:settings", settings);
   ```

2. **批量操作适配**：
   ```java
   // 集群环境下安全的批量操作
   Map<String, String> keyValues = getBatchData();
   
   // 按哈希槽分组处理
   Map<Integer, Map<String, String>> slotGroupedData = groupBySlot(keyValues);
   
   // 分组执行批量操作
   for (Map<String, String> slotData : slotGroupedData.values()) {
       redisTemplate.opsForValue().multiSet(slotData);
   }
   ```

3. **脚本执行**：
   ```java
   // 确保脚本中使用的键使用相同的哈希标签
   String script = "return redis.call('set', KEYS[1], ARGV[1])";
   redisTemplate.execute(
       RedisScript.of(script, String.class),
       Collections.singletonList("{user:1001}:lock"),
       "locked"
   );
   ```

## 版本兼容性说明（Spring Boot 2.x vs 3.x）

### 主要变化

| 特性 | Spring Boot 2.x | Spring Boot 3.x |
|-----|----------------|----------------|
| **Redis客户端** | 默认Lettuce，支持Jedis | 默认Lettuce，支持Jedis |
| **Java版本要求** | Java 8+ | Java 17+ |
| **序列化器** | 默认JDK序列化 | 默认JDK序列化 |
| **反应式支持** | ReactiveRedisTemplate | 增强的ReactiveRedisTemplate |
| **依赖路径** | spring-boot-starter-data-redis | 不变 |
| **自动配置** | RedisAutoConfiguration | 不变，增加了更多细节配置 |
| **SSL支持** | 有限支持 | 全面支持 |
| **Jakarta命名空间** | javax.\* | jakarta.\* |

### 版本迁移注意事项

1. **Jakarta EE迁移**：
   ```java
   // Spring Boot 2.x (javax命名空间)
   import javax.annotation.PostConstruct;
   
   // Spring Boot 3.x (jakarta命名空间)
   import jakarta.annotation.PostConstruct;
   ```

2. **RedisJSON支持**：
   ```java
   // Spring Boot 3.x 中可以更简单地整合RedisJSON模块
   @Bean
   public RedisJSONClient redisJSONClient(RedisConnectionFactory connectionFactory) {
       return RedisJSONClient.create(connectionFactory);
   }
   ```

3. **Redis Stack支持**：
   ```java
   // Spring Boot 3.x 更好地支持Redis Stack功能
   @Bean
   public RedisStackCommands redisStackCommands(RedisConnectionFactory connectionFactory) {
       RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
       redisTemplate.setConnectionFactory(connectionFactory);
       return new RedisStackCommandsImpl(redisTemplate.getConnectionFactory());
   }
   ```

4. **配置方式**：
   ```yaml
   # application.properties 或 application.yml
   
   # Spring Boot 2.x
   spring.redis.host=localhost
   spring.redis.port=6379
   spring.redis.timeout=2000
   
   # Spring Boot 3.x (新增配置选项)
   spring.redis.host=localhost
   spring.redis.port=6379
   spring.redis.timeout=2000
   spring.redis.connect-timeout=2000
   spring.redis.client-name=my-app
   spring.redis.client-type=lettuce
   ```

通过以上全面的RedisTemplate知识体系，您可以在各种Spring应用中高效、安全地使用Redis，并针对不同场景选择最优的实现方案。