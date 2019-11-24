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

