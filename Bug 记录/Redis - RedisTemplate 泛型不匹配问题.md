
## 介绍

开发 Aurain 的验证码系统的时候，出现了明明存入了数据到 redis 中却无法根据前端的值查询到数据，通过分析之后发现是出现了 “泛型不匹配问题”

## 快速记录

不同的泛型类型可能使用不同的序列化器
无指定泛型的 RedisTemplate 默认使用JdkSerializationRedisSerializer 而
- RedisTemplate<String,String> 可能配置使用 StringRedisSerializer
- 这导致相同的键在 Redis 中实际上存储的是不同的值
1. **获取的时候找不到键**：
    
    - 由于序列化方式不同，即使使用了相同的字符串构造键名，在 Redis 中实际的键也不相同
    - 这就导致无论输入什么验证码都无法找到对应的值

## 解决方案

1. **统一 RedisTemplate 配置**：
    
    JAVA
    
    `// 在 ValidateCodeServiceImpl.java 中也使用带泛型的 RedisTemplate @Autowired private RedisTemplate<String, String> redisTemplate;`
    
2. **确保一致的序列化方式**：
    
    - 创建一个 RedisConfig 配置类，确保所有 RedisTemplate 使用相同的序列化器


问题的核心在于 Redis 的序列化方式不一致，导致无法正确查询存储的验证码数据。
## 解释

![[泛型不匹配问题.png]]![[泛型不匹配问题1.png]]