
# Spring MVC 知识
## Servlet 生命周期有哪些

Servlet 生命周期包含以下四个主要阶段：

1. **加载和实例化（Loading and Instantiation）**
   - Servlet 容器加载 Servlet 类
   - 使用无参构造函数创建一个 Servlet 实例
   - 根据 `<load-on-startup>` 设置决定是否在启动时加载

2. **初始化（Initialization）**
   - 容器调用 `init(ServletConfig config)` 方法
   - 整个生命周期中只执行一次
   - 用于进行 Servlet 的初始化工作（如建立数据库连接）
   - 必须成功完成初始化，Servlet 才能处理请求

3. **请求处理（Request Handling）**
   - 客户端请求到达时，容器创建 ServletRequest 和 ServletResponse 对象
   - 调用 `service(ServletRequest req, ServletResponse resp)` 方法
   - 对于 HttpServlet，会根据 HTTP 方法调用相应的 doXxx() 方法（如 doGet()、doPost()）
   - 可以处理多个并发请求（多线程）

4. **销毁（Destruction）**
   - 容器决定移除 Servlet 时（通常是关闭应用或容器）
   - 调用 `destroy()` 方法，且只执行一次
   - 用于释放资源（如关闭数据库连接）
   - 调用后，容器可能回收 Servlet 实例

Servlet 容器负责管理整个生命周期，开发者主要通过实现或重写特定方法来插入自定义逻辑。


# Spring框架基础概念及面试题详解

## 基础面试题

### IoC (控制反转)

1. **什么是IoC？它解决了什么问题？**
2. **BeanFactory和ApplicationContext有什么区别？**
3. **Spring中bean的作用域有哪些？**
4. **Spring中bean的生命周期是怎样的？**
5. **什么是依赖注入？有哪几种方式？**

### AOP (面向切面编程)

1. **什么是AOP？它有什么优势？**
2. **AOP的核心概念有哪些？（切面、通知、切点等）**
3. **Spring AOP和AspectJ的区别是什么？**
4. **如何在Spring中实现AOP？**
5. **说出AOP中通知的类型及其执行顺序**

### Spring MVC

1. **SpringMVC的核心组件有哪些？**
2. **描述一下DispatcherServlet的工作流程**
3. **@Controller和@RestController有什么区别？**
4. **如何处理Spring MVC中的异常？**
5. **SpringMVC如何实现文件上传？**

### Spring Boot

1. **Spring Boot有哪些优点？**
2. **什么是Spring Boot的自动配置？**
3. **如何在Spring Boot中自定义配置？**
4. **Spring Boot中的starter是什么？有什么作用？**
5. **如何改变Spring Boot的默认端口？**

## 详细解答

### 什么是IoC？

IoC (Inversion of Control, 控制反转) 是Spring框架的核心原则之一，它将传统上由程序代码直接操控的对象的创建、依赖关系的管理等控制权，交由Spring容器来管理。

具体来说：

- 在IoC模式下，对象的创建和依赖注入由Spring容器负责
- 开发者只需声明需要什么，而不必关心如何获取

IoC的实现主要通过DI (Dependency Injection, 依赖注入)，它允许Spring在运行时将依赖的组件注入到使用它们的类中。

### 什么是AOP？

AOP (Aspect-Oriented Programming, 面向切面编程) 是一种编程范式，允许将横切关注点(如日志记录、事务管理、安全控制等)从主业务逻辑中分离出来。

核心概念包括：

- **切面(Aspect)**: 跨多个类的关注点模块化(如事务管理)
- **连接点(Join Point)**: 程序执行过程中的某个特定点(如方法执行)
- **通知(Advice)**: 在特定连接点执行的动作(如前置通知、后置通知)
- **切点(Pointcut)**: 匹配连接点的表达式
- **引入(Introduction)**: 向现有类添加新方法或属性
- **织入(Weaving)**: 将切面应用到目标对象并创建新的代理对象的过程

Spring AOP主要通过代理模式实现，默认使用JDK动态代理(基于接口)或CGLIB代理(基于类)。

### SpringMVC工作原理

SpringMVC采用了前端控制器设计模式，其工作流程如下：
![[面试八股 - Spring.png]]
1. 用户发送请求至前端控制器 DispatcherServlet
2. DispatcherServlet 收到请求后，调用 HandlerMapping 处理器映射器
3. 处理器映射器根据请求 UR 找到对应的 Handler(Controller)，并返回 HandlerExecutionChain(包含Handler和拦截器)
4. DispatcherServlet 调用 HandlerAdapter 处理器适配器
5. HandlerAdapter 执行 Handler(Controller)
6. Controller 执行完成后，返回 ModelAndView
7. HandlerAdapter 将ModelAndView 返回给DispatcherServlet
8. DispatcherServlet 将 ModelAndView 传给 ViewResolver 视图解析器
9. ViewResolver 解析后返回具体 View
10. DispatcherServlet 根据 View 进行渲染视图
11. DispatcherServlet 响应用户

### SpringBoot启动流程

SpringBoot的启动流程主要包括：

1. **创建SpringApplication对象**：
    
    - 设置资源加载器和主配置类
    - 判断应用类型(Web或非Web)
    - 设置初始化器(Initializers)和监听器(Listeners)
2. **执行run方法**：
    
    - 创建并启动计时器
    - 创建应用上下文(ApplicationContext)
    - 准备上下文环境(Environment)
    - 打印Banner信息
    - 创建SpringApplication实例
    - 执行初始化器
    - 加载配置信息
    - 注册命令行属性
    - 刷新上下文，加载所有单例Bean
    - 执行上下文的后置处理
    - 发布应用启动完成事件
    - 执行 runners(ApplicationRunner和CommandLineRunner)

### Spring配置文件有哪几种方式

Spring支持以下几种配置方式：

1. **XML配置**：传统方式，通过XML文件定义Bean和依赖关系
2. **注解配置**：使用@Component、@Service等注解标记类，使用@Autowired等注解注入依赖
3. **Java配置**：使用@Configuration注解创建配置类，使用@Bean定义Bean
4. **YAML配置**：在Spring Boot中常用，配置应用属性

### XML格式和YAML格式的区别，谁更好

**XML格式**：

- 结构清晰，有开闭标签
- 支持丰富的校验机制(DTD、XSD)
- 冗长，不够简洁
- 在Spring传统应用中广泛使用

**YAML格式**：

- 层次结构更加清晰，使用缩进表示层级关系
- 更加简洁，不需要结束标签
- 支持复杂数据结构(列表、映射等)的简洁表示
- 在Spring Boot中广泛使用
- 对数据类型有更好的支持

**谁更好？**  
这取决于具体场景：

- YAML对于配置文件更简洁易读，特别适合Spring Boot应用
- XML有更成熟的工具支持和验证机制，适合复杂配置
- 对于新项目，特别是Spring Boot项目，YAML通常是更好的选择
- 对于遗留项目或需要严格验证的场景，XML可能更合适

### JDBC连接数据库的过程和原理

JDBC (Java Database Connectivity) 是Java访问数据库的标准API，连接数据库的过程如下：

1. **注册驱动**：
    
    JAVA
    
    `Class.forName("com.mysql.jdbc.Driver");`
    
2. **建立连接**：
    
    JAVA
    
    `Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db", "username", "password");`
    
3. **创建Statement/PreparedStatement**：
    
    JAVA
    
    `PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?"); ps.setInt(1, userId);`
    
4. **执行SQL并处理结果**：
    
    JAVA
    
    `ResultSet rs = ps.executeQuery(); while(rs.next()) {     // 处理结果 }`
    
5. **关闭资源**：
    
    JAVA
    
    `rs.close(); ps.close(); conn.close();`
    

**原理**：

- JDBC是一套规范，定义了Java程序访问数据库的标准接口
- 各数据库厂商提供实现这些接口的驱动程序
- JDBC API调用驱动程序，驱动程序与数据库通信
- DriverManager负责管理驱动程序，选择合适的驱动建立连接
- 数据库连接是一种重量级资源，通常使用连接池技术(如HikariCP、DBCP、C3P0)管理连接
- Spring提供JdbcTemplate简化JDBC操作，处理资源管理和异常转换

在Spring中，通过配置DataSource和使用JdbcTemplate，大大简化了JDBC操作的复杂性。