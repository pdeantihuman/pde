# io_uring

Linux 上已经有很多实现文件 IO 的方式了，例如最古老的 `read`, `write` 系统调用，POSIX 的 `pread`, `pwrite` API。
后来我们还有了基于向量的 POSIX API，`preadv` 和 `pwritev`。这还不够，他们有了升级版 `preadv2` 和 `pwritev2` 使得系统调用支持 flag。但是以上所有都是同步 API，也就是当他们执行完成 IO 后才会返回。而我们更想要异步 API。POSIX 有了 `aio_read` 和 `aio_write` 系统调用来满足这个需求。
但是 `aio_write` 和 `aio_read` 的当前实现并不能令人满意。详情可读 [aio 的缺陷](aio.md)。

## 开始

一开始我们的想法是改进原来的 aio 接口。

- 添加新接口需要等待审阅和通过，这是一个很漫长的过程。
- 改进旧接口可以省下来很多事。特别是测试框架。

现有的 aio 接口有三个接口，一个是 `io_context` 做初始化工作，一个 `io_submit` 提交 io 操作，和一个 `io_getevents` 去实际执行 io 或者获取事件。（用过 boost.asio 的朋友应该会觉得很眼熟）

因为我们打算更改 aio 的行为，我们需要更改多个 aio 系统调用，因此我们需要一个新的系统调用将数据传入内核。

## 新 API 的设计目标

1. 易用，难以错用。
2. 可拓展
3. 包含新特性。
4. 高性能
5. 可伸缩

由于我们不想要拷贝，因此我们必须使用一种方式优雅地让用户和内核共享 IO 本身和完成事件的结构。我们不想要系统调用。
满足这种苛刻需求的数据结构是单消费者和单生产者的共享环形缓冲区。这样我们可以消除程序和内核之间共享锁定的需要。

io_uring 有两个关键数据结构，对应两个事件：用户向内核提交请求，内核产生的完成事件。前者的消费者是内核，后者的消费者是用户。因此，我们需要一对环形缓冲区，提交队列(SQ)和完成队列(CQ)。

CQ 上的事件的内存布局是这样的。cqe 主要需要包含的内容是 io 的结果，io 请求的标识符，
```c
struct io_uring_cqe {
	__u64 user_data; // 请求的信息
	__s32 res; // 请求的结果
	__u32 flags; // 成功则返回字节数，失败则返回错误的类型（负值）
}
```

请求的结构则要复杂得多，我们现在是这个样子的

```c
struct io_uring_sqe {
	__u8 opcode;
	__u8 flags;
	__u16 ioprio;
	__s32 fd;
	__u64 off;
	__u64 addr;
	__u32 len;
	union {
		__kernel_rwf_t rw_flags;
        __u32 fsync_flags;
        __u16 poll_events;
      	__u32 sync_range_flags;
      	__u32 msg_flags;   
   	};
   	__u64 user_data;
   	union {
	   	__u16 buf_index;
   		__u64 __pad2[3];
   	};
};
```

