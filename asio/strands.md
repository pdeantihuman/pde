# Strands: 你（不）需要显式锁定

链是事件处理程序的严格顺序调用。它是一种执行期对象，通过 `bind_executor` 助手函数将 executor 绑定到一个 completion handler 来让 handler 运行在这个执行器上。

如果只有一个线程调用了 `io_context::run`, 这就是一条隐式链，隐式链可以保证 handler 仅会在 `run()` 的内部被调用。

在单链的情况下，因为handler不可能并发执行，因为他们有严格的顺序，因此，没有线程安全问题。

在组合异步操作情况下，如果完成处理在一个链上，那么中间处理过程也应该在一个链上。这样可以保证线程安全。

所有异步操作通过get_associated_executor获取关联的执行器。

```cpp
boost::asio::associated_executor_t<Handler> a = boost::asio::get_associated_executor(h);
```

associated_exectuor 必须满足 Executor 的要求。异步操作会用这个executor，将异步操作submit或者post到这个executor上。

可以通过 `get_executor()` 来自定义执行器。

```cpp
class my_handler
{
public:
  // Executor 类型的自定义实现。
  typedef my_executor executor_type;

  // 返回一个自定义的 executor。
  executor_type get_executor() const noexcept
  {
    return my_executor();
  }

  void operator()() { ... }
};
```

在更复杂的情况下，associated_executor 需要直接指定偏特化。

```cpp
struct my_handler
{
  void operator()() { ... }
};

// 直接指定偏特化
namespace boost { namespace asio {
  template <class Executor>
  struct associated_executor<my_handler, Executor>
  {
    // Custom implementation of Executor type requirements.
    typedef my_executor type;

    // Return a custom executor implementation.
    static type get(const my_handler&,
        const Executor& = Executor()) noexcept
    {
      return my_executor();
    }
  };
} } // namespace boost::asio
```

`boost::asio::bind_executor()` 是用来将一个特定的执行器对象，例如strand，绑定到一个 completion handler 上的。