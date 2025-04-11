管理软件，又称为项目管理软件，有java语言本身没有自带项目管理工具，所以需要引入第三方的管理软件 maven或者gradle，这里使用maven进行讲解
### 安装
1. 先安装java jdk 并配置（在此不过多解释）
2. 在https://maven.apache.org/download.cgi 下载Maven
3. Windows中使用系统管理器配置环境变量
```
配置M2_HOME M2 MAVEN_OPTS 
M2_HOME=C:\Program Files\Apache Software Foundation\apache-maven-3.2.5
    
M2=%M2_HOME%\bin
    
MAVEN_OPTS=-Xms256m -Xmx512m
```
     (linux 和 mac)
```
export M2_HOME=/usr/local/apache-maven/apache-maven-3.2.5
    
export M2=$M2_HOME/bin
    
export MAVEN_OPTS=-Xms256m -Xmx512m
```
4. Windows 添加字符串 “;%M2%” 到系统“Path”变量末尾 ,
   Linux export PATH=$M2:$PATH
   Mac export PATH=$M2:$PATH
5. 测试配置 mvn --version
如果成功显示 
![[p1.png|675]]
说明安装成功
### 新建一个maven管理的项目
打开idea 新建
![[p2.png]]
新建一个maven项目
![[p3.png|700]]
![[p4.png]]
至此 我们已经完成了成功的一大部分啦，现在只需要了解maven导入依赖就行啦
> 在此之前我再次声明，本文章只是简单的用于介绍如何基础的使用maven管理和将项目跑起来，配置maven/conf/settings.xml等信息没有写清楚，以及一些maven的基础概念，需要了解的话自行查阅相关资料哦

### pom.xml
![[p5.png]]
这个就是用来管理maven环境的地方
maven由很多的特性，以后可以详解的时候再写
接下来需要使用这个标签
```xml
<dependencies>  
    <dependency> 
        <groupId>org.springframework</groupId>  
        <artifactId>spring-core</artifactId>  
        <version>6.1.13</version>  
    </dependency></dependencies>
```
这一段的内容直接写在project标签中，其中的
groupId  组id
artifactId  名称
version  版本号
用来组成一个pom的核心
![[p7.png]]
导入成功后有一个按钮再右上角，触发idea就会将依赖下载![[p8.png]]

### hello world 项目
这样一个基本的maven操作流程就完成了，接下来可以尝试一下比如junit
![[p9.png]]
可以安装maven serch插件，再这里使用maven search
![[p10.png]]
![[p11.png]]
接下来使用@Test
![[p12.png]]
这样就实现了一个maven工程的搭建,更多关于maven的详细使用，可以参照其他资源