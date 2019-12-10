# LIFO Semaphore

LifoSem是一个信号量，它唤醒它的等待者的方式是为了最大化性能而不是公平性。当所有的等待者都相等时，应该首选 mutex + condvar 或POSIX sem_t 解决方案。它比condvar或sem_t更快，而且它有一个关闭状态，在关闭您的工作管道时，这可能会为您节省很多复杂性。LifoSem是sem_t的一个超集，但这只是因为它使用填充和对齐来避免错误共享。

LifoSem允许multi-post和multi-tryWait，并提供唤醒所有等待者的关机状态。LifoSem比sem_t更快，因为它执行精确的唤醒，所以它通常需要更少的系统调用。除了定时等待，它提供了sem_t的所有功能。它被称为后进先出，因为它的唤醒策略近似为后进先出，而不是通常的先进先出。