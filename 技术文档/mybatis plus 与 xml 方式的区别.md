

这两个代码的效果**不一定相同**，具体取决于 `sys_role_menu` 表的主键定义和 MyBatis-Plus 的实体类配置。以下是详细分析：

---

### **1. 代码效果对比**

#### **(1) MyBatis XML 的 `<delete>` 语句**
```xml
<delete id="deleteByRoleId">
    delete from sys_role_menu where role_id = #{roleId}
</delete>
```
- **效果**：根据 `role_id` 删除 `sys_role_menu` 表中所有匹配的记录。
- **特点**：直接通过 `role_id` 字段删除数据，无论 `role_id` 是否是主键。

#### **(2) MyBatis-Plus 的 `deleteById` 方法**
```java
sysRoleMenuMapper.deleteById(assginRoleDto.getRoleId());
```
- **效果**：根据 `主键` 删除 `sys_role_menu` 表中匹配的记录。
- **特点**：`deleteById` 是 MyBatis-Plus 的默认方法，其行为依赖实体类的主键定义：
  - 如果 `sys_role_menu` 表的主键是 `id` 字段（如自增主键），则此代码会尝试删除 `id = roleId` 的记录（逻辑错误）。
  - 如果 `sys_role_menu` 表的主键是 `role_id` 字段，则此代码会删除 `role_id = roleId` 的记录（与 XML 的 `<delete>` 语句效果相同）。

---

### **2. 关键差异**

| 特性                | MyBatis XML `<delete>` 语句       | MyBatis-Plus `deleteById` 方法       |
|---------------------|----------------------------------|-------------------------------------|
| **删除依据**         | 根据 `role_id` 字段删除           | 根据 `主键` 字段删除                  |
| **删除范围**         | 删除所有 `role_id` 匹配的记录      | 删除单条主键匹配的记录（若主键是 `role_id`，则效果相同） |
| **适用场景**         | 通用场景                         | 依赖主键定义的场景                    |

---

### **3. 常见场景分析**

#### **(1) 场景 1：`sys_role_menu` 表的主键是 `id`**
- **表结构示例**：
  ```sql
  CREATE TABLE sys_role_menu (
      id BIGINT PRIMARY KEY AUTO_INCREMENT,  -- 主键是自增的 id
      role_id BIGINT,
      menu_id BIGINT,
      ...
  );
  ```
- **效果对比**：
  - **XML 的 `<delete>` 语句**：正确删除所有 `role_id = #{roleId}` 的记录。
  - **`deleteById(roleId)`**：尝试删除 `id = roleId` 的记录（逻辑错误，因为 `roleId` 不是主键值）。

#### **(2) 场景 2：`sys_role_menu` 表的主键是 `role_id`**
- **表结构示例**：
  ```sql
  CREATE TABLE sys_role_menu (
      role_id BIGINT PRIMARY KEY,  -- 主键是 role_id
      menu_id BIGINT,
      ...
  );
  ```
- **效果对比**：
  - **XML 的 `<delete>` 语句**：删除 `role_id = #{roleId}` 的记录。
  - **`deleteById(roleId)`**：删除 `role_id = #{roleId}` 的记录（效果相同）。

---

### **4. MyBatis-Plus 的正确用法**
如果目标是删除 `role_id = #{roleId}` 的记录，应使用 MyBatis-Plus 的条件删除方法，而非 `deleteById`：
```java
// 使用 QueryWrapper 指定删除条件
sysRoleMenuMapper.delete(
    new QueryWrapper<SysRoleMenu>()
        .eq("role_id", assginRoleDto.getRoleId())
);
```
- **效果**：与 XML 的 `<delete>` 语句完全一致。

---

### **5. 总结**
- **若 `role_id` 是主键**：`deleteById(roleId)` 与 XML 的 `<delete>` 语句效果相同。
- **若 `role_id` 不是主键**：`deleteById(roleId)` 是错误用法，需改用条件删除（`QueryWrapper`）。
- **建议**：始终通过 `QueryWrapper` 明确删除条件，避免依赖主键字段的隐含逻辑。