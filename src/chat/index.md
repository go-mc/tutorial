# 聊天 · Chat

Go-MC 在 `go-mc/chat` 包的提供了操作聊天消息的 API。

```go
import "github.com/Tnze/go-mc/chat"
```

其中 `chat.Message` 是主要结构，每个 `chat.Message` 结构体可表示一条聊天消息。

可以用标准库简单地将结构体与 JSON 互转：

```go
func ToJson(msg chat.Message) (data []byte, err error) {
    return json.Marshal(msg)
}

func FromJson(data []byte) (msg chat.Message, err error) {
	err = json.Unmarshal(data, &msg)
	return
}
```

在 Go-MC 中构造一条纯文本消息：

```go
msg := chat.Text("hello, world")
```

## 翻译 · Translate

// TODO

## 连接 · Connection

// Extra
