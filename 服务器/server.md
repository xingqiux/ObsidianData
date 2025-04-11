# server

## Docker 

### docker run：运行一个新的容器

- -d：以后台模式运行容器

- -p：端口映射，将容器内部的端口映射到宿主机的端口

- --name：给容器命名

- -v：卷映射，将宿主机的目录或文件映射到容器内的目录或文件

### docker logs：查看容器的日志

- -f：实时查看新的日志条目

### docker exec：在运行的容器中执行命令

### Docker 镜像：用于创建容器

### Docker 容器：运行的实例

### 卷映射：将宿主机的目录或文件映射到容器内的目录或文件

### 端口映射：将容器内部的端口映射到宿主机的端口

## Linux 相关：

### /etc/resolv.conf：DNS 配置文件

### /var/log：系统日志目录

### 命令

- systemctl：管理系统服务

- ping：检查网络连通性

- echo：输出文本

- cat：查看文件内容

- 用户操作

	- /etc/passwd 存储用户信息

	- userdel -r  [用户] ： 用于删除用户，-r 删除用户对应的主目录

	- adduser [用户名] ：用于创建用户

	- 密钥

		- ssh-keygen -t rsa -b 4096 -c "you@mail.com" 
生成ssh密钥对
-t rsa 指定使用 RSA 算法。
-b 4096 指定密钥长度为 4096 位。
-C "youla@example.com" 可选，用于注释密钥，便于识别。

## 窗口screen

### screen -ls 显示所有的screan窗口

### screen -r 633245.pts-1.shuwu 进入指定窗口

### 终端输入 ctrl+a,d 退出当前窗口

## bug解决

###  

## 1panel

### 关于配置反向代理

- 首先在网站-》网站-》新建网站

- 选择反向代理

-  

- 这里就可以进行反向代理了

- https

	- 如果要开启https的话需要最好改成 http可直接访问，不然可能会出现循环重定向情况

