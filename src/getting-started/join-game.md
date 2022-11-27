# 加入游戏 · Join Game

虽然看上去简单，但是玩家连接服务器、进入游戏其实是一个有点复杂的过程。在客户端和服务端创建连接成功后，还需要经过握手、认证、加密、压缩等几个过程，
经过一系列数据包的交换，登录阶段以一个服务器发给客户端的[0x02数据包](https://wiki.vg/Protocol#Login_Success)结束，玩家正式开始游戏。

处理登录实在有点烦人，不过好在 `bot` 包可以帮你完成整个步骤，而你只需要调用一个 `JoinServer` 函数，就能成功加入游戏！

首先快速看一下一个最基础的 bot 程序长什么样：

```go
package main

import (
	"log"

	"github.com/Tnze/go-mc/bot"
)

func main() {
	client := bot.NewClient()
	// ...

	err := client.JoinServer("localhost:25565")
	if err != nil {
		log.Fatal(err)
	}

	log.Println("Login success")

	if err = client.HandleGame(); err != nil {
		log.Fatal(err)
	}
}
```

## 第一步 · 创建客户端

调用 `bot.NewClient()` 函数会返回一个 `*bot.Client` 客户端对象，表示一个游戏客户端。没有需要特别注意的，
但是同一时刻一个客户端只能连接一个服务器，不能在多个 goroutine 中共用一个客户端连接服务器。

玩家加入游戏后客户端对象也会被用到，一般要把它保存在一个变量里以供后续使用，变量名一般是`client`或者`c`。

```go
client := bot.NewClient()
```

## 第二步 · 设置玩家名

更改玩家名的方法比较直接，但是请注意不要和 `client.Name` 搞混了， `client.Auth` 里的才是登入时发送给服务器的认证信息。
而 `client.Name` 是登录成功后服务器发回来的。

```go
client.Auth.Name = "Tnze"
```

## 第三步 · 登录正版账号

如果你想加入开启了正版验证的服务器（online-mode），还需要把 **UUID** 和 **AccessToken** 一起填上。

```go
client.Auth = bot.Auth{
    Name: "Tnze",
    UUID: "58f6356e-b30c-4811-8bfc-d72a9ee99e73",
    AsTk: "*******************",
}
```

以前 GO-MC 自带的 `yggdrasil` 包可以直接获取 AccessToken ，
但转为使用微软账户之后，这事儿变得有点麻烦[^Issue #106]，并且似乎需要调用浏览器或 WebView。


将这些东西集成在 Go-MC 里并不合适，所以请使用好心人编写的工具库，已知的库如下（排名不分先后，请自行选择判断），也欢迎新的库添加到这个列表来。

- <https://github.com/maxsupermanhd/go-mc-ms-auth>
- <https://github.com/BaiMeow/msauth>

[^Issue #106]: Issue [#106](https://github.com/Tnze/go-mc/issues/106)

## 第四步 · 加入游戏

调用 `JoinServer()` 并传入服务器地址即可加入游戏。服务器地址可以是域名也可以是IP地址，可以带端口号也可以不带（默认为25565）。
设计上，该地址字符串与原版游戏客户端加入服务器的地址输入框保持一致。

```go
err := client.JoinServer("localhost:25565")
if err != nil {
    log.Fatal(err)
}
```

## 第五步 · 处理游戏

在登录游戏成功后 `JoinServer()` 方法就会返回，如果什么都不做，程序就会结束，连接就会断开，自然也就达不到机器人的目的了。
所以在最后需要调用客户端上的 `HandleGame()` 方法，开始收发游戏数据包：

```go
if err := client.HandleGame(); err != nil {
    log.Fatal(err)
}
```

该方法会持续不断地接收数据包，并根据数据包ID执行相应的处理函数。
当任意处理函数产生了错误时 `HandleGame()` 就会返回，如果不希望程序就此退出，可以在处理完错误后再次调用 `HandleGame()` 继续游戏。
多次调用不会产生问题，设计如此，无需担心。

```go
var err error
for {
    if err = client.HandleGame(); err == nil {
        panic("HandleGame never return nil")
    }
	log.Printf(err)
}
```

在 `HandleGame()` 返回错误时，需要区分错误是否可恢复，如果是连接断开等不可恢复错误则需要停止程序，如果是单个数据包的处理出错则可以恢复。
判断返回的错误是否可以恢复的方法是使用 `errors.As()` 函数，判断 `err` 是否是一个 `bot.PacketHandlerError` ，具体使用方法如下：

```diff
  var err error
  var perr bot.PacketHandlerError
  for {
      if err = client.HandleGame(); err == nil {
          panic("HandleGame never return nil")
      }
+     if errors.As(err, &perr) {
+         log.Print(perr)
+     } else {
+         log.Fatal(err)
+     }
  }
```

## 下一步 · The next step

由于服务器的 “KeepAlive” 机制，当前编写的 bot 程序不会响应服务器的心跳包，就会在一定时间后会被服务器判定为无响应并自动踢出。
为了解决这个问题，需要使用下一章提到的 `bot/basic` 模块。