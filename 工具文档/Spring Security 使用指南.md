
Spring Security主要是提供了身份验证，授权管理，防护机制等功能。
核心概念包括：
- **身份验证 (Authentication)**: 验证用户的身份（例如，用户名/密码）。
- **授权 (Authorization)**: 确定用户是否有权限访问特定资源。
- **安全上下文 (Security Context)**: 存储已认证用户的详细信息，应用程序中可以访问。

# 一. 准备工作
## 1.1 引入依赖

spring-boot-starter-security

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>3.3.4</version>
</dependency>
```

jjwt-api
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
```

jjwt-impl
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

jjwt-jackson
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

一般会创建一个 `SecurityConfig` 类 ，管理所有的与 `security` 相关配置。

```java
@Configuration
@EnableWebSecurity		// 该注解启用 Spring Security 的 web 安全功能。
public class SecurityConfig {
}
```

## 1.2 用户认证的配置

基于内存的用户认证

通过 `createUser` , manager 把用户配置的账号密码添加到spring的内存中, `InMemoryUserDetailsManager`  类中有一个 `loadUserByUsername` 的方法通过账号（username）从内存中获取我们配置的账号密码，之后调用其他方法来判断前端用户输入的密码和内存中的密码是否匹配。

```java
@Bean
public UserDetailsService userDetailsService() {
	// 创建基于内存的用户信息管理器
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
 
	
    manager.createUser(
	    // 创建UserDetails对象，用于管理用户名、用户密码、用户角色、用户权限等内容
				User
				.withDefaultPasswordEncoder()
				.username("user")
				.password("user123")
				.roles("USER")
				.build()
    ); 		
    // 如果自己配置的有账号密码, 那么上面讲的 user 和 随机字符串 的默认密码就不能用了
    return manager;
}
```

**基于数据库的用户认证**

spring security 是通过 `loadUserByUsername` 方法来获取 `User` 并用这个 `User` 来判断用户输入的密码是否正确。所以我们只需要继承 `UserDetailsService` 接口并重写 `loadUserByUsername` 方法即可

下面的样例我用的 mybatis-plus 来查询数据库中的 `user`, 然后通过当前查询到的 `user` 返回特定的 `UserDetails` 对象

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
 
    @Autowired
    UserMapper userMapper;
 
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        QueryWrapper<User> queryWrapper = new QueryWrapper<User>();
        queryWrapper.eq("username", username);	// 这里不止可以用username，你可以自定义，主要根据你自己写的查询逻辑
        User user = userMapper.selectOne(queryWrapper);
        if (user == null) {
            throw new UsernameNotFoundException(username);
        }
        return new UserDetailsImpl(user);	// UserDetailsImpl 是我们实现的类
    }
}
```

`UserDetailsImpl` 是实现了 `UserDetails` 接口的类。`UserDetails` 接口是 Spring Security 身份验证机制的基础，通过实现该接口，开发者可以定义自己的用户模型，并提供用户相关的信息，以便进行身份验证和权限检查。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor	// 这三个注解可以帮我们自动生成 get、set、有参、无参构造函数
public class UserDetailsImpl implements UserDetails {
 
    private User user;	// 通过有参构造函数填充赋值的
 
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of();
    }
 
    @Override
    public String getPassword() {
        return user.getPassword();
    }
 
    @Override
    public String getUsername() {
        return user.getUsername();
    }
 
    @Override
    public boolean isAccountNonExpired() {  // 检查账户是否 没过期。
        return true;
    }
 
    @Override
    public boolean isAccountNonLocked() {   // 检查账户是否 没有被锁定。
        return true;
    }
 
    @Override
    public boolean isCredentialsNonExpired() {  //检查凭据（密码）是否 没过期。
        return true;
    }
 
    @Override
    public boolean isEnabled() {    // 检查账户是否启用。
        return true;
    }
    
    // 这个方法是 @Data注解 会自动帮我们生成，用来获取 loadUserByUsername 中最后我们返回的创建UserDetailsImpl对象时传入的User。
    // 如果你的字段包含 username和password 的话可以用强制类型转换, 把 UserDetailsImpl 转换成 User。如果不能强制类型转换的话就需要用到这个方法了
	public User getUser() {	
	        return user;	
	    }
	}
```

## 1.3 基本的配置

下面这个是 `security` 的默认配置。我们可以修改并把它加到spring容器中，完成我们特定的需求。

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            // 开启授权保护
            .authorizeHttpRequests(authorize -> authorize
                    // 不需要认证的地址有哪些
                    .requestMatchers("/blog/**", "/public/**", "/about").permitAll()	// ** 通配符
                    // 对所有请求开启授权保护
                    .anyRequest().
                    // 已认证的请求会被自动授权
                    authenticated()
            )
            // 使用默认的登陆登出页面进行授权登陆
            .formLogin(Customizer.withDefaults())
            // 启用"记住我"功能的。允许用户在关闭浏览器后，仍然保持登录状态，直到他们主动注销或超出设定的过期时间。
            .rememberMe(Customizer.withDefaults());
    // 关闭 csrf CSRF（跨站请求伪造）是一种网络攻击，攻击者通过欺骗已登录用户，诱使他们在不知情的情况下向受信任的网站发送请求。
    http.csrf(csrf -> csrf.disable());
 
    return http.build();
}
```
关于上面放行路径写法的一些细节问题:  
如果在 `application.properities` 中配置的有 `server.servlet.context-path=/api` 前缀的话，在放行路径中不需要写 `/api`。  
如果 `@RequestMapping(value = "/test/")` 中写的是 `/test/`, 那么放行路径必须也写成 `/test/`, （`/test`）是不行的，反之亦然。  
如果 `@RequestMapping(value = "/test")` 链接 `/test` 后面要加查询字符的话（`/test?type=0`），不要写成 `@RequestMapping(value = "/test/")`

上面都是一些细节问题，但是遇到 bug 的时候不容易发现。


以下为常用的配置，配置了jwt，加密的类，过滤器启用 jwt，而不是session

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
 
    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
 
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
 
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
 
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(CsrfConfigurer::disable) // 基于token，不需要csrf
                .sessionManagement((session) -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // 基于token，不需要session
                .authorizeHttpRequests((authz) -> authz
                        .requestMatchers("/login/",  "/getPicCheckCode").permitAll() // 放行api
                        .requestMatchers(HttpMethod.OPTIONS).permitAll()
                        .anyRequest().authenticated()
                )
                .addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
 
        return http.build();
    }
}
```

# 二. 加密

Spring Security 提供了多种加密和安全机制来保护用户的敏感信息，尤其是在用户身份验证和密码管理方面。（我们只讲默认的 `BCryptPasswordEncoder`）

## 1. Spring Security 的加密机制
### 1.1 PasswordEncoder 接口

Spring Security 提供了 `PasswordEncoder` 接口，用于定义密码的加密和验证方法。主要有以下几种实现

- **BCryptPasswordEncoder**：基于 BCrypt 算法，具有适应性和强大的加密强度。它可以根据需求自动调整加密的复杂性。
- **NoOpPasswordEncoder**：不执行加密，适用于开发和测试环境，不建议在生产环境中使用。
- **Pbkdf2PasswordEncoder**、**Argon2PasswordEncoder** 等：这些都是基于不同算法的实现，具有不同的安全特性。
### 1.2 BCrypt 算法

BCrypt 是一种安全的密码哈希算法，具有以下特点：
- **盐值（Salt）**：每次加密时都会生成一个随机盐值，确保相同的密码在每次加密时生成不同的哈希值。
- **适应性**：通过增加计算复杂性（如工作因子），可以提高密码的加密强度。


## 2. 使用 PasswordEncoder

### 2.1 配置 PasswordEncoder

在 Spring Security 配置类中定义 `PasswordEncoder` bean
```java
@Configuration
public class SecurityConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        // 也可用有参构造，取值范围是 4 到 31，默认值为 10。数值越大，加密计算越复杂
        return new BCryptPasswordEncoder();	
    }
}
```

### 2.2 加密密码

在注册用户或更新密码时，可以使用`PasswordEncoder` 来加密密码：
```java
@Autowired
private PasswordEncoder passwordEncoder;
 
public void registerUser(String username, String rawPassword) {
    String encryptedPassword = passwordEncoder.encode(rawPassword);
    // 保存用户信息到数据库，包括加密后的密码
}
```

### 2.3 验证密码

在用户登录时，可以使用 `matches` 方法验证输入的密码与存储的加密密码是否匹配：
```java
public boolean login(String username, String rawPassword) {
    // 从数据库中获取用户信息，包括加密后的密码
    String storedEncryptedPassword = // 从数据库获取;
    return passwordEncoder.matches(rawPassword, storedEncryptedPassword);
}
```

# 三. 前后端分离

## 1. 用户认证流程
### 1.1 用户登录
- **前端**：
    - 用户在登录界面输入用户名和密码。
    - 前端将这些凭证以 JSON 格式发送到后端的登录 API（例如 `POST /api/login`）。
- **后端**：
    - Spring Security 接收请求，使用 `AuthenticationManager` 进行身份验证。
    - 如果认证成功，后端生成一个 JWT（JSON Web Token）或其他认证令牌，并将其返回给前端。

## 2. 使用 JWT 进行用户认证

### 2.1 前端存储 JWT
- 前端收到 JWT 后，可以将其存储在 `localStorage` 或 `sessionStorage` 中，以便后续请求使用。
#### 2.2 发送受保护请求

- 在发送需要认证的请求时，前端将 JWT 添加到请求头中：

```json
fetch('/api/protected-endpoint', {
    method: 'GET',
    headers: {
        'Authorization': `Bearer ${token}`
    }
});
```

### 2.3 后端解析 JWT
- Spring Security 通过过滤器来解析和验证 JWT。可以自定义一个 `OncePerRequestFilter` 以拦截请求，提取 JWT，并验证其有效性。

## 3. 退出登录
由于 JWT 属于无状态，所以后端是不需要记录会话状态的。用户可以通过前端操作（例如，删除存储的 JWT）来"退出登录"。可以实现一个注销接口，用于前端执行相关逻辑。

### 4. 保护敏感信息

- 确保 HTTPS：在前后端通信中使用 HTTPS，确保传输中的数据安全。
- 令牌过期：设置 JWT 的有效期，过期后需要用户重新登录。
- 刷新令牌：可以实现刷新令牌的机制，以提高用户体验，避免频繁登录。
- 跨站脚本攻击（XSS）防护：使用 HTTPOnly 和 Secure cookie 标志，限制 JavaScript 对敏感数据的访问。
- 跨站请求伪造（CSRF）防护：对于需要修改服务器状态的操作，实施适当的 CSRF 防护措施。

# 四. 实现 JWT

## 1. OncePerRequestFilter

- **作用**：`OncePerRequestFilter` 是 Spring Security 提供的抽象类，确保每个请求只执行一次特定过滤逻辑。是实现自定义过滤器的基础，通常用语对请求进行预处理和后处理(`jwt` 实现依赖这个接口)
- **功能**：提供一个机制，确保过滤器逻辑每个请求只执行一次，使用对每个请求进行处理的场景。通过继承这个类可以实现自定义过滤器的记录日志、身份验证、权限检查等场景
- **实现**：继承 `OncePerRequestFilter` 类，并重写 `doFilterInternal` 方法。
```java
public class CustomFilter extends OncePerRequestFilter {
 
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        // 自定义过滤逻辑，例如记录请求日志
        System.out.println("Request URI: " + request.getRequestURI());
        
        // 继续执行过滤链
        filterChain.doFilter(request, response);
    }
}
```
- **注册过滤器**：可以在 Spring Security 配置类中注册自定义的过滤器。
```java
@EnableWebSecurity
public class SecurityConfig {
 
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.addFilterBefore(new CustomFilter(), UsernamePasswordAuthenticationFilter.class);
        // 其他配置...
        return http.build();
    }
```

## 2. 生成 JWT

基于上面的三个 JWT 相关依赖，写 `JWTUtil` 类
- **功能**：`JWTUtil` 提供了生成和解析 JWT 功能，是基于 JWT 的认证和授权的重要工具。
- **应用场景**：可以在用户登录后生成 JWT ，并在后续请求中携带 JWT 用于用户身份验证和权限校验。设置合理时间提高安全性
```java
@Component
public class JwtUtil {
    public static final long JWT_TTL = 60 * 60 * 1000L * 24 * 14;  // 有效期14天
    public static final String JWT_KEY = "SDFGjhdsfalshdfHFdsjkdsfds121232131afasdfac";
 
    public static String getUUID() {
        return UUID.randomUUID().toString().replaceAll("-", "");
    }
 
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());
        return builder.compact();
    }
 
    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if (ttlMillis == null) {
            ttlMillis = JwtUtil.JWT_TTL;
        }
 
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .id(uuid)
                .subject(subject)
                .issuer("sg")
                .issuedAt(now)
                .signWith(secretKey)
                .expiration(expDate);
    }
 
    public static SecretKey generalKey() {
        byte[] encodeKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        return new SecretKeySpec(encodeKey, 0, encodeKey.length, "HmacSHA256");
    }
 
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .verifyWith(secretKey)
                .build()
                .parseSignedClaims(jwt)
                .getPayload();
    }
}
```

### 2.1. 类注解与常量

- **`@Component`**：将这个类标记为 Spring 组件，允许 Spring 管理该类的生命周期，便于依赖注入。
- **`JWT_TTL`**：JWT 的有效期，设置为 14 天（单位为毫秒）。
- **`JWT_KEY`**：用于签名的密钥字符串，经过 Base64 编码。它是生成和验证 JWT 的关键。

### 2.2. UUID 生成
```java
public static String getUUID() {
    return UUID.randomUUID().toString().replaceAll("-", "");
}
```
- **作用**：生成一个唯一的 UUID，去掉了其中的连字符。这通常用作 JWT 的 ID（`jti`），确保每个 JWT 都是唯一的。

### 2.3. 创建 JWT

```java
public static String createJWT(String subject) {
    JwtBuilder builder = getJwtBuilder(subject, null, getUUID());
    return builder.compact();
}
```

- **参数**：`subject` 是 JWT 的主题，通常是用户的身份信息。
- **功能**：调用 `getJwtBuilder` 方法生成一个 JWT 构建器，并将其压缩为字符串返回。

### 2.4. JWT 构建器
```java
private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
    SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
    SecretKey secretKey = generalKey();
    long nowMillis = System.currentTimeMillis();
    Date now = new Date(nowMillis);
    if (ttlMillis == null) {
        ttlMillis = JwtUtil.JWT_TTL;
    }

    long expMillis = nowMillis + ttlMillis;
    Date expDate = new Date(expMillis);
    return Jwts.builder()
            .id(uuid)
            .subject(subject)
            .issuer("sg")
            .issuedAt(now)
            .signWith(secretKey)
            .expiration(expDate);
}
```
- **功能**：生成一个 JWT 的构建器，配置 JWT 的各个属性，包括：
    - **`id(uuid)`**：设置 JWT 的唯一标识。
    - **`subject(subject)`**：设置 JWT 的主题。
    - **`issuer("sg")`**：设置 JWT 的发行者，通常是你的应用或服务名称。
    - **`issuedAt(now)`**：设置 JWT 的签发时间。
    - **`signWith(signatureAlgorithm, secretKey)`**：使用指定的算法和密钥进行签名。
    - **`expiration(expDate)`**：设置 JWT 的过期时间。

### 2.5. 生成密钥

```java
public static SecretKey generalKey() {
    byte[] encodeKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
    return new SecretKeySpec(encodeKey, 0, encodeKey.length, "HmacSHA256");
}
```

- **功能**：将 `JWT_KEY` 进行 Base64 解码，生成一个 `SecretKey` 对象，用于 JWT 的签名和验证。使用 HMAC SHA-256 算法进行签名。

### 2.6. 解析 JWT

```java
public static Claims parseJWT(String jwt) throws Exception {
    SecretKey secretKey = generalKey();
    return Jwts.parser()
            .verifyWith(secretKey)
            .build()
            .parseSignedClaims(jwt)
            .getPayload();
}
```

- **参数**：`jwt` 是要解析的 JWT 字符串。
- **功能**：
    - 使用之前生成的密钥解析 JWT。
    - 如果 JWT 有效，将返回 JWT 的声明（`Claims` 对象），其中包含了 JWT 的主题、发行者、过期时间等信息。
    - 该方法可能会抛出异常，表示 JWT 无效或解析失败。

## 3. 应用 JWT

先加一个依赖 `JetBrains Java Annotations`, 下面的代码会用到其中的一个注解 `@NotNull`。

这个 `JwtAuthenticationTokenFilter` 类是一个实现了 `OncePerRequestFilter` 接口自定义的 Spring Security 过滤器。
- **功能**：`JwtAuthenticationTokenFilter` 通过 JWT 对用户进行身份验证，确保请求中携带的 JWT 是有效的，并根据 JWT 提供的用户信息设置认证状态。
- **应用场景**：通常用于保护需要身份验证的 API 接口，确保只有经过认证的用户才能访问资源。该过滤器通常放置在 Spring Security 过滤链的合适位置，以确保在处理请求之前进行身份验证。

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Autowired
    private UserMapper userMapper;
 
    @Override
    protected void doFilterInternal(HttpServletRequest request, @NotNull HttpServletResponse response, @NotNull FilterChain filterChain) throws ServletException, IOException {
        String token = request.getHeader("Authorization");
 
        if (!StringUtils.hasText(token) || !token.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
 
        token = token.substring(7);
 
        String userid;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
 
        User user = userMapper.selectById(Integer.parseInt(userid));
 
        if (user == null) {
            throw new RuntimeException("用户名未登录");
        }
 
        UserDetailsImpl loginUser = new UserDetailsImpl(user);
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser, null, null);
 
        // 如果是有效的jwt，那么设置该用户为认证后的用户
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
 
        filterChain.doFilter(request, response);
    }
}
```

#### 1. 类注解与继承

- **`@Component`**：标记该类为 Spring 组件，使其能够被 Spring 扫描并注册到应用上下文中。
- **继承 `OncePerRequestFilter`**：确保每个请求只调用一次该过滤器，适合进行请求预处理。

#### 2. 依赖注入

```java
@Autowired
private UserMapper userMapper;
```

- **`UserMapper`**：一个数据访问对象（DAO），用于与数据库交互，以便根据用户 ID 查询用户信息。通过 Spring 的依赖注入机制自动注入。

#### 3. 获取 JWT

```java
String token = request.getHeader("Authorization");

if (!StringUtils.hasText(token) || !token.startsWith("Bearer ")) {
    filterChain.doFilter(request, response);
    return;
}
```

- 从请求头中获取 `Authorization` 字段的值，检查是否包含 JWT。
- 如果没有 JWT 或格式不正确，直接调用 `filterChain.doFilter(request, response)` 继续执行下一个过滤器，返回。

#### 4. 解析 JWT

```java
token = token.substring(7); // 去掉 "Bearer " 前缀

String userid;
try {
    Claims claims = JwtUtil.parseJWT(token);
    userid = claims.getSubject();
} catch (Exception e) {
    throw new RuntimeException(e);
}
```

- 从 token 中去掉 "Bearer " 前缀，只保留 JWT 部分。
- 调用 `JwtUtil.parseJWT(token)` 解析 JWT，提取出用户 ID (`userid`)。如果解析失败，会抛出异常。

#### 5. 查询用户信息

```java
User user = userMapper.selectById(Integer.parseInt(userid));

if (user == null) {
    throw new RuntimeException("用户名未登录");
}
```

- 根据解析出的用户 ID 从数据库中查询用户信息。如果用户不存在，抛出异常。

#### 6. 设置安全上下文

```java
UserDetailsImpl loginUser = new UserDetailsImpl(user);
UsernamePasswordAuthenticationToken authenticationToken =
        new UsernamePasswordAuthenticationToken(loginUser, null, null);

// 如果是有效的jwt，那么设置该用户为认证后的用户
SecurityContextHolder.getContext().setAuthentication(authenticationToken);
```

- 创建一个自定义的 `UserDetailsImpl` 对象，将查询到的用户信息封装。
- 创建一个 `UsernamePasswordAuthenticationToken` 对象，表示用户的认证信息，并将其设置到 Spring Security 的 `SecurityContextHolder` 中，以便后续请求能够访问到用户的认证信息。

#### 7. 继续过滤链

```java
filterChain.doFilter(request, response);
```

- 调用 `filterChain.doFilter(request, response)`，继续执行后续的过滤器和最终的请求处理。

### 3.4 注册自定义的 `JwtAuthenticationTokenFilter` 过滤器

把我们的过滤器应用到 `spring security` 中

```java
http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
```

## 5、常用操作

### 5.1 配置 AuthenticationManager

将 `AuthenticationManager` 对象添加到spring容器中。（添加到我们创建的 `SecurityConfig` 配置类中）

```java
@Bean
public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
    return authConfig.getAuthenticationManager();
}
```

### 5.2 验证当前用户

```java
@Autowired
private AuthenticationManager authenticationManager;

UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username, password);

// 从数据库中对比查找，如果找到了会返回一个带有认证的封装后的用户，否则会报错，自动处理。
// （这里我们假设我们配置的security是基于数据库查找的）
Authentication authenticate = authenticationManager.authenticate(authenticationToken);

// 获取认证后的用户
User user = (User) authenticate.getPrincipal();
// 如果强制类型转换报错的话，可以用我们实现的 `UserDetailsImpl` 类中的 getUser 方法了

String jwt = JwtUtil.createJWT(user.getId().toString());
```

### 5.3 获取当前用户的认证信息

以下是获取当前用户认证信息的具体步骤：

```java
// SecurityContextHolder 是一个存储安全上下文的工具类，提供了一个全局访问点，用于获取当前请求的安全上下文。
// SecurityContext 是当前线程的安全上下文，包含了当前用户的认证信息（即 Authentication 对象）。
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
User user = (User) authenticate.getPrincipal();

// 另一种获取 User 的方法
UsernamePasswordAuthenticationToken authenticationToken = 
    (UsernamePasswordAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();
UserDetailsImpl loginUser = (UserDetailsImpl) authenticationToken.getPrincipal();
User user = userDetails.getUser();
```

### 5.4 对比 `authenticationManager` 和 `SecurityContext` 获取 `Authentication`

1. **`SecurityContextHolder.getContext().getAuthentication()`**:
   - 这个方法获取当前线程的安全上下文中的 `Authentication` 对象。
   - 通常在用户已经被认证后使用，用于获取当前用户的信息（如身份、权限等）。

2. **`authenticationManager.authenticate(authenticationToken)`**:
   - 这个方法是用于实际执行身份验证的过程。
   - 它接收一个凭证（如用户名和密码）并返回一个经过验证的 `Authentication` 对象，或者抛出异常。
   - 在用户登录或验证时使用，属于身份验证的实际逻辑。

### 5.5 Authentication 接口

`Authentication` 接口包含以下重要方法：

- **`getPrincipal()`**：返回当前用户的身份，通常是用户的详细信息（如用户名）。
- **`getCredentials()`**：返回用户的凭证（如密码），但通常不直接使用此方法。
- **`getAuthorities()`**：返回用户的权限列表（角色或权限）。

## 6、授权

Spring Security 的授权机制用于控制用户对应用程序资源的访问权限。授权通常是基于用户角色或权限的，以下是对 Spring Security 授权的详细讲解。

### 1. 授权的基本概念

- **授权（Authorization）**：指的是决定用户是否有权限访问特定资源的过程。在用户认证成功后，系统会根据预定义的规则来判断用户是否可以访问某些资源。

### 2. 授权的主要组成部分

#### 2.1 权限与角色

- **权限（Permission）**：通常指对特定资源的操作能力，如"读取"、"写入"或"删除"。
- **角色（Role）**：一组权限的集合。例如，管理员角色可能具有所有权限，而普通用户角色可能只有读取权限。

#### 2.2 GrantedAuthority 接口

- Spring Security 使用 `GrantedAuthority` 接口表示用户的权限。每个用户的权限可以通过实现该接口的对象来表示。

### 3. 授权方式

Spring Security 支持多种授权方式，主要包括：

#### 3.1 基于角色的授权

- **定义角色**：通常在用户注册或用户管理中定义。
- **配置角色授权**：可以在 Spring Security 配置类中设置基于角色的访问控制。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .antMatchers("/admin/**").hasRole("ADMIN")  // 只有 ADMIN 角色可以访问
        .antMatchers("/user/**").hasAnyRole("USER", "ADMIN")  // USER 和 ADMIN 角色可以访问
        .anyRequest().authenticated();  // 其他请求需要认证
}
```

#### 3.2 基于权限的授权

- **定义权限**：与角色类似，定义更细粒度的权限。
- **配置权限授权**：在配置类中设置基于权限的访问控制。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .antMatchers("/edit/**").hasAuthority("EDIT_PRIVILEGE")  // 仅具有 EDIT_PRIVILEGE 权限的用户可以访问
        .anyRequest().authenticated();
}
```

### 4. 自定义授权逻辑

如果需要更复杂的授权逻辑，可以实现自定义的 `AccessDecisionVoter` 或 `AccessDecisionManager`。

#### 4.1 AccessDecisionVoter

- 负责评估用户的访问请求，返回授权决策。

#### 4.2 AccessDecisionManager

- 管理多个 `AccessDecisionVoter` 的调用，确定用户是否有权访问请求的资源。

## 7、其他自定义行为

以下接口和类用于处理不同的安全事件，提供了自定义处理的能力：

### 1. AuthenticationSuccessHandler

- **作用**：用于处理用户成功认证后的逻辑。
- **功能**：可以自定义成功登录后的跳转行为，比如重定向到特定页面、返回 JSON 响应等。
- **实现**：实现 `AuthenticationSuccessHandler` 接口，并重写 `onAuthenticationSuccess` 方法。

```java
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, 
                                       Authentication authentication) throws IOException, ServletException {
        // 自定义成功认证后的处理逻辑
    }
}

// 让我们实现的类生效
http.formLogin(form ->
                form.successHandler(new MyAuthenticationSuccessHandler()));
```

### 2. AuthenticationFailureHandler

- **作用**：用于处理用户认证失败的逻辑。
- **功能**：可以自定义失败登录后的行为，比如返回错误信息、重定向到登录页面并显示错误提示等。
- **实现**：实现 `AuthenticationFailureHandler` 接口，并重写 `onAuthenticationFailure` 方法。

```java
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, 
                                       AuthenticationException exception) throws IOException, ServletException {
        // 自定义认证失败后的处理逻辑
    }
}

// 让我们实现的类生效
http.formLogin(form ->
        form.failureHandler(new MyAuthenticationFailureHandler()));
```

### 3. LogoutSuccessHandler

- **作用**：用于处理用户成功登出的逻辑。
- **功能**：可以自定义注销成功后的行为，比如重定向到登录页面、显示注销成功的消息等。
- **实现**：实现 `LogoutSuccessHandler` 接口，并重写 `onLogoutSuccess` 方法。

```java
public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, 
                               Authentication authentication) throws IOException, ServletException {
        // 自定义注销成功后的处理逻辑
    }
}

// 让我们实现的类生效
http.logout(logout -> {
               logout.logoutSuccessHandler(new MyLogoutSuccessHandler());
           });
```

### 4. AuthenticationEntryPoint

- **作用**：用于处理未认证用户访问受保护资源时的逻辑。
- **功能**：可以自定义未认证用户的响应，比如返回 401 状态码、重定向到登录页面等。
- **实现**：实现 `AuthenticationEntryPoint` 接口，并重写 `commence` 方法。

```java
// 重写 AuthenticationEntryPoint 接口
public class MyAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, 
                        AuthenticationException authException) throws IOException, ServletException {
        // 自定义未认证用户访问受保护资源的处理逻辑
    }
}

// 让我们重写的类生效
http.exceptionHandling(exception -> {
    exception.authenticationEntryPoint(new MyAuthenticationEntryPoint());
});
```

### 5. SessionInformationExpiredStrategy

- **作用**：用于处理用户会话过期的逻辑。
- **功能**：可以自定义会话超时后的响应，比如重定向到登录页面或返回 JSON 响应等。
- **实现**：实现 `SessionInformationExpiredStrategy` 接口，并重写 `onExpiredSession` 方法。

```java
// 重写 SessionInformationExpiredStrategy 接口
public class MySessionInformationExpiredStrategy implements SessionInformationExpiredStrategy {
     @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException, ServletException {
        // 自定义会话过期的处理逻辑
    }
}

// 让我们重写的类生效
http.sessionManagement(session -> {
    session.maximumSessions(1).expiredSessionStrategy(new MySessionInformationExpiredStrategy());
});
```

### 6. AccessDeniedHandler

- **作用**：用于处理用户访问被拒绝的情况。当用户尝试访问没有权限的资源时，Spring Security 会调用实现了 `AccessDeniedHandler` 接口的处理器。
- **功能**：可以自定义拒绝访问后的响应行为，比如重定向到特定的错误页面、返回错误信息或 JSON 响应。
- **实现**：实现 `AccessDeniedHandler` 接口，并重写 `handle` 方法。

```java
// 重写 AccessDeniedHandler 接口
public class MyAccessDeniedHandler implements AccessDeniedHandler {
     @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                      AccessDeniedException accessDeniedException) throws IOException, ServletException {
        // 自定义访问被拒绝的处理逻辑
    }
}

// 在安全配置中注册自定义的 AccessDeniedHandler
http.exceptionHandling(exception -> {
    exception.accessDeniedHandler(new MyAccessDeniedHandler());
});
```

通过这些自定义的处理器，可以精确控制 Spring Security 在不同安全事件发生时的行为，使其更符合具体应用的需求。