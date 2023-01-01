# 聊天消息 · Chat Message

聊天消息是游戏中玩家们的主要沟通方式。一些服务器要求玩家在进入后输入指令登录。
聊天功能不仅可用于向服务器内的玩家报告当前状态，也可以用于接收玩家给的指令，是机器人所必备的重要功能之一

为 `bot` 包启用消息功能需要引入 `bot/msg` 模块，并同时依赖于 `bot/basic` 和 `bot/playerlist`。

```go
var (
    chatHandler *msg.Manager
    playerList  *playerlist.PlayerList
)


playerList = playerlist.New(client)
chatHandler = msg.New(client, player, playerList, msg.EventsHandler{})
```

## 发送消息 · Send Messages

要发送消息，请调用 `msg.Manager` 上的 `SendMessage(msg string) error` 方法。

```go
if err := chatHandler.SendMessage("Hello, world"); err != nil {
	return err
}
```
## 接收消息 · Receive Messages

目前（1.19.3）在协议层面有三种消息可被接收：
- 玩家消息（Player Chat）：玩家发送的消息，可以被签名确保是由玩家本人发送的。
- 系统消息（System Chat）：系统发送的提示，如“Tnze加入了游戏”，分为显示在消息栏和屏幕中央两种。
- 伪消息（Disguised Chat）：在服务器后台用“/say”命令发送的消息。

以下是接收聊天消息的代码示例：

```go
chatHandler = msg.New(client, player, playerList, msg.EventsHandler{
    SystemChat:        onSystemMsg,
    PlayerChatMessage: onPlayerMsg,
    DisguisedChat:     onDisguisedMsg,
})


func onSystemMsg(msg chat.Message, overlay bool) error {
	log.Printf("System: %v", c)
	return nil
}

func onPlayerMsg(msg chat.Message, validated bool) error {
	log.Printf("Player: %s", msg)
	return nil
}

func onDisguisedMsg(msg chat.Message) error {
	log.Printf("Disguised: %v", msg)
	return nil
}
```

## 消息处理 · Handle Message

Minecraft 使用 JSON 格式储存结构化的聊天消息，在消息中可以包含颜色、加粗、斜体等格式，以及点击事件、鼠标悬浮显示内容等功能。

在 `go-mc/chat` 包提供了操作聊天消息的 API，请查看[处理聊天消息 · Chat](/chat/index.md)。
