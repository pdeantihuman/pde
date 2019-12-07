# 面试准备

首先一般一个 http 服务器的原理都是，从url解析应该委托给哪个handler, 然后 handler 将数据写入一个http response writer。所以，为了recover的话，在委托那一块写一个 recover() 就可以阻止整个程序崩溃了。

```go
func RecoverHandler(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    defer func(w, r) {
      if request := recover(); request != nil {
      httplog.Error(r, fmt.Sprintf("%v\n%s", result, debug.Stack()))
      w.WriteHeader(http.StatusInternalServerError)
    }
    }()
    next.ServerHTTP(w, r)
  }) 
}
```

