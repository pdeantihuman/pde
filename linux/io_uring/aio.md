# aio 的缺陷

## aio 仅支持 O_DIRECT 模式

O_DIRECT 模式禁用了缓存，而更多时候我们需要的 buffered aio。

## aio 还是会阻塞

aio 并不能保证你使用方式正确的话就不会在 submission 时阻塞。

## aio 的 API 很糟

aio 的 API 设计导致 overhead 很大而且进行一次 io 至少需要两次 syscall。