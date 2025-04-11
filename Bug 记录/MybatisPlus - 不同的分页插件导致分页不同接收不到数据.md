## 今日遇到一个数据无法解析问题

后端发送给前端数据成功，但无法解析，
![[Pasted image 20250311102636.png]]


代码是这个问题，其中的 

```javascript

// 定义分页查询方法
const fetchData = async () => {
    if (createTimes.value.length == 2) {
        queryDto.value.createTimeBegin = createTimes.value[0]
        queryDto.value.createTimeEnd = createTimes.value[1]
    }
    // 请求后端接口进行分页查询
    const { code , message , data } = await GetSysUserListByPage(pageParams.value.page , pageParams.value.limit , queryDto.value)
    console.log(data)
    list.value = data.list
    total.value = data.total
}
```
问题就出在这个 data.list ，但之前用角色管理的接口就是list
![[Pasted image 20250311102853.png]]

然后就习惯的用list 处理，主要问题就在于
后端的处理时，一个使用的 MybatiPlus 的分页查询，但另一个用的是  **import com.github.pagehelper.PageInfo;** 
这个分页插件，就导致两个分页插件的数据不同，从而解析失败

解决方案，只使用一个分页插件进行查询
![[Pasted image 20250311103917.png]]
```java
@Override  
public Page<SysRole> findByPage(SysRoleDto sysRoleDto, Integer pageNum, Integer pageSize) {  
    // 设置分页的页码和每页显示的条数  
    Page<SysRole> page = new Page<>(pageNum , pageSize) ;  
  
    // 构建查询条件  
    LambdaQueryWrapper<SysRole> queryWrapper = new LambdaQueryWrapper<>();  
    if (sysRoleDto.getRoleName() != null && !sysRoleDto.getRoleName().isEmpty()) {  
        queryWrapper.like(SysRole::getRoleName, sysRoleDto.getRoleName());  
    }  
  
    return sysRoleMapper.selectPage(page, queryWrapper);  
}
```
完成