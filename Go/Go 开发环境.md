
## 1.1. go的项目目录
进行`Go`语言开发的时候，我们的代码总是会保存在`$GOPATH/src`目录下。在工程经过`go build`、`go install`或`go get`等指令后，会将下载的第三方包源代码文件放在`$GOPATH/src`目录下， 产生的二进制可执行文件放在 `$GOPATH/bin`目录下，生成的中间缓存文件会被保存在 `$GOPATH/pkg` 下。

如果我们使用版本管理工具（`Version Control System`，`VCS`。常用如`Git`）来管理我们的项目代码时，我们只需要添加`$GOPATH/src`目录的源代码即可。`bin` 和 `pkg` 目录的内容无需版本控制。

## 1.2. go 开发结构
![[Go 开发环境.png]]

## 1.3. 目前流行的项目结构

通常使用顶级域名来作为包名的前缀，这样就不担心项目名冲突的问题了。
因为不是每个个人开发者都拥有自己的顶级域名，所以目前流行的方式是使用个人的github用户名来区分不同的包。
![[Go 开发环境-1.png]]

举个例子：张三和李四都有一个名叫studygo的项目，那么这两个包的路径就会是：

```
import "github.com/zhangsan/studygo"
```

和

```
import "github.com/lisi/studygo"
```

以后我们从`github`上下载别人包的时候，如：

```
go get github.com/jmoiron/sqlx
```

那么，这个包会下载到我们本地`GOPATH`目录下的`src/github.com/jmoiron/sqlx`。