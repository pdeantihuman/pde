# Stateful Goroutine

在前面的例子中，我们使用mutex的显式锁定来同步多个goroutines之间对共享状态的访问。另一种选择是使用goroutines和channel的内置同步功能来实现相同的结果。这种基于channel的方法与Go的共享内存的想法是一致的，即通过通信将每一段数据完全归1个goroutine所有。

在这个例子中，我们的状态将由一个goroutine所有。这将保证数据不会因并发访问而损坏。为了读取或写入该状态，其他goroutine将向拥有goroutine的用户发送消息并接收相应的回复。这些readOp和writeOp结构封装了这些请求，并为所有goroutine提供了一种响应方式。

和以前一样，我们将计算我们执行了多少操作。

其他goroutines将使用 `read` 和 `write` channel 分别发出读取和写入请求。
这是拥有该状态的goroutine，它是一个与前面示例相同的映射，但现在是有状态goroutine的私有映射。这个goroutine反复选择读和写通道，在请求到达时做出响应。通过首先执行所请求的操作，然后在响应信道上分别发送一个值来指示成功(以及在读取的情况下的期望值)，来执行响应。
这将启动100个goroutine通过reads通道向stateful goroutine发出读取。每次读取都需要构建一个readOps，通过 `read` channel 发送，并通过提供的resp channel接收结果。

```go
type readOp struct {
  key int
  resp chan int
}

type writeOp struct {
  key int
  val int
  resp chan bool
}

func main {
  var readOps uint64
  var writeOps uint64
  reads := make(chan readOp)
  writes := make(chan writeOp)
  go func() {
    var state = make(map[int]int)
    for {
      selecct {
        case read := <- reads:
  	      read.resp <- state[read.keys]
      	case write := <- writes:
        	state[write.key] = write.val
        	write.resp <- true
      }
    }
  }()
  
  
}
```

