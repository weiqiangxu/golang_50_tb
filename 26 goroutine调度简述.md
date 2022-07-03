### GPM调度


1. Java c++的多线程类库本质上是对操作系统的内核线程封装的，上下文切换会有较大开销，尤其是线程数多过CPU核心数


2. goroutine 是 GO 运行时管理的用户层轻量级线程， 不是操作系统线程，对cpu而言是无感知goroutine的


3. 调度其实就是计算机资源分配，什么时候执行、什么时候停止执行，谁先执行


4. 每个goroutine也就消耗2k字节,go运行时runtime负责调度,g/p/m定义在源码runtime/runtime2.go


5. 调度模型 GPM

```

G - 执行单元 Goroutine

P - 逻辑Processor - 内核与协程中间调度器 - 处理器

M -  Machine  - 内核线程封装

```


6. GO原生支持并发是因为golang不需要第三方类库其语言自身就支持并发