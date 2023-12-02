# 编解码 NBT

Go-MC 提供了一个功能完善的 NBT 操作库，可以用于方便、高效地处理各种NBT结构。这里会介绍这个 NBT 库提供的核心API。

使用 NBT 库，需要导入以下包：

```go
import "github.com/Tnze/go-mc/nbt"
```

[![Go Reference](https://pkg.go.dev/badge/github.com/Tnze/go-mc/nbt.svg)](https://pkg.go.dev/github.com/Tnze/go-mc/nbt)

该 NBT 库的用法与标准库的 `encoding/json` 库比较相似，查看文档及示例很容易理解，这里是一些注意事项。

不同类型的Go变量会被转换为不同的 NBT 标签，对应关系如下：

| Go 类型                   | NBT 标签       |
|-------------------------|--------------|
| bool, int8, uint8       | TagByte      |
| int16, uint16           | TagShort     |
| int32, uint32           | TagInt       |
| int64, uint64           | TagLong      |
| float32                 | TagFloat     |
| float64                 | TagDouble    |
| string                  | TagString    |
| struct, map             | TagCompound  |
| []bool, []int8, []uint8 | TagByteArray |
| []int32, []uint32       | TagIntArray  |
| []int64, []uint64       | TagLongArray |
| []T                     | TagList      |

NBT库内部使用反射来实现以上功能，但是实现了 `nbt.Marshaler` 和 `nbt.Unmarshaler` 接口的值不会被反射，NBT 库会调用它们自身的实现。

你可以通过实现这两个接口实现自定义类型的支持。NBT库也有三个特殊类型实现了这两个接口，你可以利用它们完成一些特殊功能：

- `nbt.RawMessage` - “原始数据”类型，NBT 数据将被原样保存，可以容纳任意合法的NBT数据。
- [`nbt.StringifiedMessage`](./snbt.md) - 以 S-NBT 格式保存数据，可以与 `string` 相互转换。
- [`dynbt.Value`](./dynbt.md) - “动态”类型，可以保存任意数据，并且提供了方便的API用于操控 NBT。

## 结构体标签和选项 · Supported Struct Tags and Options

你可以给结构体标记 Tag 以指定 nbt 库如何将其编码为 NBT 及 如何将 NBT 解码到其之上。目前支持的标签有以下两个：

- `nbt` - 用于指定名称和参数
- `nbtkey` - 只用于指定名称 （可以包含逗号 `,` ）

### The `nbt` tag

大多数情况下，你只需要用这个 tag 来设置 NBT 标签名。

`nbt` tag 的格式： `<nbt tag>[,opt]`.

这是一个逗号分隔的列表。
第一项是标签名，其余部分为选项。

像这样：
```go
type MyStruct struct {
    Name string `nbt:"name"`
}
```

使用 `-` 可以让 NBT 库忽略某个字段：
```go
type MyStruct struct {
    Internal string `nbt:"-"`
}
```

使用 `omitempty` 可以让 NBT 库仅在为零值时忽略某个字段：
```go
type MyStruct struct {
    Name string `nbt:"name,omitempty"`
}
```

类型为 `[]byte`， `[]int32` 和 `[]int64` 的值默认将被分别编码为 `TagByteArray`， `TagIntArray` 和 `TagLongArray`。  
你可以通过 `list` 标记将它们指定为 `TagList`：
```go
type MyStruct struct {
    Data []byte `nbt:"data,list"`
}
```

### The `nbtkey` tag

官方 JSON 标准库有一个严重的问题：无法指定包含逗号的key（逗号后面的会被识别为其他选项）
（例如 `{"a,b" : "c"}`）

Go官方是非常自傲的，仅仅因为他们“觉得”key里不应该有逗号，他们就不让你加逗号，尽管JSON标准是完全支持的。  
但是我们不一样，对这种需求我们提供了支持的方案：

```go
type MyStruct struct {
    AB string `nbt:",omitempty" nbtkey:"a,b"`
}
```