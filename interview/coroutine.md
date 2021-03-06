# 协程

2018年以后，协程的概念火遍了编程圈。那么，为什么协程会占据如此重要的地位，协程究竟是什么，为什么以前没有协程呢？本文会依次解答这三个问题。

协程是一种用户态线程，由用户态的调度器实现协作式的调度。用户态的调度无需陷入内核，速度就会比 Linux 的调度器快很多。因此，协程快就快在超越了 Linux 的调度器，协程的速度超越了以前所有的系统级线程。

在操作系统课程中，我们知道，只要陷入内核，很容易就可以暂停线程，那么，协程的实现要怎么暂停线程呢？

既然是协作式的调度，自然是让线程自己去让渡控制权。Go 语言的 Goroutine 是通过在每个函数调用前插入代码实现让渡控制权的。每次函数调用，调度器都会判断 Goroutine 是否要让渡控制权。如果需要调度，则保存上下文，将其他的 Goroutine 切换过来，让其他的 Goroutine 执行。

为什么以前就没有协程呢？根本原因是 Linux 最原始的 IO 系统调用是同步阻塞的而不是异步非阻塞的。直到今天，Linux 仍然没有成熟的异步文件系统调用，只有成熟的异步网络系统调用，也就是 epoll。Linux 的阻塞系统调用阻塞的是整个线程，用户态的调度器也无法使它让渡调度权。在早期没有异步IO系统调用，协程就变成了一个不实用的东西。协程的思想和理论很早就有了，只是系统调用还没有准备好。

而后来，Linux 出现了 epoll。协程就有了用武之地。最早出现的著名 epoll 应用是 nginx，而后有了 nodejs，最后出现了 go。事实上 C++ 是有协程的。但是，nodejs 和 go 的协程有所不同。Nodejs 是单线程的，也就是说，所有的协程都绑定在一个线程上。这使得 nodejs 没有同步问题。而 go 的协程则可以运行在调度器管理的任何线程上。

请注意一点，虽然网络有了异步版系统调用，但是文件系统调用依然是同步的，所以，这些有协程的语言都只能额外用多线程模拟异步。go的实现方式是一旦你使用了文件api，该线程就会被该协程独占。因此，如果你在go中大量使用了同步的系统调用，就会导致go会创建大量的线程。这是写go一个需要警惕的地方。

当 nodejs 调用文件 api 时，nodejs 会在另一个线程上进行文件io。当 go 调用文件 api 时，go 会将这个协程与线程绑定。
