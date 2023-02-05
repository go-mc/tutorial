# 聊天 · Chat

Go-MC 在 `go-mc/chat` 包的提供了操作聊天消息的 API。

```go
import "github.com/Tnze/go-mc/chat"
```

其中 `chat.Message` 是主要结构，每个 `chat.Message` 结构体可表示一条聊天消息。

## 使用 · Usage

该结构体实现了 `pk.Filed` 接口，可以直接用于网络包字段的发送与接收：

```go
func HandleDisguisedChatMessagePacket(p pk.Packet) error {
    var (
        Message chat.Message
        ChatType pk.VarInt
        ChatTypeName chat.Message
        TargetName pk.Option[chat.Message, *chat.Message]
    )
    err := p.Scan(&Message, &ChatType, &ChatTypeName, &TargetName)
    // ...
}
```

该结构体通过与 `json` 标准库交互，也可以实现和 `[]byte` 互转：

```go
func ToJson(msg chat.Message) (data []byte, err error) {
    return json.Marshal(msg)
}

func FromJson(data []byte) (msg chat.Message, err error) {
    err = json.Unmarshal(data, &msg)
    return
}
```

通过 `fmt` 库可以直接输出带样式的消息文本，原理是 `.String()` 方法将颜色等样式转换为[ANSI 转义序列](https://en.wikipedia.org/wiki/ANSI_escape_code)。
所以在类 Unix 系统上的大部分终端中，都支持通过 `fmt.Print(msg)` 直接实现输出带样式的聊天消息。
注意 **为了支持 Windows 系统** ，需要使用 [go-colorable](https://github.com/mattn/go-colorable) 这个库。

如果你需要将 `chat.Message` 转换为 **不包含 ANSI 转义序列** 的纯文本，则可以选择调用 `.ClearString()` 方法。

## 构造 · Construction

聊天消息按使用到的特性可分为如下三种

- 普通消息
- 附加消息
- 翻译消息

### 普通消息 · Normal

普通消息的格式很简单，仅仅是包含了 `.Text` 字段（而不包含 `.Translate`，`.With`，`.Extra` 等）。

```go
msg := chat.Message {
    Text: "hello, world",
}
```

```
hello, world
```

普通消息像其他消息一样，也可以包含一些样式，包括：

- 加粗：Bold
- 倾斜：Italic
- 下划线：UnderLined
- 删除线：StrikeThrough
- 随机化：Obfuscated
- 指定字体：Font
- 指定颜色：Color

### 附加消息 · Extra

附加消息就是将一个或多个另外的额外消息连接在主消息之后。可以用于对消息不同部分设置不同样式。

附加的部分都放在 `.Extra` 字段中：

```go
msg := chat.Message {
    Text: "hello",
    Extra: []chat.Message {
        chat.Message {
            Text: ", ",
        },
        chat.Message {
            Text: "world",
        },
    },
}
```

```
hello, world
```

### 翻译消息 · Translate

翻译消息与普通消息完全不同，它 **不包含 `.Text` 字段** ，取而代之的是 `.Translate` 和 `.With`。

翻译消息用于向设置不同语言的客户端展示不同的消息，其原理可以简单理解为格式化字符串（`printf`）。

例如，玩家加入服务器时服务器会发送一条 `.Translate == "multiplayer.player.joined"` 的消息。
该消息在中文客户端中内容为 `"%s加入了游戏"`，而在英语客户端中则为 `%s joined the game`。而玩家名则通过 `.With` 字段提供。

以下是一个简单的例子：

```go
msg := chat.Message {
    Trasnlate: "multiplayer.player.joined",
    With: []chat.Message {
        chat.Message {
            Text: "Tnze",
        },
    },
}
```

```go
Tnze加入了游戏
```

在这里 `Tnze` 会被填充到翻译消息的 `%s` 处，最终在中文客户端中显示为 `Tnze加入了游戏`，而在英语客户端中显示为 `Tnze joined the game`。

*所有可用的消息都可以在 `go-mc/data/lang` 下找到。*

> 如果你想用 Go-MC 实现一个自己的服务端，但是又想为自定义消息提供多语言功能，默认提供的翻译消息列表可能就不够用了。
> 你可以通过读取客户端发送的 `ClientInformation` 包内的 `Local` 字段获知客户端设置的语言，并在服务端进行翻译工作。

### 快捷方式 · Shortcrust

Go-MC 提供了一些常用的构造函数，用来快速生成你想要的消息结构体。我相信，这些函数不需要说明也能很容易的被理解和使用。

```go
chat.Text("hello, world")
chat.Text("hello").Append(chat.Text(", "), chat.Text("world"))
chat.Text("Waring").SetColor(chat.Yellow)
chat.TranslateMsg("multiplayer.player.joined", chat.Text("Tnze"))
```

## 更多 · More

聊天消息还支持悬停事件、点击事件等功能，详细的使用方法请参考 <https://wiki.vg/Chat> 以及 `go-mc/chat` 包源代码。
