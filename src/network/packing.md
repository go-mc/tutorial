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

操作起来有点繁琐，好在 Go-MC 为这样的操作提供了一个帮助函数：`pk.Marshal()`。经过改进的代码如下：

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

```go
var (
	x        pk.Double
	y        pk.Double
	z        pk.Double
	onGround pk.Boolean
)
r := bytes.NewReader(p.Data)
_, _ = x.ReadFrom(r)
_, _ = y.ReadFrom(r)
_, _ = z.ReadFrom(r)
_, _ = onGround.ReadFrom(r)
```

加上错误处理更加繁琐了！好在 Go-MC 也提供了一个帮助函数：`p.Scan()`。经过改进的代码如下：

```go
var (
    x        pk.Double
    y        pk.Double
    z        pk.Double
    onGround pk.Boolean
)
_ = p.Scan(&x, &y, &z, &onGround)
```

在实际使用中，一般需要根据情况选择是否使用帮助函数，请自行判断。

## 使用 `pk.Array`

在一些数据包格式中，会存在 `Array of X` 类型的数组字段。
这些数组的前一个字段通常是一个 `VarInt` 类型的长度字段， Go-MC 提供了一个帮助类型来处理这种常见情况。

以 Commands 包为例，目前该数据包格式如下：

| Field Name | Field Type      | Notes                                         |
|------------|-----------------|-----------------------------------------------|
| Count      | VarInt          | Number of elements in the following array.    |
| Nodes      | Array of `Node` | An array of nodes.                            |
| Root index | VarInt          | Index of the root node in the previous array. |

我们假设你已经定义好 `Node` 类型并为其实现了 `pk.Field` 接口，为了接收这个数据包中的 Nodes 数组，
我们可以采用以下繁琐的代码：

```go
var (
	count pk.VarInt
	nodes []Node
	root  pk.VarInt
)
r := bytes.NewReader(p.Data)

_, _ = count.ReadFrom(r)
nodes = make([]Node, count)
for i := range nodes {
	_, _ = nodes[i].ReadFrom(r)
}
root.ReadFrom(r)
```

为了使用 `p.Scan()` 简化工作量，我们需要用到 `pk.Array()` 函数：

```go
var (
	nodes []Node
	root  pk.VarInt
)
_ = p.Scan(
	pk.Array(&nodes),
	&root,
)
```

使用 `pk.Array()` ，会自动处理数组长度的 `VarInt`。返回值实现 `pk.Field` 接口，可用于 `pk.Marshal()` 与 `pk.Scan()`。  

注意，当切片的元素类型不支持相应调用的 `WriteTo` 和 `ReadFrom` 方法时，会 panic。
请确保将 `pk.Array()` 的返回值用作 `FieldEncoder` 时，切片元素类型也实现 `FieldEncoder`，
将 `pk.Array()` 的返回值用作 `FieldDecoder` 时，切片元素类型也实现 `FieldDecoder`。

## 使用 `pk.Option`

一些数据包格式中，会存在 `Optional X` 类型的可选字段。顾名思义，这种字段在数据包中是可选的。
可选字段是否存在需要根据上下文进行判断，上下文通常指其前面的 Boolean 值，例如：

| Field Name | Field Type      | Notes                           |
|------------|-----------------|---------------------------------|
| Is Signed  | Boolean         |                                 |
| Signature  | Optional String | Exist only if Is Signed is true |

读取该数据包的伪代码如下：

```go
IsSigned = ReadBoolean()
if IsSigned {
	Signature = ReadString()
}
```

为了能在 `p.Scan()` 函数调用中提供这样的判断逻辑，Go-MC 提供了四个帮助类型：`pk.Option` 、 `pk.OptionDecoder` 、 `pk.OptionEncoder` 和 `pk.Opt`。

`pk.Opt` 是一个一般不会用到的原始类型，如需使用请自行阅读注释及源码，在此不做说明。

`pk.Option` 是一个泛型类型，有两个泛型参数：`T` 和 `P`，其中后者是前者的指针类型。
`T` 必须满足 `pk.FieldEncoder` 接口，`P` 必须满足 `FieldDecoder` 接口。

例如：`pk.Option[pk.String, *pk.String]`，`pk.Option[pk.VarInt, *pk.VarInt]`等，都是合法的类型。

需要手动指定 `P` 是因为当前 Go 编译器泛型实现不够完善[^1]，在 Go 支持推导结构体泛型参数类型之后，
上面的两个例子就可以简单写成 `pk.Option[pk.String]` 和 `pk.Option[pk.VarInt]` 了。

[^1] [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md#pointer-method-example)

`pk.OptionDecoder` 和 `pk.OptionEncoder` 是两个变体，分别去掉了 对`T`的约束 和 对`P`的约束，很好理解。

以下给出读写上面例子数据包的完整代码

```go
var Signature pk.Option[pk.String, *pk.String]
_ = p.Scan(&Signature)
```
