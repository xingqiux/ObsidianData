
# 概述

## 主要功能

### 服务注册与发现
![[Spring Cloud Alibaba Nacos 注册配置中心 1.png|575]]
注册中心的主要作用是作为协调者，主要作用是将分开的微服务项目通过服务注册和发现模型，实现项目的**跨进程跨主机进行通信**，实现**相互通信**和**相互调用**

微服务启动过程中，将自身的服务名称，IP 地址，端口号发送到注册中心。

微服务相互调用的时候，通过实例的 IP 地址和端口号进行调用

### 配置管理

主要的功能是动态的更新各个微服务的配置信息

## 接入 Nacos 配置中心

### 导入依赖

因为这个文档本质上是使用 Spring Cloud ，所以需要提前导入 Spring Cloud 使用的依赖，然后导入 Nacos 配置中心的依赖

```html
<!-- Nacos 服务发现（必须） -->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>  
  
<!-- Nacos 配置中心（按需添加） -->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>  
</dependency>  
  
  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-bootstrap</artifactId>  
</dependency>
```
**注意**：版本 [2.1.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 2.1.x 版本。版本 [2.0.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 2.0.x 版本，版本 [1.5.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 1.5.x 版本。

更多版本对应关系参考：[版本说明 Wiki](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

2. 在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名
```json
spring.application.name=ziyu-cloud-nacos-config-user  # 这个名称就是配置的名称，在下方有提及
spring.cloud.nacos.config.server-addr=127.0.0.1:8848  
spring.cloud.nacos.config.refresh.enabled=true
```

这里配置`spring.cloud.nacos.config.extension-configs=yaml ` 是可选，默认是使用 properties 配置

![[Spring Cloud Alibaba Nacos 注册配置中心 1.png]]
说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

**这一部分是超级重点，这个配置的文件必须要设定清楚，不然无法自动更新！！！！！**

### 配置 Nacos

![[Spring Cloud Alibaba Nacos 注册配置中心-1 1.png]]

接下是应用配置，这里主要部分是

```json
spring:  
  config:  
    import:  
      - nacos:ziyu-cloud-nacos-config-user.properties
```

导入的是 Ncos 中存储的信息
![[Spring Cloud Alibaba Nacos 注册配置中心-2 1.png|500]]
注意这里 Data ID 中的 properties 前面必须是 spring.application.name

### 导入配置信息

导入配置信息的 controller 代码就是最基础的导入配置文件中代码即可，加上 `@RefreshScope ` 自动刷新即可 

```java
  
@RestController  
@RequestMapping("/nacos/bean")  
@RefreshScope  
public class ConfigControllerExample {  
  
    @Value("${order.user.name}")  
    private String username;  
  
    @Value("${order.user.age}")  
    private String userage;  
  
    @GetMapping("/1")  
    public Map<String,String> getNacosConfig() {  
        Map<String,String> map = new HashMap<>();  
        map.put("username",username);  
        map.put("userage",userage);  
        return map;  
    }  
  
}
```

### 运行测试代码

```java
@SpringBootApplication  
public class Main {  
    public static void main(String[] args) {  
        SpringApplication.run(Main.class, args);  
    }  
}
```

按照以上内容运行即可得到一个最基础的 demo 代码用语 nacos 的配置中心体现

## 接入 Nacos 注册中心


本节通过实现一个简单的 `echo service` 演示如何在您的 Spring Cloud 项目中启用 Nacos 的服务发现功能，如下图示:
![[Spring Cloud Alibaba Nacos 注册配置中心-3 1.png]]
完整示例代码请参考：[nacos-spring-cloud-discovery-example](https://github.com/nacos-group/nacos-examples/tree/master/nacos-spring-cloud-example/nacos-spring-cloud-discovery-example)

### 1. 添加依赖：

```html
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${latest.version}</version>
</dependency>
```

**注意**：版本 [2.1.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-discovery) 对应的是 Spring Boot 2.1.x 版本。版本 [2.0.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-discovery) 对应的是 Spring Boot 2.0.x 版本，版本 [1.5.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-discovery) 对应的是 Spring Boot 1.5.x 版本。

更多版本对应关系参考：[版本说明 Wiki](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

### 2. 配置服务提供者，从而服务提供者可以通过 Nacos 的服务注册发现功能将其服务注册到 Nacos server 上。

i. 在 `application.properties` 中配置 Nacos server 的地址：

```json
server:  
  port: 8890  
  
spring:  
  application:  
    name: service-provider  
  
  cloud:  
    nacos:  
      server-addr: 127.0.0.1:8848
```

ii. 通过 Spring Cloud 原生注解 `@EnableDiscoveryClient` 开启服务注册发现功能：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

  public static void main(String[] args) {
    SpringApplication.run(NacosProviderApplication.class, args);
  }

  @RestController
  class EchoController {
    @RequestMapping(value = "/echo/{string}", method = RequestMethod.GET)
    public String echo(@PathVariable String string) {
      return "Hello Nacos Discovery " + string;
    }
  }
}
```

### 3. 配置服务消费者，从而服务消费者可以通过 Nacos 的服务注册发现功能从 Nacos server 上获取到它要调用的服务。

i. 在 `application.properties` 中配置 Nacos server 的地址：

```json
server:  
  port: 8891  
  
spring:  
  application:  
    name: service-consumer  
  
  
  cloud:  
    nacos:  
      server-addr: 127.0.0.1:8848
```

ii. 通过 Spring Cloud 原生注解 `@EnableDiscoveryClient` 开启服务注册发现功能。给 [RestTemplate](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-resttemplate.html) 实例添加 `@LoadBalanced` 注解，开启 `@LoadBalanced` 与 [Ribbon](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html) 的集成：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

    @RestController
    public class TestController {

        private final RestTemplate restTemplate;

        @Autowired
        public TestController(RestTemplate restTemplate) {this.restTemplate = restTemplate;}

        @RequestMapping(value = "/echo/{str}", method = RequestMethod.GET)
        public String echo(@PathVariable String str) {
            return restTemplate.getForObject("http://service-provider/echo/" + str, String.class);
        }
    }
}
```

### 4. 启动 `ProviderApplication` 和 `ConsumerApplication`
调用 `http://localhost:8080/echo/2018`，返回内容为 `Hello Nacos Discovery 2018`。