# 基础功能 · Basic Module

在 `bot` 包实现了最基础的登录逻辑之后，其他可选的功能都放在子包下以便按需引用，
其中一个几乎必选的子包就是 `bot/basic` ，从名字你就能听出来它提供的功能有多基础了。

`bot/basic` 包提供了以下功能：

- KeepAlive 用于服务器测试连接状态，客户端不处理会被踢出游戏
- Client Settings 向服务器发送的一些客户端设置，例如语言、主手、装备可见性、客户端名称等
- Player Info 储存玩家的实体ID，游戏模式等
- World Info 储存服务器信息，包括世界维度、视距、最大玩家总数等

整个框架以多级模块化的方式构建，`bot` 下的各个包并不是完全平级的关系，目前 `bot/msg` 和 `bot/world` 就都依赖 `bot/basic` 包。
各个模块之间的依赖在初始化时显式指定，不需要特殊关注。

## 使用方法 · Usage

功能模块的加载需要在 `JoinServer` 之前完成。

```diff
  package main

  import (
      "log"

      "github.com/Tnze/go-mc/bot"
  )

  var (
      client *bot.Client
      player *basic.Player
  )

  func main() {
      client = bot.NewClient()

+     player = basic.NewPlayer(client, basic.DefaultSettings, basic.EventsListener{})

      err := client.JoinServer("localhost:25565")
      if err != nil {
          log.Fatal(err)
      }

      log.Println("Login success")

      var perr bot.PacketHandlerError
      for {
          if err = client.HandleGame(); err == nil {
              panic("HandleGame never return nil")
          }
          if errors.As(err, &perr) {
              log.Print(perr)
          } else {
              log.Fatal(err)
          }
      }
  }
```

通过调用 `basic.NewPlayer()` ，`basic` 会将 `player` 对象注册到传入的 `client` 上，
`client` 收到响应的数据包会自动解析之后储存到 `player` 内部以供读取。

> 现在，`client` 和 `player` 是两个单独的对象，`client` 相当于打开的游戏程序窗口，而 `player` 相当于加入服务器的一个游戏会话，
> 前者承载了网络连接与客户端设置，后者承载了游戏过程中角色的状态数据。这是一种合理的设计，但一开始并不是显而易见的。

## 监听事件 · Events

`basic` 包还提供了几个可供注册的事件，如**游戏开始**，**血量变化**，**角色死亡**等。
以血量变化为例，在调用 `basic.NewPlayer()` 传入事件响应函数即可。

```go
func main() {
	// ...
    player = basic.NewPlayer(client, basic.DefaultSettings, basic.EventsListener{
        HealthChange: onHealthChange,
    })
	// ...
}

func onHealthChange(health float32) error {
    log.Printf("HealthChanged: %v", health)
    return nil
}
```

完整的例子可在 [go-mc/examples/daze](https://github.com/Tnze/go-mc/tree/master/examples/daze) 查看
