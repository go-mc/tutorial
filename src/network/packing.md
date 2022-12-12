# 数据打包 · Packing

当连接建立成功，在 `net.Conn` 实例上能调用的方法有三个：

- 发送数据 `WritePacket(p pk.Packet) error`
- 接收数据 `ReadPacket(p *pk.Packet) error`
- 关闭连接 `Close() error`

> 在 Login 阶段还需要按协议调用 `SetCipher` 和 `SetThreshold`，这里不做讨论。

无论是发送数据还是接收数据，Minecraft 协议都是以数据包（`pk.Packet`）为单位处理的。
包长度、加密、压缩等传输细节[^Packet Format] Go-MC 都已经实现并封装到 `pk.Packet` 结构体内了，用户直接使用即可。

[^Packet Format]: 数据包格式：<https://wiki.vg/Protocol#Packet_format>

结构体 `pk.Packet` 定义如下：

```go
type Packet struct {
	ID   int32
	Data []byte
}
```

`ID` 是一个枚举值，表示数据包的功能，`Data` 的格式也需要根据它判断。可用的值在 `data/packetid` 中找到。

`Data` 逻辑上包含了一个或多个数据项，例如客户端在玩家移动时可能会发送的 Move 包可能格式如下：

| Field Name | Field Type | Notes                                                 |
|------------|------------|-------------------------------------------------------|
| X          | Double     | Absolute position.                                    |
| Feet Y     | Double     | Absolute feet position, normally Head Y - 1.62.       |
| Z          | Double     | Absolute position.                                    |
| On Ground  | Boolean    | True if the client is on the ground, false otherwise. |

即 `Data` 包含4个数据字段：三个 Double 和一个 Boolean。

所有的字段类型都定义在 `net/packet` 包，例如 `pk.Double`、`pk.Boolean`。
这些类型都实现了 `pk.Field` 接口，你可以调用它们的 `ReadFrom` 和 `WriteTo` 方法。

以下代码可以用来生成一个这样的数据包。

```go
var buffer bytes.Buffer
_, _ = pk.Double(x).WriteTo(&buffer)
_, _ = pk.Double(y).WriteTo(&buffer)
_, _ = pk.Double(z).WriteTo(&buffer)
_, _ = pk.Boolean(onGround).WriteTo(&buffer)

p := pk.Packet {
	ID: packetid.ServerboundMovePlayerPos,
	Data: buffer.Bytes(),
}
```

操作起来有点繁琐，不是吗。好在 Go-MC 为这样的操作提供了一个帮助函数：`pk.Marshal()`。经过改造的代码如下：

```go
p := pk.Marshal(
    packetid.ServerboundMovePlayerPos,
    pk.Double(x),
    pk.Double(y),
    pk.Double(z),
	pk.Boolean(onGround),
)
```

反过来，如果服务器想接收一个这样的数据包，那么需要用到 `ReadFrom` 方法，用笨办法就是如下这样：

// TODO

## 使用 `pk.Array`

// TODO

## 使用 `pk.Option`

// TODO
