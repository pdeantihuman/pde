# 从 Go 的 `/etc/hosts` 谈起

我们都知道 Go 的最大亮点是它使用了 goroutine 这个机制。goroutine 这个机制充分发挥了 `epoll` 的长处，goroutine可以由Go的用户态调度器调度，而无需一直由内核级的调度器调度。它是所谓的有栈协程技术。