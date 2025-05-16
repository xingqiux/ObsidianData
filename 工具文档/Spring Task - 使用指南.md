## 1. 概述

Spring Task是Spring框架提供的任务调度工具,它提供了轻量级的定时任务解决方案。相比Quartz,它更加简单易用,适合单体应用中的定时任务需求。

主要特点:
- 基于Spring框架,无需额外依赖
- 支持注解配置,使用简单
- 支持cron表达式
- 支持固定延时调度
- 可以动态调整任务执行时间

## 2. 环境准备

### 2.1 创建Spring Boot项目

使用Spring Initializr创建项目,添加以下依赖:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

### 2.2 启用定时任务

在启动类上添加@EnableScheduling注解:

```java
@SpringBootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 3. 创建定时任务

### 3.1 基本用法

使用@Scheduled注解创建定时任务:

```java
@Component
public class ScheduledTasks {
    
    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("当前时间: {}", new Date());
    }
}
```

### 3.2 调度方式

@Scheduled支持多种调度方式:

1. fixedRate: 固定速率执行
```java
@Scheduled(fixedRate = 5000) // 每5秒执行一次
```

2. fixedDelay: 固定延时执行
```java
@Scheduled(fixedDelay = 5000) // 上一次执行完成后延迟5秒再执行
```

3. cron表达式
```java
@Scheduled(cron = "0 0/1 * * * ?") // 每分钟执行一次
```

### 3.3 Cron表达式详解

cron表达式格式: 秒 分 时 日 月 周

常用示例:
- "0 0 12 * * ?" : 每天12点触发
- "0 15 10 ? * *" : 每天10:15触发
- "0 15 10 * * ?" : 每天10:15触发
- "0 15 10 * * ? 2023" : 2023年每天10:15触发

这些字段的取值范围如下所示：

| 字段             | 允许的值                     | 允许的特殊字符 |
| ---------------- | ---------------------------- | -------------- |
| 秒（Seconds）    | 0-59                         | , - * /        |
| 分（Minutes）    | 0-59                         | , - * /        |
| 时（Hours）      | 0-23                         | , - * /        |
| 日（DayofMonth） | 1-31                         | , - * ? / L W  |
| 月（Month）      | 1-12 or JAN-DEC              | , - * /        |
| 周（DayofWeek）  | 0-6 or SUN-SAT (0表示星期天) | , - * ? / L #  |
| 年（Year）       | 留空或1970-2099              | , - * /        |

注：如日和月同时维护，例如：3 50 18 15 2 4，需要注意二月的星期四，不一定是15号，此时星期和日是有冲突的，通常需要舍掉一个，被舍掉的参

数用**?**占位

Cron表达式的时间字段除允许设置数值外，还可使用一些特殊的字符，提供列表、范围、通配符等功能。

可查看阿里云开发者社区手册：https://developer.aliyun.com/article/942392

## 4. 高级配置

### 4.1 动态调整任务

实现SchedulingConfigurer接口:

```java
@Configuration
public class SchedulingConfig implements SchedulingConfigurer {
    
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addTriggerTask(
            () -> {
                // 任务逻辑
            },
            triggerContext -> {
                // 动态获取下次执行时间
                String cron = getCronFromDB(); // 从数据库获取cron
                CronTrigger trigger = new CronTrigger(cron);
                return trigger.nextExecutionTime(triggerContext);
            }
        );
    }
}
```

### 4.2 异步执行

启用异步支持:

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(2);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("GithubLookup-");
        executor.initialize();
        return executor;
    }
}
```

在任务上添加@Async注解:

```java
@Async
@Scheduled(fixedRate = 5000)
public void asyncTask() {
    // 异步执行的任务
}
```

### 4.3 异常处理

```java
@Scheduled(fixedRate = 5000)
public void taskWithErrorHandler() {
    try {
        // 任务逻辑
    } catch (Exception e) {
        // 异常处理
        log.error("任务执行出错", e);
    }
}
```

## 5. 最佳实践

### 5.1 避免任务重复执行

在分布式环境中使用分布式锁:

```java
@Scheduled(fixedRate = 5000)
public void taskWithLock() {
    RLock lock = redissonClient.getLock("scheduledTask");
    try {
        // 尝试获取锁
        if (lock.tryLock(0, 5, TimeUnit.SECONDS)) {
            try {
                // 任务逻辑
            } finally {
                lock.unlock();
            }
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}
```

### 5.2 配置线程池

```java
@Configuration
public class SchedulingConfig implements SchedulingConfigurer {
    
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskScheduler());
    }
    
    @Bean(destroyMethod = "shutdown")
    public Executor taskScheduler() {
        return Executors.newScheduledThreadPool(10);
    }
}
```

## 6. 注意事项

1. fixedRate和fixedDelay的区别
- fixedRate: 固定速率执行,不管任务执行多长时间
- fixedDelay: 在上次任务完成后,延迟指定时间再执行

2. 任务超时处理
- 考虑使用异步执行
- 设置合理的执行间隔
- 添加任务超时监控

3. 避免任务堆积
- 合理配置线程池大小
- 监控任务执行时间
- 及时处理异常情况

## 7. 总结

Spring Task适用于:
- 单体应用的定时任务
- 简单的调度需求
- 不需要任务持久化的场景

如果需要更复杂的功能,建议使用:
- Quartz: 支持集群、持久化
- XXL-Job: 分布式任务调度平台
- Elastic-Job: 分布式调度解决方案

## 8. 完整示例

GitHub仓库地址: [Spring Task Demo](https://github.com/your-repository)

这个教程涵盖了Spring Task的主要功能和使用方法。如果你有任何问题,欢迎讨论。