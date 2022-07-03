### 其中到最终会发现老老实实的写 if err != nil 吧，最简单也最直观


### 文章中优化的方式

1. 封装公用函数并且在顶层 recover捕获 (捕获的同时要兼顾释放资源)

```
func check(err error) {
	if err != nil {
		panic(err)
	}
}

var r, w *os.File

defer func() {
    if r != nil {
        r.Close()
    }
    if w != nil {
        w.Close()
    }
    if e := recover(); e != nil {
        if w != nil {
            os.Remove(dst)
        }
        err = fmt.Errorf("copy %s %s: %v", src, dst, err)
    }
}()
```

2. 封装全局对象struct

```
type FileCopier struct {
	w   *os.File
	r   *os.File
	err error
}
所有的 func 执行后都往 err 填入值
然后统一判断 
if err != nil 

其实也是很不直观
```