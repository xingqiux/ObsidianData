
# 概述

## 主要功能

### 服务注册与发现
![[Spring Cloud Alibaba Nacos 注册配置中心 1.png|525]]
注册中心的主要作用是作为协调者，主要作用是将分开的微服务项目通过服务注册和发现模型，实现项目的**跨进程跨主机进行通信**，实现**相互通信**和**相互调用**

微服务启动过程中，将自身的服务名称，IP 地址，端口号发送到注册中心。

微服务相互调用的时候，通过实例的 IP 地址和端口号进行调用

### 配置管理

主要的功能是动态的更新各个微服务的配置信息

## Nacos 配置中心

### 基础使用

![[Spring Cloud - Nacos 注册配置中心-3.png|650]]

#### 导入依赖

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

^c35ca4

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

#### 配置 Nacos

![[Spring Cloud Alibaba Nacos 注册配置中心-1 1.png|650]]

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

#### 导入配置信息

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

#### 运行测试代码

```java
@SpringBootApplication  
public class Main {  
    public static void main(String[] args) {  
        SpringApplication.run(Main.class, args);  
    }  
}
```

按照以上内容运行即可得到一个最基础的 demo 代码用语 nacos 的配置中心体现

### 配置优先级

Spring 遵循 先导入优先和外部优先规则

```json
spring.config.import=nacos:service-testConfig.properties,nacos:common.properties
```

### 数据隔离

当一个项目有多个环境例如 dev , test , prod 每一个环境需要的配置都不同，则需要不同的环境变量，这时需要进行数据隔离

![[Spring Cloud - Nacos 注册配置中心-4.png|650]]

在上图中，说明了，Namespace 用来区分开发环境， Group 进行多种微服务的分组， Data-id 作为实际数据集实现多种配置，来实现


## 接入 Nacos 注册中心

### 服务注册与发现

因为这里已经导入了依赖，所以只需要完成以下任务

1. 创建模块项目
2. 配置端口，服务名称，nacos 连接信息 [[#^c35ca4|配置]]
3. 启动实例，如果有多个可以启动多个实例进行服务注册

### 获取实例的方式

使用 Spring 自动注入
```java
@Autowired  
private NacosServiceDiscovery nacosServiceDiscovery;
```
这里面的基础方法有

```java
List<String> services = nacosServiceDiscovery.getServices();
// services = [service-provider, service-consumer]

List<ServiceInstance> instances = nacosServiceDiscovery.getInstances("service-provider"); //service-provider为需要的服务地址
ServiceInstance instance = instances.get(0);  
String url = "http://"+instance.getHost() + ":" + instance.getPort() + "/order";
// url = http://192.168.137.1:8890/order
```

通过这些方法就可以获取到实例的具体地址从而进行远程调用

### 基础远程调用

1. 在启动类设置 bean 
   ```java
	   @Bean  
		public RestTemplate restTemplate() {  
		    return new RestTemplate();  
		}
	```
2. 进行自动注入然后调用

```java
@Autowired  
private  RestTemplate restTemplate;

Order proOrder = restTemplate.getForObject( url, Order.class);
```


### 负载均衡

主要是使用 LoadBalancerClient
1. 导入依赖
2. 自动注入
3. 使用 chose() 方法根据他的负载平衡算法选择服务的实例

#### 导入依赖
```html
<dependency>  
	<groupId>org.springframework.cloud</groupId>  
	<artifactId>spring-cloud-starter-loadbalancer</artifactId>  
</dependency>
```

#### 自动注入与使用

```java
@Autowired  
	private LoadBalancerClient loadBalancerClient;


ServiceInstance instance = loadBalancerClient.choose("service-provider");  
  
String url = "http://"+instance.getHost() + ":" + instance.getPort() + "/order";  
log.info("url = " + url );
```

![[Spring Cloud - Nacos 注册配置中心.png|550]]
使用这个代码测试负载均衡得到结果
![[Spring Cloud - Nacos 注册配置中心-1.png]]
说明负载均衡正常运行

### 注解负载均衡

直接在配置 RestTemplate 的时候添加一个 @LoadBalanced 注解，即可让 restTemplate 获得动态负载能力

#### 进行负载均衡设置
```java
@LoadBalanced  
@Bean  
public RestTemplate restTemplate() {  
    return new RestTemplate();  
}
```

#### 进行负载均衡调用

将 url 配置为服务的名称(这里为 service-provider )

```java
@GetMapping("balanceAnnotation")  
public Order userOrderBalanceAnnotation() throws NacosException {  
    return restTemplate.getForObject( "http://service-provider/order", Order.class);  
}
```
(结果也是负载均衡的调用，但是我懒得一个个看了)


# 原理问题

## 如果注册中心宕机，是否还能远程调用

在实际的设计中，如果每次都向注册中心请求微服务的地址列表，在给对方服务发送地址请求，那这个过程非常耗费性能，所以可以将实例的信息进行缓存，也就是 **实例缓存** ，工作流程就变成了，当需要发起请求的时候，会从实例缓存中获取，然后发送请求，而实例缓存与注册中心进行同步和实时更新

那如果注册中心宕机了，就可以分为两种情况
1. 远程调用过服务：可以不依赖注册中心进行访问，可以通过
2. 没有调用过：无法进行远程调用

![[Spring Cloud - Nacos 注册配置中心-2.png]]