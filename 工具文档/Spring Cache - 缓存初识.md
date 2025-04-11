
### 介绍

**Spring Cache**是一个框架，实现了基于注解的缓存功能，只需要简单地加一个注解，就能实现缓存功能，**大大简化我们在业务中操作缓存的代码**。

Spring Cache只是提供了一层抽象，底层可以切换不同的cache实现。具体就是通过**CacheManager**接口来统一不同的缓存技术。CacheManager是Spring提供的各种缓存技术抽象接口。

针对不同的缓存技术需要实现不同的CacheManager：

| **CacheManager**    | **描述**                           |
| ------------------- | ---------------------------------- |
| EhCacheCacheManager | 使用EhCache作为缓存技术            |
| GuavaCacheManager   | 使用Google的GuavaCache作为缓存技术 |
| RedisCacheManager   | 使用Redis作为缓存技术              |



### 注解

在SpringCache中提供了很多缓存操作的注解，常见的是以下的几个：

| **注解**         | **说明**                                                         |
| -------------- | -------------------------------------------------------------- |
| @EnableCaching | 开启缓存注解功能                                                       |
| @Cacheable     | 在方法执行前spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放到缓存中 |
| @CachePut      | 将方法的返回值放到缓存中                                                   |
| @CacheEvict    | 将一条或多条数据从缓存中删除                                                 |

在spring boot项目中，使用缓存技术只需在项目中导入相关缓存技术的依赖包，并在启动类上使用**@EnableCaching**开启缓存支持即可。使用Redis作为缓存技术，只需要导入Spring data Redis的maven坐标即可。



#### @Cacheable

1、作用: 在方法执行前，spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放到缓存中

2、value: 缓存的名称，每个缓存名称下面可以有多个key

3、key: 缓存的key  ----------> 支持Spring的表达式语言SPEL语法

```java
@Override
@Cacheable(value = "userCache" , key = "#userId")
public User findById(Long userId) {
    log.info("用户数据查询成功...");
    User user = new User() ;
    user.setAge(23);
    user.setUserName("尚硅谷");
    return user;
}
```



#### @CachePut

作用: 将方法返回值，放入缓存

value: 缓存的名称, 每个缓存名称下面可以有很多key

key: 缓存的key  ----------> 支持Spring的表达式语言SPEL语法

当前UserController的save方法是用来保存用户信息的，我们希望在该用户信息保存到数据库的同时，也往缓存中缓存一份数据，我们可以在save方法上加上注解 @CachePut，用法如下： 

```java
@CachePut(value = "userCache", key = "#user.userName")
public User saveUser(User user) {
    log.info("用户数据保存成功...");
    return user ;
}
```

**key的写法如下**： 

#user.id : #user指的是方法形参的名称, id指的是user的id属性 , 也就是使用user的id属性作为key ;

#user.userName: #user指的是方法形参的名称, name指的是user的name属性 ,也就是使用user的name属性作为key ;

#result.id : #result代表方法返回值，该表达式 代表以返回对象的id属性作为key ；

#result.userName: #result代表方法返回值，该表达式 代表以返回对象的name属性作为key ；



#### @CacheEvict

作用: 清理指定缓存

value: 缓存的名称，每个缓存名称下面可以有多个key

key: 缓存的key  ----------> 支持Spring的表达式语言SPEL语法

当我们在删除数据库user表的数据的时候,我们需要删除缓存中对应的数据,此时就可以使用**@CacheEvict**注解, 具体的使用方式如下: 

```java
@CacheEvict(value = "userCache" , key = "#userId")
public void deleteById(Long userId) {
    log.info("用户数据删除成功...");
}
```



## 缓存所有分类数据

需求：给查询所有的分类数据添加查询缓存，使用Spring Cache框架

步骤：

1、在service-product服务中的pom.xml文件中添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

2、配置Redis的key的序列化器

```java
// com.atguigu.spzx.cache.config;
@Configuration
public class RedisConfig {

    @Bean
    public CacheManager cacheManager(LettuceConnectionFactory connectionFactory) {

        //定义序列化器
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();


        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                //过期时间600秒
                .entryTtl(Duration.ofSeconds(600))
                // 配置序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(stringRedisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(genericJackson2JsonRedisSerializer));

        RedisCacheManager cacheManager = RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();

        return cacheManager;
    }

}
```

3、在启动类上添加 **@EnableCaching** 注解

4、在CategoryServiceImpl类中的 findCategoryTree 方法上添加 **@Cacheable** 注解

```java
@Cacheable(value = "category" , key = "'all'")
public List<Category> findAllCategory() {
..
    return oneCategoryList;
}
```

5、启动程序进行测试
