# 设计模式

## 池化

```go
conn := db.freeConn[0]
copy(db.freeConn, db.freeConn[1:])
db.freeConn = db.freeConn[:numFree-1]
conn.inUse = true
db.mu.Unlock()
if conn.expired(lifetime) {
  conn.Close()
  return nil, driver.ErrBadConn
}
conn.Lock()
err := conn.lastErr
conn.Unlock()
if err == driver.ErrBadConn {
  conn.Close()
  return nil, driver.ErrBadConn
}
```



## 单例模式

### Check-Lock-Check 模式

在C++和其他语言中，确保最小锁定并且仍然是线程安全的最佳和最安全的方法是在获取锁时使用众所周知的模式，称为检查-锁定-检查。模式的伪代码是这样的。

```go
if check() {
  lock() {
    if check() {
      // 执行关键区
    }
  }
}
```



这种模式背后的想法是，您应该首先检查，以最小化任何激进的锁定，因为IF语句比锁定便宜。其次，我们希望等待并获取排他锁，这样在同一时间只有一个执行在该块内。但是在第一次检查和获取排他锁之间，可能有另一个线程确实获取了锁，因此我们需要再次检查锁内部，以避免用另一个线程替换实例。

多年来，和我一起工作的人都很清楚这一点，在对这种模式和线程安全思想进行代码评审时，我对我的工程团队非常严格。

如果我们将这个模式应用到`GetInstance()`方法中，我们会得到如下结果:

```go
func GetInstance() *singleton {
  	// 因为不是完全原子的，因此不完美
    if instance == nil {     // <-- Not yet perfect. since it's not fully atomic
        mu.Lock()
        defer mu.Unlock()

        if instance == nil {
            instance = &singleton{}
        }
    }
    return instance
}
```

Check-Lock-Check 模式本身是挺好的，但是对对象的检查本身不是原子的（这是编译器决定的，因此我们应该用一个原子变量来实现。

这是一种更好的方法，但仍不完美。因为由于编译器优化，对实例存储状态没有原子检查。考虑到所有的技术因素，这仍然不是完美的。但是它比最初的方法好得多。

但是使用`sync/atom`包，我们可以原子地加载和设置一个标志来指示我们是否已经初始化了我们的实例。

```go
import "sync/atomic"
var used uint32

func GetInstance() *singleton {
  	// 因为不是完全原子的，因此不完美
  	if atomic.LoadUint32(&initialized) == 1 {     // <-- Not yet perfect. since it's not fully atomic
      // 上锁
        mu.Lock()
      // 释放锁
        defer mu.Unlock()
				
        if initialized == 0 {
            instance = &singleton{}
          atomic.StoreUint32(&initialized, 1)
        }
    }
    return instance
}
```



### 使用 Go 标准库

我们希望利用Go惯用的做事方式来实现这个单件模式。因此，我们必须看看名为sync的优秀标准库。我们可以找到一次类型。该对象将只执行一次操作，不再执行。下面你可以从 Go 标准库中找到源代码。

```GO
type Once struct {
  m Mutex
  done uint32
}

func(o *Once) Do(f func() {
  // 先检查是否已经执行过了
  if atomic.LoadUint32(&o.done) == 1 {
    return
  }
  // 如果未执行过，上锁
  o.m.Lock()
  defer o.m.Unlock()
  if o.done == 0 {
    // 执行完了上锁
    defer atomic.StoreUint32(&o.done, 1)
    f()
  }
})
```

这意味着我们可以利用棒极了的Go同步包只调用一次方法。因此，我们可以这样调用 `once.Do()` 方法:

```go
once.Do(func() {
  
})
```

```go
package singleon

import (
	"sync"
)

type singleton struct {
  
}

var instance *singleton
var once sync.Once
func GetInstance() *singleton {
  once.Do(func() {
    instance = &singleton{}
  })
  return instance
}
```

## fanout

### 引入

Go的并发原语使得构建有效利用输入/输出和多个处理器的流数据管道变得容易。本文给出了这些管道的例子，强调了操作失败时出现的微妙之处，并介绍了干净地处理失败的技术。

### 什么是 pipeline? 

Go中没有管道的正式定义；这只是多种并发程序中的一种。非正式地说，管道是由 channel 连接的一系列阶段，其中每个阶段是一组运行相同功能的goroutines。在每个阶段，goroutines会

- 通过入站 channel 从上游接收值
- 对数据执行一些功能，通常产生新值
- 通过出站 channel 向下游发送值

每个阶段都有任意数量的入站和出站 channel ，但第一阶段和最后一阶段除外，这两个阶段分别只有出站或入站 channel 。第一阶段有时被称为来源或生产者；最后一个阶段，水槽或消费者。

我们将从一个简单的管道示例开始解释这些想法和技术。稍后，我们将给出一个更现实的例子。

### 计算平方数

考虑一个有三个阶段的管道。

第一阶段gen是一个函数，它将整数列表转换为一个 channel ，该 channel 发出列表中的整数。gen函数启动一个goroutine，在 channel 上发送整数，并在所有值都发送完毕后关闭 channel :

```go
// gen 函数生产一个 channel
func gen(nums ...int) <-chan int {
    // 创建一个新的 channel 
    out := make(chan int)
    go func() {
      	// 依次将数据写入 channel
        for _, n := range nums {
            out <- n
        }
      	// 写入完毕后关闭 channel
        close(out)
    }()
    return out
}
```

第二级,sq,从一个 channel 接收整数，并返回一个 channel ，该 channel 发出每个接收到的整数的平方。在入站 channel 关闭并且该阶段已经向下游发送了所有值之后，它关闭出站 channel :

```go
// sq 接收一个 channel，返回一个 channel
// in 是被操作数列表，返回值是平方后的结果
func sq(in <-chan int) <- chan int {
  out := make(chan int)
  go func() {
    for n := range in {
      out <- n * n
    }
    close(out)
  }()
  return
}
```

### fan-out, fan-in

多个功能可以从同一个 channel 读取，直到该 channel 关闭；这叫做 fan-out 。这提供了一种在一组 worker 之间分配工作的方式，以并行化CPU使用和输入/输出

一个函数可以从多个输入中读取数据，并通过将输入 channel 多路复用到一个 channel 上继续进行，直到所有输入都关闭时，该 channel 才关闭。这叫做 fan-in 。

我们可以更改我们的管道来运行sq的两个实例，每个实例都从同一个输入 channel 读取。我们引入了一个新的功能，merge，以 fan-in 结果:

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}
```

merge 功能通过为每个入站 channel 启动一个goroutine将 channel 列表转换为单个 channel ，并将值复制到唯一的出站 channel 。一旦所有输出goroutine都已启动，merge就会再启动一个goroutine，以便在出站 channel 上的所有发送完成后关闭该 channel 。

在 closed channel 上发送紧急消息，因此在调用 close 之前确保所有发送都已完成非常重要 sync.WaitGroup 类型提供了安排这种同步的简单方法:

```go
// merge 允许merge 多个channel
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)
  
  	// 
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

### 快速停止

我们的管道功能有一个模式:

- 所有发送操作完成后，各阶段会关闭其出站 channel 。
- 阶段不断从入站 channel 接收值，直到这些 channel 关闭。

这种模式允许将每个接收级写入一个range循环，并确保一旦所有值成功发送到下游，所有goroutines都会退出。

但是在真正的管道中，每个阶段并不总是接收所有的入站值。有时这是有意为之的:接收者可能只需要一个值子集就能取得进展。更常见的情况是，一个阶段提前退出，因为入站值代表早期阶段的错误。无论哪种情况，接收器都不应该等待剩余值到达，我们希望早期阶段停止产生后期阶段不需要的值。

在我们的示例管道中，如果一个阶段未能使用所有入站值，试图发送这些值的goroutines将无限期阻塞:

```
    out := merge(c1, c2)
    // 只消费 merge 后的 channel 的第一值
    fmt.Println(<-out) // 4 or 9
    return
    // 因为我们不继续接受值了，因此其中一个线程就终止了
}
```

这是一个资源泄漏:goroutine消耗内存和运行时资源，goroutine堆栈中的堆引用防止数据被垃圾收集。Goroutines 无法被 gc；他们必须自己离开。

我们需要安排管道的上游阶段退出，即使下游阶段未能接收到所有入站值。一种方法是将出站 channel 更改为有缓冲区。缓冲区可以保存固定数量的值；如果缓冲区中有空间，发送操作将立即完成:

```go
c := make(chan int, 2) // buffer size 2
c <- 1  // succeeds immediately
c <- 2  // succeeds immediately
c <- 3  // blocks until another goroutine does <-c and receives 1
```

当 channel 创建时知道要发送的值的数量时，缓冲区可以简化代码。例如，我们可以重写gen，将整数列表复制到缓冲 channel 中，避免创建新的goroutine:

```go
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
  	// close 后的 buffered channel 仍然是可以使用的
    return out
}
```

回到我们管道中被阻塞的goroutines，我们可以考虑向`merge`返回的出站 channel 添加一个缓冲区:

```
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int, 1) // enough space for the unread inputs
    // ... the rest is unchanged ...
```

