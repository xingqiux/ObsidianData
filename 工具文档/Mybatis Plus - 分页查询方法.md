## 基本流程

#### 1. 创建分页对象

使用的是  Page 这个分页对象类

#### 2.创建查询条件

一般使用 LambdaQueryWrapper 创建分页查询的条件，在这个链接中可以查询所有的查询拼接方法

[https://baomidou.com/guides/wrapper/](MybatisPlus 拼接方法链接)

#### 3.执行分页查询
以下是完整的实现：

```java
@PostMapping("/findByPage/{current}/{pageSize}")
public Result findByPage(@PathVariable("current") Integer current,
                         @PathVariable("pageSize") Integer pageSize,
                         @RequestBody SysRoleDto sysRoleDto) {

    // 创建分页对象
    Page<SysRole> page = new Page<>(current, pageSize);
    
    // 创建查询条件
    LambdaQueryWrapper<SysRole> queryWrapper = new LambdaQueryWrapper<>();
    
    // 根据DTO中的条件构建查询条件
    if (StringUtils.hasText(sysRoleDto.getRoleName())) {
        queryWrapper.like(SysRole::getRoleName, sysRoleDto.getRoleName());
    }
    
    if (StringUtils.hasText(sysRoleDto.getRoleCode())) {
        queryWrapper.like(SysRole::getRoleCode, sysRoleDto.getRoleCode());
    }
    
    // 可以添加更多查询条件...
    
    // 执行分页查询
    sysRoleService.page(page, queryWrapper);
    
    // 返回结果
    return Result.ok(page);
}
```

## 说明

1. 创建了 `Page` 对象，设置当前页码和每页大小
2. 构建 `LambdaQueryWrapper` 用于设置查询条件
3. 根据传入的 `sysRoleDto` 中的属性值设置相应的查询条件（模糊查询、精确匹配等）
4. 调用 `sysRoleService.page()` 执行分页查询
5. 返回查询结果

如果您需要在 DTO 和实体类之间进行转换，可以修改代码以适应您的需求。例如，如果需要将返回的实体列表转换为 DTO，您可以添加转换逻辑或使用 MapStruct 等工具。