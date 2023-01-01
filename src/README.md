# 简介 · Introduction

**Go-MC** 是一个用 Go 语言编写的用于与 Minecraft 交互的库。最初从网络库开始，用纯 Go 实现了 MC 基于 TCP 的网络通信协议。
并提供了一个 `bot` 包以方便编写命令行MC机器人程序。

Go-MC 也提供了用于编写服务端的框架 `server`，并且有一个[独立的仓库](https://github.com/go-mc/server)作为一个服务端实现。服务端的开发暂不包含在本教程中。

在使用 `bot` 包的过程中，随着发送消息、玩家移动、解析区块等等新功能实现的需要，
逐步对游戏使用的基础数据结构如 *基于JSON的聊天消息*, *NBT 二进制命名标签*, *区域储存格式 .mca 文件* 等编写了 Go 版本的实现。

- 在`net`包实现MC的网络传输协议及`VarInt`等数据结构。
- 在`chat`包实现基于 JSON 的聊天消息格式的编解码。
- 在`level`包实现表示区块的数据结构、BitStorage、调色板等等。
- 在`level/block`包提供了方块表示数据结构及方块状态表。
- 在`nbt`包实现基于反射的二进制命名标签格式编解码，提供与标准库`json`类似的API。
- 在`offline`包提供离线模式下将玩家名转换为UUID的算法。
- 在`save/region`包中实现了`.mca`文件的读取和写入。
- 而`yggdrasil`和`realms`包已经不再维护了。

在添加这些功能时，都同时提供了客户端和服务端侧的接口，例如我们的 RCON 库就提供了服务端的实现。
所以 Go-MC 不仅能用来编写客户端，也可以用来实现服务端。在 `server` 包中还提供了相应的框架。

## 翻译 · Translation

本文本应使用英语编写，但受限于英语表达水平与有限的精力，使用中文才能保证文档的清晰、准确与全面。
对于母语非中文的读者，请尝试使用浏览器机器翻译功能。若机器翻译的效果不足以能够理解文章，请告知我。

## 许可 · License

本作品采用 [知识共享署名-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-sa/4.0/) 进行许可。

[![知识共享许可协议](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)
