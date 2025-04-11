在后端（Java Spring Boot）中，将 `Long` 类型的字段以字符串形式返回给前端有以下几种实现方式：

---

### **1. 使用 `@JsonFormat` 注解（推荐）**

直接在实体类字段上添加 `@JsonFormat(shape = JsonFormat.Shape.STRING)`，强制将该字段序列化为字符串：


```java
public class ProductSku extends BaseEntity {    // 其他字段...    
@JsonFormat(shape = JsonFormat.Shape.STRING)  // 关键注解    
private Long productId;    // 其他字段...}
```

**优点**：

- 简单直接，无需修改业务逻辑。
- 精准控制特定字段的序列化行为。