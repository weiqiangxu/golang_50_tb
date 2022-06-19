
### init的特性

不能显示调用

无参数无返回值

一个包内可以有多个且是顺序执行 - 执行顺序取决于源文件传递编译器的顺序 - 不要依赖init的执行次序

只会执行一次


### init的初始化顺序

main(import pkg1) -> pkg1(import pkg2) -> pkg2(import pkg3)

1. 深度优先 - pkg3的init是最先被执行初始化的(常量 -> 变量 -> init函数)

2. main包最后被初始化

3.  init 函数适合做包级数据初始化和初始状态检查 , 执行顺位排在其所在包的包级变量之后（常量和变量都初始化了）,这个其实意思就是在init函数之中可以对包级变量或常量操作 - 因为都已经在init之前初始化好了;


```
main 包依赖 pkg1 和 pkg3；
pkg1 依赖 pkg2
```

```
pkg2: const c init
pkg2: var v init
pkg2: init
pkg1: const c init
pkg1: var v init
pkg1: init
pkg3: const c init
pkg3: var v init
pkg3: init
main: const c1 init
main: const c2 init
main: var v1 init
main: var v2 init
main: init
```

```
pkg2 -> pkg1 -> pkg3 -> main 的包顺序 以及 包内“常量” -> “变量” -> init 函数
```



### init的用法

1. 重置包级变量值

2. 对包级变量进行初始化，保证其后续可用

3. init 函数中的“注册模式”

4. init 函数中检查失败的处理方法

当init检查状态不正确的时候一般建议直接调用 panic。或通过 log.Fatal 等方法记录异常日志后再调用 panic 使程序退出



### init 函数的注册模式

典型的应用是将 github.com/go-sql-driver/mysql 驱动注入框架之中，作为 database/sql 的实现

import第三方包github.com/go-sql-driver/mysql 的时候执行该包内的 init func 

init调用database/sql.Register func将第三方包的mysqlDriver传递进入database/sql


示例代码

```
import (
	"database/sql"
	"time"

	_ "github.com/go-sql-driver/mysql"
)


db, err := sql.Open("mysql", "user:password@/dbname")
if err != nil {
	panic(err)
}
// See "Important settings" section.
db.SetConnMaxLifetime(time.Minute * 3)
db.SetMaxOpenConns(10)
db.SetMaxIdleConns(10)
```


```
// github.com/go-sql-driver/mysql@v1.6.0/driver.go
func init() {
	sql.Register("mysql", &MySQLDriver{})
}
```












