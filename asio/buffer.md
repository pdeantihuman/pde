# 缓冲区

boost.asio 的缓冲区有两个重要概念

1. *scatter* `read` 将数据读入多个缓冲区。
2. *gather* `write` 写入传输多个缓冲区。

从根本上来说，IO涉及到内存中相邻区域之间数据的传输。缓冲区可以简单地用地址+缓冲区字节大小来表示。但是为了开发高效网络程序，请使用 boost.asio 对 *gather* 和 *scatter* 的支持。

我们需要抽象来表示缓冲区的集合。boost.asio 的策略是定义了两个类型来表示一个缓冲。

boost.asio 作出了一个区分，区分可变的内存区域和不可变的内存区域。

```cpp
// 可变缓冲区
typedef std::pair<void*, std::size_t> mutable_buffer;
// 不可变缓冲区
typedef std::pair<const void*, std::size_t> const_buffer;
```

这里，可变缓冲区可以转换为常量缓冲区，但是反方向的转换是无效的。

然而，boost.asio 没有按原样使用上述定义，而是定义了两个类: `mutable_buffer` 和 `const_buffer` 。其目的是提供连续存储器的不透明表示，其中:

1. 类型在转换中的行为与 `std::pair` 相同。也就是说，可变缓冲区可转换为常量缓冲区，但不允许相反的转换。
2. 缓冲区溢出有保护措施。给定一个缓冲区实例，用户只能创建另一个代表相同内存范围或其子范围的缓冲区。为了提供进一步的安全性，该库还包括从POD元素的数组、`boost::array`或`std::vector`或`std::string`自动确定缓冲区大小的机制。
3. 使用`data()`成员函数显式访问底层内存。一般来说，应用程序不应该这样做，但是库实现需要将原始内存传递给底层操作系统函数。

最后，通过将缓冲区对象放入容器中，可以将多个缓冲区传递给分散收集操作(如read()或write())。已经定义了可变缓冲区序列和常量缓冲区序列概念，以便可以使用容器，如`std::vector`、`std::list`、`std::array`或`boost::array`。

> 这里 boost.asio 做了两个工作，一个区分了可变的缓冲区和不可变的缓冲区，另一个是配置了一个自动确定缓冲区大小的机制防止缓冲区溢出。

<!-- 因此，我们需要抽象来表示缓冲区的集合。升压中使用的方法。Asio是定义一个类型(实际上是两个类型)来表示一个缓冲区。这些可以存储在一个容器中，该容器可以被传递给分散-聚集操作。 -->

<!-- 除了将缓冲区指定为指针和字节大小之外，还可以使用Boost。Asio区分了可修改内存(称为可变内存)和不可修改内存(后者是从常量限定变量的存储中创建的)。因此，这两种类型可以定义如下: -->

## Streambuf: 支持 iostream

boost::asio::basic_streambuf类是从std::basic_streambuf派生的，用于将输入序列和输出序列与某个字符数组类型的一个或多个对象相关联，这些对象的元素存储任意值。这些字符数组对象是streambuf对象的内部对象，但是提供了对数组元素的直接访问，以允许它们用于输入/输出操作，例如套接字的发送或接收操作:

streambuf的输入序列可通过data()成员函数访问。该函数的返回类型满足常量缓冲序列要求。

streambuf的输出序列可通过prepare()成员函数访问。该函数的返回类型满足可变缓冲序列要求。

通过调用commit()成员函数，数据从输出序列的前面传输到输入序列的后面。

通过调用consume()成员函数，从输入序列的前面移除数据。

streambuf构造函数接受size_t参数，该参数指定输入序列和输出序列大小之和的最大值。如果成功，任何将内部数据增长到超过此限制的操作都将引发std::length_error异常。

```cpp
// 片段的功能是从 socket 读取一段数据转换为 std::string
//// 先创建 streambuf 对象
boost::asio::streambuf sb;
...
// 从 socket 读取数据，长度为 n，源是 sock，缓冲区为 sb
std::size_t n = boost::asio::read_until(sock, sb, '\n');
// 从 streambuf 中读出数据
boost::asio::streambuf::const_buffers_type bufs = sb.data();
// 转换为 std::string。这里解释了如何获取常值缓冲区的地址。
std::string line(
    boost::asio::buffers_begin(bufs),
    boost::asio::buffers_begin(bufs) + n);
```

