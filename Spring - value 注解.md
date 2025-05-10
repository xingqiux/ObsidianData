
# 作用
该注解的作用是将我们配置文件的属性读出来，有 **@Value(“${}”)** 和 **@Value(“#{}”)**

# ${}和#{}区别.

@Value的值有两类：
① ${ property : default_value }
② #{ obj.property? :default_value }
第一个注入的是外部配置文件对应的property，
第二个则是SpEL表达式对应的内容。 那个default_value，就是前面的值为空时的默认值。

注意二者的不同，#{}里面那个obj代表对象。