# 快速上手 · Getting Started

编写自动化 Minecraft 机器人程序是 Go-MC 开发的初衷之一，
通过编写简单的客户端程序并登入官方服务端，可以验证 Go-MC 提供的其他功能性模块实现是否与原版游戏保持一致。
继承于 Go-MC 的前身项目 [gomcbot](https://github.com/Tnze/gomcbot)，`bot`包是 Go-MC 最早被开发出来的模块。

`bot` 包提供了一个轻量级的机器人框架，实现了获取服务器状态（Ping and List）、连接服务器登入游戏（Join Game）等功能，
还包含了一个简单的数据包 Handler 注册器，统一地管理从收到的数据包，使得对聊天、区块、窗口等功能的模块化处理成为可能。
