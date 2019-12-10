# ThreadPoolExecutor

这个函数将用来池化线程。当它准备好消费工作时，它必须调用`thread->startupBaton.post()`。

## virtual

`virtual`说明符指定非静态成员函数是虚拟的，并支持动态分派。它只能出现在非静态成员函数初始声明的decl说明符序列中(即，当它在类定义中声明时)。

虚拟成员函数和抽象类的意思差不多，就是这个方法仅仅是一个接口，调用的话调用的是实现类的方法。但是这里禁止静态函数声明为virtual还是感觉挺奇怪的。更加根本的原因是，virtual函数选择实现的原理是根据对象指针去理解对象的类型是什么，从而去选择实现。

## `std::lock_guard`

类lock_guard是一个mutex包装器，它提供了一种方便的RAII风格的机制，用于在限定范围的块期间拥有mutex。

当一个lock_guard对象被创建时，它试图获得它被赋予的mutex的所有权。当控制权离开创建lock_guard对象的范围时，lock_guard被析构，mutex被释放。

lock_guard类不可复制。

## Baton

baton允许一个线程阻塞一次并被唤醒。捕获一次切换，在它的生命周期(从construction/reset到destruction/reset)中，baton必须或者每一次都被`post()`并`wait()`，或者根本不被`post()`或者`wait()`。

baton没有内部填充，只有4字节大小。任何避免错误共享的对齐或填充都取决于用户。

这基本上是一个精简的信号量，只支持对`sem_pos`t的一次调用和对`sem_wait`的一次调用。

非阻塞版本(MayBlock == false)通过在关键路径中仅使用load acquire和store release操作，以不允许阻塞为代价，提供了更高的速度。

当前`posix`信号量`sem_t`还不算太坏，但是这提供了更多的速度、内联、更小的大小、实现不会改变的保证以及与`DeterministicSchedule`的兼容性。通过更严格的生命周期，我们还可以添加一系列断言，帮助提前捕捉race condition。

停止n个线程，并将它们的线程放入stoppedThreads_ queue，并从线程列表中删除它们，同步或异步先决条件:线程列表锁_写锁定

## mutable

auto、regsiter、static和extern是C语言中的存储类说明符。typedef也被认为是C语言中的存储类说明符。C++也支持所有这些存储类说明符。除了这个C++之外，还添加了一个mutable的重要存储类说明符。

