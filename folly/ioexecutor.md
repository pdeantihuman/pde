# IOExecutor

IOExecutor是在至少一个EventBase上运行的执行器。

其中一个EventBase应该可以通过getEventBase()访问。

调用getEventBase()返回的event base依赖于实现。

请注意，IOExecutors不一定自己在base上循环——例如，EventBase本身是IOExecutors，但不驱动自己。

IOExecutor的实现有资格成为全局IO执行器，每次调用getIOExecutor()时都会通过setIOExecutor()返回。

这些函数是在GlobalExecutor.h中声明的。如果调用getIOExecutor，但没有设置任何函数，将创建并返回一个默认的全局IOThreadPoolExecutor。