## 1.6 事务失效

当我们自定义了切面类以后，如果不注意异常的处理，那么此时就会出现事务失效的情况。

### 1.6.1 事务失效演示

以给角色分配菜单的代码为例，演示事务失效的问题，代码如下所示：

```java
// com.atguigu.spzx.manager.service.impl.SysRoleMenuServiceImpl
@Log(title = "角色菜单模块" , businessType = 2 )		
@Transactional
@Override
public void doAssign(AssginMenuDto assginMenuDto) {

    // 根据角色的id删除其所对应的菜单数据
    sysRoleMenuMapper.deleteByRoleId(assginMenuDto.getRoleId());

    int a = 1 / 0 ;		// 手动抛出异常

    // 获取菜单的id
    List<Map<String, Number>> menuInfo = assginMenuDto.getMenuIdList();
    if(menuInfo != null && menuInfo.size() > 0) {
        sysRoleMenuMapper.doAssign(assginMenuDto) ;
    }

}
```

**注意**：不加@Log注解事务可以进行回滚，但是加上该注解以后事务就会失效。



### 1.6.2 问题分析

Spring的事务控制是通过aop进行实现的，在框架底层会存在一个事务切面类，当业务方法产生异常以后，事务切面类感知到异常以后事务进行回滚。

当系统中存在多个切面类的时候，Spring框架会按照 **@Order** 注解的值对切面进行排序，@Order的值越小优先级越高，@Order的值越大优先级越低。优先级越高的切面类越优先执行，当我们没有给切面类指定排序值的时候，我们自定义的切面类的优先级和aop切面类的优先级相同，那么此时**事务切面类的优先级要高于自定义切面类**，那么切面类的执行顺序如下所示：

![[Spring 事务失效.png]] 

当在自定义切面类中对异常进行了捕获，没有将异常进行抛出，那么此时事务切面类是感知不到异常的存在，因此事务失效。



### 1.6.3 问题解决

解决方案一：使用@Order注解提高自定义切面类的优先级

解决方案二：在自定义切面类的catch中进行异常的抛出

![[Spring 事务失效-1.png]]







