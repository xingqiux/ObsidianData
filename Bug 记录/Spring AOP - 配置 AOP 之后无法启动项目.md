
> AOP 设置的参数错误导致无法创建注解绑定

## 核心原理

在 AspectJ 中，`@Around` 通知的参数绑定遵循严格的规则，特别是当涉及到注解绑定和 `ProceedingJoinPoint` 的使用时。

### 1. ProceedingJoinPoint 必须是第一个参数

在使用 `@Around` 通知时，`ProceedingJoinPoint` **必须作为第一个参数**。这是因为：

- `@Around` 通知是唯一能够控制是否继续执行目标方法的通知类型（通过 `proceed()` 方法）
- AspectJ 需要明确识别用于控制执行流程的参数
- 当绑定其他参数时，AspectJ 需要一个稳定的约定来解析每个参数的含义

### 2. 注解参数绑定的工作原理

当使用 `@annotation(log)` 切点表达式时，发生了以下过程：

1. AspectJ 在运行时解析切点表达式
2. 它尝试将表达式中的形参 `log` 与方法签名中的某个参数匹配
3. 当找到类型匹配的参数（在这里是 `Log log`）时，它建立绑定
4. 在执行通知时，AspectJ 将目标方法上的 `@Log` 注解实例注入到这个参数中

### 3. 参数顺序错误导致的问题

在原始代码中：
```java
@Around(value = "@annotation(log)")
public Object doAroundAdvice(Log log, ProceedingJoinPoint join) { ... }
```

AspectJ 在尝试解析时遇到问题，因为：

- 它期望 `ProceedingJoinPoint` 是第一个参数
- 当第一个参数是 `Log log` 时，AspectJ 无法正确建立绑定关系
- 这导致 "formal unbound in pointcut" 错误，表示无法将切点表达式中的形参与方法参数绑定

### 4. 正确的绑定方式

修复后的代码：
```java
@Around(value = "@annotation(log)")
public Object doAroundAdvice(ProceedingJoinPoint join, Log log) { ... }
```

这样工作是因为：
- `ProceedingJoinPoint` 在第一位，符合 AspectJ 的要求
- `Log log` 参数与切点表达式中的 `log` 形参名称匹配
- 参数类型与预期的注解类型一致

## Spring AOP 的特殊处理

Spring AOP 是 AspectJ 的一种轻量级实现，它主要使用动态代理而非编译时织入。在处理 `@Around` 通知时：

1. Spring 创建通知的方法拦截器
2. 拦截器在调用目标方法时，构造 `ProceedingJoinPoint` 对象
3. 如果切点表达式包含形参绑定，Spring 会尝试解析这些绑定
4. 如果无法解析绑定关系，Spring 在启动时就会抛出异常

这就是为什么正确的参数顺序至关重要的原因 - 它直接影响 Spring 能否正确解析和应用切面。

## 总结

参数顺序问题解决了，因为：

1. AspectJ 规范要求 `ProceedingJoinPoint` 作为 `@Around` 通知的第一个参数
2. 参数名称 `log` 与切点表达式中的形参名称 `@annotation(log)` 保持一致
3. 参数类型 `Log` 与目标方法上的注解类型相符

这种严格的参数顺序和命名约定是 AspectJ 确保切面正确应用的核心机制。