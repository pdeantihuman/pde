# Timer.5

如果你的处理程序需要很长时间才能完成，或者你的程序的多核拓展性不佳，你可能会用线程池去调用 `io_context::run()`。但是，那你如何去访问共享资源呢？这里我们介绍一种用 strand 模板实现的同步方式。

我们定义一个 printer 类。

```cpp
class printer{
public:
```

定义构造函数。接受一个参数 io_context。根据 io_context 构造 strand，然后根据 io_context 定义两个 timer 和一个 counter。

strand 类模板是一个执行器适配器。它提供的功能是提供断言：上一个handler完成后下一个handler才能开始。无论有多少个线程在执行 `io_context::run`，这一点都可以得到保障。但是，不同的链分发的handler是可以并发执行的。

```cpp
    printer(boost::asio::io_context& io)
    : strand_(boost::asio::make_strand(io)),
      timer1_(io, boost::asio::chrono::seconds(1)),
      timer2_(io, boost::asio::chrono::seconds(1)),
      count_(0)
    {
```

初始化异步操作时，所有的 handler 都会与一个 strand 对象用 bind 绑定起来。通过将这两个 handler 都绑定到一个 strand 上，我们可以保证他们两个不会并发执行。

```cpp
    timer1_.async_wait(boost::asio::bind_executor(strand_,
          boost::bind(&printer::print1, this)));

    timer2_.async_wait(boost::asio::bind_executor(strand_,
          boost::bind(&printer::print2, this)));
  }

  ~printer()
  {
    std::cout << "Final count is " << count_ << std::endl;
  }
```

因为是多线程程序，他们访问的共享资源是 counter 对象和 cout 对象。

```cpp
  void print1()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 1: " << count_ << std::endl;
      ++count_;

      timer1_.expires_at(timer1_.expiry() + boost::asio::chrono::seconds(1));

      timer1_.async_wait(boost::asio::bind_executor(strand_,
            boost::bind(&printer::print1, this)));
    }
  }

  void print2()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 2: " << count_ << std::endl;
      ++count_;

      timer2_.expires_at(timer2_.expiry() + boost::asio::chrono::seconds(1));

      timer2_.async_wait(boost::asio::bind_executor(strand_,
            boost::bind(&printer::print2, this)));
    }
  }
```