# Go 使用定时器

```go
func NewCounterMonitor(ctx context.Context) chan <- int {
  ch := make(chan int)
  go func() {
    count := 0
    for{
      select {
      case i, ok := <- ch:
        if !ok {
          return
        }
        counter += i
      case <- ctx.Done():
        fmt.Printf("final_count: %d\n", counter)
        return
        
      }
    }
  }()
  return ch
}
```

