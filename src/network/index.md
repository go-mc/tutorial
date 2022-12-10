# 网络连接 · Network

> 本章假设读者理解 TCP 协议的工作方式并有一定使用经验。  
> 关于 Minecraft 网络协议的详细说明可阅读 <https://wiki.vg/Protocol>。

众所周知，Minecraft Java 版使用的是基于 TCP 的网络传输协议，而 TCP 具有点对点、传输可靠、保证顺序、不保留消息边界等特点。
除了消息边界以外，这些特点同样适用于 Minecraft 协议，请在编写代码时时刻谨记。

Minecraft 在 TCP 的基础上构造了基于长度的分包、基于 zlib 算法的压缩以及基于 RSA 和 AES/CFB8 的加密等协议。

如需收发网络包，请导入 Go-MC 提供的网络库：

```go
import (
	"github.com/Tnze/go-mc/net"
	pk "github.com/Tnze/go-mc/net/packet"
)
```

特殊情况下，如需同时使用标准库的 `net` ，请为 Go-MC 的网络库设置别名：

```go
import (
    "net"
    mcnet "github.com/Tnze/go-mc/net"
    pk "github.com/Tnze/go-mc/net/packet"
)
```

本章提到的 `net` 包如无特殊说明均指的是 `go-mc/net` 包，而不是 Go 标准库的 `net` 包。

## 创建连接 · Create Connection

使用 `go-mc/net` 包创建连接的方式与使用 Go 标准库创建 TCP 连接的方式非常类似。

### 客户端 · Client

以下代码连接运行于 `localhost:25565` 的服务器：

```go
conn, err := net.DialMC("localhost:25565")
if err != nil {
    log.Fatal(err)
}
defer conn.Close()
```

### 服务端 · Server

以下代码在**所有本机 IP 地址的 25565 端口**监听连接：

```go
listener, err := net.ListenMC("0.0.0.0:25565")
if err != nil {
    log.Fatal(err)
}

for {
    conn, err := listener.Accept()
    if err != nil {
        log.Fatal(err)
    }
    go handle(&conn)
}
```

## 发送数据包 · Sending Packets

要发送数据包，你需要有一个网络连接 `net.Conn(conn)` 与一个数据包 `pk.Packet(p)`。
通过简单地调用 `conn.WritePacket(p)` 即可将数据包发送。

关于如何获得 `pk.Packet`，请参考下一节[数据打包 · Packing](packing.md)。

```go
err := conn.WritePacket(p)
if err != nil {
	log.Print(err)
	return
}
```

## 接收数据包 · Receiving Packets

接收数据包与发送数据包类似，你同样需要有一个 `net.Conn(conn)` 与一个 `pk.Packet(p)`。
不同的是这次的操作是从连接读取数据并写入数据包。

```go
var p pk.Packet
err := conn.ReadPacket(&p)
if err != nil {
	log.Print(err)
	return
}
```

之所以设计为需要传入 `&p` 而不是直接返回一个新的 `p`，是为了方便复用 `pk.Packet` 对象。

当循环接收数据包时，每个连接只使用一个 `pk.Packet`，可以复用内部缓冲区，起到减少内存分配，减轻 GC 压力的作用。

```go
var p pk.Packet
for {
	err := conn.ReadPacket(&p)
	if err != nil {
		log.Print(err)
		break
	}
	handle(p) // 需要注意 p 只在下次 ReadPacket 调用前有效
}
```
