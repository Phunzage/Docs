---
title: Go 并发编程入门：goroutine 与 channel
tags:
  - Go
  - 并发
  - 教程
createTime: 2025/07/30 10:00:00
permalink: /blog/go/go-concurrency/
---

## 为什么 Go 天生适合写并发

在 C/Java 里写并发，要手动管理线程、加锁、处理共享内存，稍不留神就数据竞争、死锁。而 Go 选择了一条完全不同的路：采用 **CSP（Communicating Sequential Processes，通信顺序进程）** 并发模型，核心思想是**「通过通信共享内存」**，而不是**「通过共享内存来通信」**。

对新手来说，Go 的并发有三大优点：

- **goroutine 极轻量**：初始栈只有几 KB，可以轻松创建成千上万个，完全不怕"开线程开销大"
- **channel 是一等公民**：goroutine 之间用 channel 传数据，从设计上减少了对锁的依赖
- **语法极简**：并发只需要一个 `go` 关键字

## goroutine：用 go 关键字开启并发

Goroutine 是 Go 程序中最基本的并发执行单元。创建它只需要在函数调用前加一个 `go` 关键字：

```go
package main

import "fmt"

func hello() {
	fmt.Println("Hello Goroutine")
}

func main() {
	go hello() // 开启一个 goroutine
	fmt.Println("Hello Main Goroutine")
}
```

::: tip
`main` 函数本身运行在一个 **main goroutine** 中，`go hello()` 会再开一个新的 goroutine，两者**并发**执行，谁先执行完完全不确定。
:::

### 第一个坑：main goroutine 提前退出

运行上面的程序，输出可能是以下三种之一：

1. 先打印 `Hello Goroutine`，再打印 `Hello Main Goroutine`
2. 先打印 `Hello Main Goroutine`，再打印 `Hello Goroutine`
3. 只打印 `Hello Main Goroutine` —— 子 goroutine 没来得及执行

::: warning
如果 main goroutine 先执行完，**整个程序就结束了**，`hello` goroutine 可能来不及执行就被终止。初学者最常问的"为什么我的 goroutine 没执行？"，多半就是这个原因。
:::

那怎么让主程序"等等"子 goroutine？这就轮到 `sync.WaitGroup` 登场了。

## sync.WaitGroup：让主程序等一等

`sync.WaitGroup` 内部维护一个计数器，提供三个方法：

| 方法 | 作用 |
|------|------|
| `Add(n)` | 计数器 +n，通常表示要启动几个 goroutine |
| `Done()` | 计数器 -1，在 goroutine 内调用，表示自己干完了 |
| `Wait()` | 阻塞，直到计数器归零 |

### 等待单个 goroutine

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func hello() {
	fmt.Println("Hello Goroutine")
	wg.Done() // 计数器 -1
}

func main() {
	wg.Add(1) // 计数器 +1
	go hello()

	fmt.Println("Hello Main Goroutine")
	wg.Wait() // 阻塞，等计数器归零再退出
}
```

### 匿名函数 + 闭包陷阱

批量启动 goroutine 常写成匿名函数，但要注意**闭包**问题——在 Go 1.22 之前，循环变量 `i` 会被所有 goroutine **共享**，直接引用可能全部打印 10：

```go
for i := 0; i < 10; i++ {
	go func() {
		fmt.Println("hello", i) // ❌ 闭包捕获循环变量
		wg.Done()
	}()
}
```

::: warning
正确做法是把 `i` 作为参数传进去（沿用上面的 `wg`），值拷贝后互不干扰：

```go
for i := 0; i < 10; i++ {
	go func(n int) {
		fmt.Println("hello", n) // ✅ 值拷贝，互不干扰
		wg.Done()
	}(i)
}
```
:::

## Channel：goroutine 之间的"传送带"

光有 goroutine 还不够，它们之间怎么通信？Go 给出的答案是 **channel（通道）**，它遵循**先入先出（FIFO）**规则，就像一条传送带。

::: tip
channel 和 slice、map 一样属于**引用类型**，必须用 `make` 初始化。没初始化的 channel 是 nil，对它收发都会永久阻塞。
:::

### 无缓冲通道（同步通道）

```go
ch := make(chan int) // 无缓冲
ch <- 10             // 发送：阻塞，直到有 goroutine 来接收
x := <-ch            // 接收：阻塞，直到有 goroutine 来发送
```

无缓冲通道的收发必须**同时准备好**，就像两人面对面交接货物——这天然实现了**同步**。

### 有缓冲通道（异步通道）

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 1) // 容量为 1 的缓冲通道
	ch <- 10                // ✅ 不阻塞：缓冲区有空位
	fmt.Println("发送成功")
	x := <-ch
	fmt.Println(x) // 10
	close(ch)      // 关闭通道
}
```

缓冲通道像**带货架的传送带**：只要货架没满就能先放上去，不用干等别人来取。

### 通道操作速查

| 操作 | 语法 | 说明 |
|------|------|------|
| 创建 | `make(chan 类型, 容量)` | 无缓冲则不指定容量 |
| 发送 | `ch <- 值` | 向通道发送数据 |
| 接收 | `x := <-ch` | 从通道接收数据 |
| 关闭 | `close(ch)` | 关闭后不能再发送 |
| 长度/容量 | `len(ch)` / `cap(ch)` | 元素数量 / 缓冲区大小 |

::: warning
对**已关闭**的通道发送会 `panic`；对已关闭且为空的通道接收会得到**零值**。所以一般约定：**发送方**负责关闭通道。
:::

## for-range 遍历通道

持续从通道读数据，用 `for range` 最省事——它会一直读到通道被**关闭**为止：

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	go func() {
		for i := 0; i < 5; i++ {
			ch <- i
		}
		close(ch) // 必须关闭，否则 range 会死锁
	}()

	for v := range ch {
		fmt.Println(v) // 0 1 2 3 4
	}
}
```

::: warning
发送方必须记得 `close(ch)`。如果通道没人再写、也没人关闭，`for range` 会一直阻塞，最终报 `fatal error: all goroutines are asleep - deadlock!`。
:::

## select 多路复用：一次监听多个通道

`select` 类似 `switch`，但专门用于 channel 通信——同时监听多个通道，谁先就绪就执行谁：

```go
select {
case v := <-ch1:
	fmt.Println("从 ch1 收到:", v)
case ch2 <- 42:
	fmt.Println("向 ch2 发送成功")
default:
	fmt.Println("所有 channel 都不可用") // 非阻塞
}
```

- 会**阻塞**，直到某个 case 可以执行
- 多个 case 同时就绪时，**随机**选一个执行
- `default` 分支让 select 变成**非阻塞**操作，配合 `time.After` 还能实现超时控制

## 补充一句：共享内存与 Mutex

channel 适合"传数据"，但有些场景必须共享内存（比如多个 goroutine 累加同一个计数器）。此时要用 `sync.Mutex` 保护临界区：

```go
mu.Lock() // 加锁
x++       // 临界区：同时只能有一个 goroutine 执行
mu.Unlock()
```

::: warning
`x++` 并不是原子操作（实际是读取 → 修改 → 写入三步），不加锁时多个 goroutine 并发执行会产生**数据竞争（data race）**，最终结果会小于预期值。读多写少的场景可以用 `sync.RWMutex` 优化。
:::

## 总结

| 知识点 | 一句话记住 |
|--------|-----------|
| goroutine | `go 函数()` 即可并发，但 main 退出程序就结束 |
| sync.WaitGroup | `Add` + `Done` + `Wait` 三件套，等所有 goroutine 干完 |
| channel | `make(chan T, 容量)` 创建，`<-` 收发；无缓冲同步、有缓冲异步 |
| for range | 遍历通道直到关闭，发送方负责 `close` |
| select | 多通道多路复用，`default` 实现非阻塞 |
| Mutex | 共享内存要加锁，防止数据竞争 |

## 参考资料

- [Go 并发编程笔记（原文）](https://docs.phunzage.art/go-notes/concurrency/)
- [Effective Go：并发](https://go.dev/doc/effective_go#concurrency)
- [Go by Example: Goroutines](https://gobyexample.com/goroutines)
