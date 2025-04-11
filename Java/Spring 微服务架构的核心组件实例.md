# 微服务架构实战教程：Nacos、Gateway与Redis缓存

本教程详细介绍了微服务架构中三个核心组件的使用：Nacos注册中心、Spring Cloud Gateway网关和Redis缓存。通过实际案例，带您逐步掌握这些技术的应用。

![[Spring 微服务架构的核心组件实例.png]]
这个图片似乎可以作为微服务的框架

## 1. [[Spring 组件 Nacos]]

## 2. Spring Cloud Gateway网关
### 2.1 Gateway简介
Spring Cloud Gateway是Spring Cloud生态系统中的API网关服务，基于Spring 5、Spring Boot 2和Project Reactor等技术。

Gateway的设计目标是提供一种简单有效的API路由管理方式，并提供强大的过滤器功能，如熔断、限流、重试等。它基于WebFlux框架实现，底层使用高性能的Netty作为通讯框架，性能优于传统的Servlet架构。
**架构图**：
![Gateway架构|850](image-20230624161556300.png)
### 2.2 三大核心概念
#### 2.2.1 Route（路由）
路由是Gateway的基本构建模块，由ID、目标URI、一系列断言和过滤器组成。如果断言为true，则匹配该路由。

#### 2.2.2 Predicate（断言）
参考Java 8的`java.util.function.Predicate`接口，开发人员可以匹配HTTP请求中的任何内容，如请求头或请求参数。

#### 2.2.3 Filter（过滤器）
指Spring框架中GatewayFilter的实例，可以在请求被路由前或之后对请求进行修改。

通过过滤器，可以实现权限控制、流量监控、统一日志处理等功能：
![过滤器工作流程](image-20230624164230054.png)
### 2.3 工作流程
![Gateway工作流程1](image-20230721092104682.png)

**工作流程说明**：
1. 客户端向Spring Cloud Gateway发出请求
2. Gateway Handler Mapping中找到与请求匹配的路由，将请求发送到Gateway Web Handler
3. Handler通过指定的过滤器链将请求发送给目标服务执行业务逻辑
4. 过滤器分为"pre"和"post"两种类型：
   - "pre"类型：在请求发送到微服务前执行，可做参数校验、权限校验、流量监控等
   - "post"类型：在微服务响应后执行，可修改响应内容、响应头，输出日志等

### Gateway vs 拦截器的区别

**相同点**：

- 都可以进行请求拦截和处理 **不同点**：

1. **部署位置**：
    - 拦截器：嵌入在单个应用内部
    - 网关：独立部署的服务，所有请求的统一入口
2. **功能范围**：
    - 拦截器：主要处理单应用内的请求过滤
    - 网关：处理所有微服务的共性问题
3. **技术实现**：
    - Gateway基于异步非阻塞的WebFlux，性能更高
    - 拦截器通常基于同步阻塞的Servlet

### Gateway的核心价值

1. **统一入口**：
    - 客户端只需与网关交互，无需关心后端服务细节
    - 简化客户端调用，隐藏后端结构
2. **智能路由**：
    - 根据请求路径、头信息、参数等动态决定路由目标
    - 支持灰度发布、A/B测试
3. **横切关注点集中处理**：
    - 认证授权：统一鉴权，无需每个服务实现
    - 安全控制：防御常见攻击如XSS、CSRF
    - 流量控制：限流、熔断，保护后端服务
    - 监控统计：请求计数、性能监测
    - 日志记录：统一日志格式和存储
4. **协议转换**：
    - 外部HTTP请求转为内部RPC调用
    - 处理复杂的协议适配 **实际应用场景**： 假设有10个微服务，每个都需要实现认证、限流、日志：

- **无网关**：这些功能需在每个服务中重复实现，代码冗余且难以统一维护
- **有网关**：这些功能集中在网关实现一次，后端服务专注于业务逻辑
## 3. 网关服务搭建
### 3.1 网关服务说明
在微服务架构中，前端系统通过网关访问后端微服务，网关作为统一入口：
![网关服务架构](image-20230704225006031.png)
### 3.2 服务网关搭建
**实现步骤**：
1. 在父项目下创建`spzx-server-gateway`模块，添加依赖：
   ```xml
   <dependencies>
       <dependency>
           <groupId>com.atguigu.spzx</groupId>
           <artifactId>common-util</artifactId>
           <version>1.0-SNAPSHOT</version>
       </dependency>
       <dependency>
           <groupId>com.atguigu.spzx</groupId>
           <artifactId>spzx-model</artifactId>
           <version>1.0-SNAPSHOT</version>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-gateway</artifactId>
       </dependency>
       <!-- loadbalancer依赖 -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-loadbalancer</artifactId>
       </dependency>
       <!-- 服务注册 -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
       <!-- 服务保护组件 -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
       </dependency>
   </dependencies>
   ```
2. 创建配置文件：
   **application.yml**：
   ```yml
   spring:
     profiles:
       active: dev
   ```
   **application-dev.yml**：
   ```yml
   server:
     port: 8500
   spring:
     application:
       name: spzx-server-gateway
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
       gateway:
         discovery:
           locator:
             enabled: true
         globalcors:
           cors-configurations:
             '[/**]':
               allowedOriginPatterns: "*"
               # 允许请求中携带的头信息
               allowedHeaders: "*"
               # 运行跨域的请求方式
               allowedMethods: "*"
               # 跨域检测的有效期,单位s
               maxAge: 36000
         routes:
           - id: service-product
             uri: lb://service-product
             predicates:
               - Path=/*/product/**
   ```
3. 创建启动类：
   ```java
   // 包路径: com.atguigu.spzx.gateway
   @SpringBootApplication
   public class GatewayApplication {
       public static void main(String[] args) {
           SpringApplication.run(GatewayApplication.class, args);
       }
   }
   ```
4. 导入日志配置文件`logback-spring.xml`，修改输出路径：
   ```xml
   <property name="log.path" value="D://logs//spzx-server-gateway//logs" />
   ```
### 3.3 服务网关测试
1. 注释掉service-product微服务Controller上的`@CrossOrigin`注解（由网关统一处理跨域）
2. 配置后端地址，指向网关：
   ![配置后端地址](image-20230704230415772.png)
## 4. Redis缓存
对于不经常变动的数据（如分类数据），可以使用缓存提高页面加载速度。
### 4.1 直接使用Redis缓存
**步骤**：
1. 在service-product微服务中添加Redis依赖：
   ```xml
   <!-- redis的起步依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```
2. 配置Redis连接：
   ```yaml
   spring:
     # Redis的相关配置
     data:
       redis:
         host: localhost
         port: 6379
   ```
3. 改造`CategoryServiceImpl`的`findOneCategory`方法：
   ```java
   @Autowired
   private RedisTemplate<String, String> redisTemplate;
   @Override
   public List<Category> findOneCategory() {
       // 从Redis缓存中查询所有的一级分类数据
       String categoryListJSON = redisTemplate.opsForValue().get("category:one");
       if(!StringUtils.isEmpty(categoryListJSON)) {
           List<Category> categoryList = JSON.parseArray(categoryListJSON, Category.class);
           log.info("从Redis缓存中查询到了所有的一级分类数据");
           return categoryList;
       }
       // 从数据库查询
       List<Category> categoryList = categoryMapper.findOneCategory();
       log.info("从数据库中查询到了所有的一级分类数据");
       // 存入Redis缓存，有效期7天
       redisTemplate.opsForValue().set("category:one", JSON.toJSONString(categoryList), 7, TimeUnit.DAYS);
       return categoryList;
   }
   ```
### 4.2 Spring Cache缓存框架
#### 4.2.1 介绍
Spring Cache是一个抽象的缓存框架，只需添加注解即可实现缓存功能，简化了业务代码中的缓存操作。它通过`CacheManager`接口统一不同的缓存技术实现。
常见的CacheManager实现：

| **CacheManager**    | **描述**                    |
| ------------------- | ------------------------- |
| EhCacheCacheManager | 使用EhCache作为缓存技术           |
| GuavaCacheManager   | 使用Google的GuavaCache作为缓存技术 |
| RedisCacheManager   | 使用Redis作为缓存技术             |
#### 4.2.2 常用注解
| **注解**         | **说明**                            |
| -------------- | --------------------------------- |
| @EnableCaching | 开启缓存注解功能                          |
| @Cacheable     | 方法执行前检查缓存，有则返回缓存数据，无则执行方法并将结果放入缓存 |
| @CachePut      | 将方法返回值放入缓存                        |
| @CacheEvict    | 从缓存中删除一条或多条数据                     |
**@Cacheable注解**：
```java
@Cacheable(value = "userCache", key = "#userId")
public User findById(Long userId) {
    log.info("用户数据查询成功...");
    User user = new User();
    user.setAge(23);
    user.setUserName("尚硅谷");
    return user;
}
```
**@CachePut注解**：
```java
@CachePut(value = "userCache", key = "#user.userName")
public User saveUser(User user) {
    log.info("用户数据保存成功...");
    return user;
}
```
**@CacheEvict注解**：
```java
@CacheEvict(value = "userCache", key = "#userId")
public void deleteById(Long userId) {
    log.info("用户数据删除成功...");
}
```
**key的常见写法**：
- `#user.id`：使用方法参数user的id属性作为key
- `#result.id`：使用方法返回值的id属性作为key
### 4.3 使用Spring Cache缓存分类数据
**需求**：使用Spring Cache框架缓存所有分类数据
**步骤**：
1. 添加Spring Cache依赖：
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-cache</artifactId>
   </dependency>
   ```
2. 配置Redis的key序列化器：
   ```java
   // 包路径: com.atguigu.spzx.cache.config
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
3. 在启动类上添加`@EnableCaching`注解，开启缓存支持
4. 在`CategoryServiceImpl`的`findAllCategory`方法上添加缓存注解：
   ```java
   @Cacheable(value = "category", key = "'all'")
   public List<Category> findAllCategory() {
       // 方法实现...
       return oneCategoryList;
   }
   ```
5. 启动服务进行测试，观察缓存是否生效
这样，当第一次请求分类数据时会从数据库查询，之后的请求会直接从Redis缓存中获取，大大提高了系统的响应速度和性能。
## 总结
本教程详细介绍了微服务架构中的三个重要组件：
1. **Nacos注册中心**：实现服务的注册与发现，便于微服务间的调用
2. **Spring Cloud Gateway**：作为统一入口，处理路由、跨域、过滤等共性问题
3. **Redis缓存**：通过直接使用和Spring Cache注解两种方式提高系统性能
通过这些组件的协同工作，可以构建出高性能、高可用的微服务架构系统。
希望本教程对您理解和应用这些技术有所帮助！