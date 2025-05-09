

在我们配置spring cloud config的客户端映射时，启动项目之后出现No spring.config.import property has been defined的问题


产生问题的原因是bootstrap.properties比application.properties的优先级要高
由于bootstrap.properties是系统级的资源配置文件，是用在程序引导执行时更加早期配置信息读取；
而application.properties是用户级的资源配置文件，是用来后续的一些配置所需要的公共参数。
但是在SpringCloud 2020.* 版本把bootstrap禁用了，导致在读取文件的时候读取不到而报错，所以我们只要把bootstrap从新导入进来就会生效了。

```java
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.0.3</version>
        </dependency>
```
