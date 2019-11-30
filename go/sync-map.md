# Sync/Map 的分析

map就像 Go `map[interface{}]interface{ }`，但对于多个 Go 程序的并发使用是安全的，无需额外的锁定或协调。加载、存储和删除以摊余常数时间运行。

map类型是专用的。大多数代码应该使用简单的Go map ，带有单独的锁定或协调，以获得更好的类型安全性，并更容易维护 map 内容中的其他不变量。

 map 类型针对两种常见的使用情况进行了优化:

1. 给定key的条目只写入一次，但读取多次，例如在只增长的缓存中
2. 多个goroutines读取、写入和覆盖不相交key集的条目。在这两种情况下，与与单独的 mutex 或RWMutex配对的Go map 相比， map 的使用可以显著减少锁争用。

零map是空的，可以使用了。map首次使用后不得复制。

read包含对并发访问安全的 map 内容部分(有或没有mu保持)。read 字段本身总是可以安全加载的，但只能在mu保持的情况下写入。写入在read中的条目可以在没有mu的情况下同时更新，但是更新之前删除的条目需要将该条目复制到 dirty map 中，并在mu保持的情况下取消扩展。

dirty包含需要保存mu的 map 内容部分。为了确保 dirty map 可以快速升级到 read map ，它还包括 read map 中所有未删除的条目。删除的条目不会存储在 dirty map 中。在新值可以存储到干净 map 中之前，必须取消干净 map 中的已删除条目并将其添加到 dirty map 中。如果 dirty map 为零，下一次对 map 的写入将通过创建干净 map 的浅拷贝来初始化它，省略陈旧条目。

misses计算自上次更新read map 以来需要锁定mu以确定 key 是否存在的加载次数。一旦发生足够多的未命中来支付复制 dirty map 的成本， dirty map 将被提升到read map (处于未修改状态)，并且 map 的下一个存储将产生新的 dirty 拷贝。

```go
type Map struct {
  mu Mutex
  read atomic.Value // readOnly
  dirty map[interface{}]*entry
  misses int
}
```

```go
// readOnly 是 Map.read 中的一个不可变结构体
type readOnly struct {
  m map[interface{}]*entry
  // 如果 dirty map 中有不在 m 中的条目的话为 true
  amended bool
}
```

```go
// expunged 是一个任意指针，用于标记已从脏映射中删除的条目。
var expunged = unsafe.Pointer(new(interface{}))
```

```go
// entry 是 map 中对应于特定 key 的槽。
type entry struct {
  p unsafe.Pointer // *interface{}
}
```

p指向为条目存储的interface{}值。

如果`p == nil`，则条目已被删除，`m.dirty == nil`。

如果`p == expunged`，该条目已被删除，`m.dirty！=nil`，并且该条目从`m.dirty`中丢失。

否则，该条目有效并记录在`m.read.m.[key]` 中并且，如果`m.dirty != nil`，以 `m.dirty[key]` 表示。

一个条目可以用原子替换为`nil`来删除: 下一次创建dirty m时，它会用原子替换为`nil`来删除，并保留`m.dirty[key]`未设置。如果`p!=explunged`。如果`p==explunged`，条目的关联值只能在第一次设置m.dirty[key] = e后更新，以便使用 dirty map 查找找到条目。  

```go
func newEntry(i interface{}) *entry {
  return &entry{p: unsafe: Pointer(&i)}
}
```

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
  // 
  read, _ := m.read.Load().(readOnly)
  e, ok := read.m[key]
  if !ok && read.amended {
    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)
    e, ok = read.m[key]
    if !ok && read.amended [
      e, ok = m.dirty[key]
      m.missLocked()
    ]
  }
}
```

