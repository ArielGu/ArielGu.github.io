---
title: 网络编程
date: 2018-09-23 19:59:17
---
### WebSocket

优势：

- WebSocket客户端基于事件的编程模式和Node中的自定义事件类似
- 实现了客户端和服务器之间的长连接，擅长处理大量高并发的客户端连接处理
- 相较于HTTP只建立了一个TCP（协议轻量），实现双向传播

实践方案：
 Node端：socket-io
 Client端：socket.io-client
 
 注意：1）挂载在HTTP Server 上
 	   2）监听WebSocket 事件
