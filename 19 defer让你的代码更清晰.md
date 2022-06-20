
### defer 的运作机制

1. 只有在函数(和方法)内部才能使用 defer

2. 后进先出(LIFO)的顺序

3. 无论return还是panic存储到 deferred 函数栈中的函数都会被调度执行

4. 资源释放函数的 defer 注册动作紧邻着资源申请成功的动作

### defer 的常见用法

1. 拦截panic作recover，deferred 函数可以拦截到绝大部分的 panic，但有些 runtime 之外的致命问题也是无法拦截并恢复的

2. deferred 函数可以修改函数的具名返回值

3. deferred 函数可以用于输出一些调试信息


### 问题


1. 明确哪些函数可以作为 deferred 函数,比如 defer append(sl, 11) 那么会抛出error 。 但是可以通过匿名函数执行

```
defer func() {
	_ = append(sl, 11)
}()
```

2. defer的执行实际上是在return之后的

```
func TestDefer(t *testing.T) {
	s := foo()
	t.Log(s)
}

func foo() []int {
	s := []int{1}
	defer func() {
		s = append(s, 11)
	}()
	return s
}
```

输出结果

```
=== RUN   TestDefer
    pool_test.go:165: [1]
```

如果返回变量指针那么值就是改变的

```
func foo() *[]int {
	s := []int{1}
	defer func() {
		s = append(s, 11)
	}()
	return &s
}
```

打印的值

```
&[1 11]
```


3. 一道很有意思的题目


```
func TestDefer(t *testing.T) {
	foo1()
	foo2()
	p := []int{1}
	changeP(p)
	fmt.Println(p)
}

func changeP(a []int) {
	// 直接覆盖原来切片a里面的数组指针 - 原始数组指针对应的数据没有被改变所以
	// 函数内的赋值没有改变原来的值
	a = []int{2}
}

func foo1() {
	sl := []int{1, 2, 3}
	defer func(a []int) {
		// 本处存储的仍然是原来的切片底层存储的数组指针
		fmt.Println(a)
	}(sl)
	// 本处改变了sl的里面存储的数据-一个数组321的指针
	sl = []int{3, 2, 1}
	_ = sl
}
func foo2() {
	sl := []int{1, 2, 3}
	defer func(p *[]int) {
		// 打印的是sl的地址对应的数据
		// 而数据的获取是在defer之后也就是说这里在defer执行了获取到321数组的地址
		fmt.Println(*p)
	}(&sl)

	sl = []int{3, 2, 1}
	_ = sl
}
```


执行后的输出

```
[1 2 3]
[3 2 1]
[1]
```

切片直接替换 - 原有底层的数组的值并没有改变



4. defer 带来的性能损耗 - 使用 defer 的函数的执行时间是没有使用 defer 函数的 8 倍左右









