# Shadowaead

包shadowaead实现了一个简单的aead保护的安全协议。

一般来说，有两种类型的连接:面向流的和面向包的。面向流的连接(例如，传输控制协议)假设可靠有序的字节传递。面向分组的连接(例如，UDP)假设分组的传送不可靠且无序，其中每个分组要么原封不动地传送，要么丢失。

加密流以随机盐开始，以导出会话密钥，随后是任意数量的加密记录。每个加密记录具有以下结构:

```
    [encrypted payload length]
    [payload length tag]
    [encrypted payload]
    [payload tag]
```

有效载荷长度为2字节无符号大端整数，上限为0x3FFF (16383)。较高的2位被保留，必须设置为零。第一个AEAD加密/解密操作使用从0开始的计数随机数。每次加密/解密操作后，随机数会增加1，就好像它是一个无符号小端整数一样。
在面向数据包的连接上传输的每个加密数据包具有以下结构:

```
    [random salt]
    [encrypted payload]
    [payload tag]
```

盐用于派生一个子项来启动AEAD。数据包使用零随机数独立加密/解密。

在面向流和面向包的连接中，随机数和标签的长度根据使用的AEAD而变化。盐应该至少有16字节长。

## stream.go

定义一个常量 payloadSizeMask，这是 payload 的最大大小（单位是字节）

```go
const payloadSizeMask = 0x3FFF // 16*1024 - 1
```

writer 结构体是私有的。writer 包含 io.Writer 作为源 writer, cipher.AEAD 作为加密算法，nonce 应该是盐，buf是一个缓冲区用来转换未加密的流为

```go
// writer 结构体，数据将写入 writer 结构体
type writer struct {
  // 输出 writer
	io.Writer
  // aead 实例
	cipher.AEAD
  // 盐
	nonce []byte
  // 缓冲区，存放待加密的数据
	buf   []byte
}
```

```go
// NewWriter 调用 newWriter 构造新的 writer
func NewWriter(w io.Writer, aead cipher.AEAD) io.Writer { return newWriter(w, aead) }
```

```go
// 指定 writer 和 aead 实例构造 writer, 返回一个 writer 指针
func newWriter(w io.Writer, aead cipher.AEAD) *writer {
	return &writer{
		Writer: w,
		AEAD:   aead,
		buf:    make([]byte, 2+aead.Overhead()+payloadSizeMask+aead.Overhead()),
		nonce:  make([]byte, aead.NonceSize()),
	}
}
```

```go
// Write 方法接受一个字节数组
func (w *writer) Write(b []byte) (int, error) {
  // 将字节数组装载入缓存
  // bytes.NewBuffer 的返回值是一个 Buffer 指针
  // ReadFrom 方法读取 buffer 并加密内容，写入内置的 io.Writer 中
	n, err := w.ReadFrom(bytes.NewBuffer(b))
	return int(n), err
}
```

```go
// ReadFrom 方法接收一个参数 io.Reader
// 返回读取的字节数
func (w *writer) ReadFrom(r io.Reader) (n int64, err error) {
	for {
    // 先取 w 的缓冲区 slice
		buf := w.buf
    // buffer 取 [2 + overhead, 2 + overhead + max - 1] 正好 max 个
		payloadBuf := buf[2+w.Overhead() : 2+w.Overhead()+payloadSizeMask]
    // 数组读入 payload 的部分
		nr, er := r.Read(payloadBuf)

    // 如果读到的字节 > 0
		if nr > 0 {
      // n 的初始值为 0，读取后 n = n + nr 
			n += int64(nr)
			buf = buf[:2+w.Overhead()+nr+w.Overhead()]
			payloadBuf = payloadBuf[:nr]
      // 0 和 1 要存 payload 体积的大端表示
      // buf[0] 存 nr>>8, nr 的高 8 位
      // buf[1] 存 nr，nr 的低 8 位
			buf[0], buf[1] = byte(nr>>8), byte(nr) // big-endian payload size
			w.Seal(buf[:0], w.nonce, buf[:2], nil)
			increment(w.nonce)

			w.Seal(payloadBuf[:0], w.nonce, payloadBuf, nil)
			increment(w.nonce)

			_, ew := w.Writer.Write(buf)
			if ew != nil {
				err = ew
				break
			}
		}

		if er != nil {
			if er != io.EOF { // ignore EOF as per io.ReaderFrom contract
				err = er
			}
			break
		}
	}

	return n, err
}
```

## Seal

`w.Seal`加密和验证明文，验证附加数据，并将结果附加到dst，返回更新的切片。对于给定的密钥，随机数必须是`NounceSize()`字节长，并且在任何时候都是唯一的。
要为加密输出重用明文存储，请使用明文`[:0]`作为dst。否则，dst的剩余容量不得与明文重叠。

### io.reader

 reader 是包装基本读取方法的接口。

读取最多可读取`len(p)`字节到`p`。它返回读取的字节数`(0 <= n <= len(p))`和遇到的任何错误。即使 `Read` 返回`n < len(p)`，它也可能在调用期间使用所有`p`作为暂存空间。如果有些数据可用，但没有字节，`Read` 通常会返回可用数据，而不是等待更多数据。

当 `Read` 成功读取`n > 0`字节后遇到错误或文件结束情况时，它返回读取的字节数。它可以从同一个调用返回(非零)错误，或者从后续调用返回错误( `n == 0` )。这种一般情况的一个例子是，在输入流末尾返回非零字节数的 reader 可能返回 `err == EOF` 或`err == nil`。下一次读取应该返回`0`，`EOF`。

在考虑错误之前，调用方应该始终处理返回的 `n > 0` 字节。这样做可以正确处理读取一些字节后发生的输入/输出错误，以及两种允许的 `EOF` 行为。

除非`len(p) == 0`，否则不鼓励读取的实现返回零字节计数和`nil` 错误。调用方应该将返回 0 和 `nil` 视为表示什么都没有发生；特别是它并不表示 `EOF` 。

实现不能保留`p`。