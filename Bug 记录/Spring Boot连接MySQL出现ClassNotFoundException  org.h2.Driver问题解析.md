## 问题现象
- 运行时报错：`java.lang.ClassNotFoundException: org.h2.Driver`
- 确认项目未显式引入h2数据库依赖

## 根本原因
1. Spring Boot自动配置机制错误激活
2. 系统自动创建了h2数据库连接池
3. 未正确加载Druid+MySQL配置

## 解决方案
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean("dataSource")
    @ConfigurationProperties(prefix = "spring.datasource.druid")
    public DruidDataSource dataSource() {
        return new DruidDataSource();
    }
}
```

## 配置验证要点
1. 确保pom.xml包含正确依赖：
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.16</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```
2. application.yml配置示例：
```yaml
spring:
  datasource:
    druid:
      url: jdbc:mysql://localhost:3306/dbname
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
```

## 原理说明
通过显式声明DataSource Bean，覆盖Spring Boot的自动配置逻辑，强制使用指定连接池配置。