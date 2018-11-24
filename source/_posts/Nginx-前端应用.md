---
title: Nginx 前端应用
date: 2018-10-20 14:31:58
tags:
---
##前言：
	用Nginx给网站做反向代理和负载均衡，是广泛使用的一种Web技术，不仅能够保证后端服务器的隐蔽性，还可以提高网站情况，部署灵活，而且以来
	源软件实现负载均衡性价比非常高

- 配置文件：

/usr/local/etc/nginx/nginx.conf
open -a /Applications/Sublime\ Text.app nginx.conf

- 安装文件

/usr/local/Cellar/nginx

http://blog.fens.me/nodejs-websocket-nginx/

##Socket 应用

原理解释：WebSocket协议相比较于HTTP协议成功握手后可以多次进行通讯，直到连接被关闭。WebSocket 和 HTTP 的握手兼容，它使用 HTTP 中的Upgrade 协议头将连接从 HTTP 升级到 WebSocket。

一般WebSocket 服务程序使用 ws 协议（是一种明文），可以通过Nginx 在客户端和服务端直接做一层转发，客户端用 wws 访问，然后nginx 和服务端通过 ws 通信。

```
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $host;
```

相当于在HTTP 的请求中多了如下头部：

```
# 状态码为101
HTTP/1.1 101 Switching Protocols
Upgrade: websocket  // 多出头部
Connection: upgrade // 多出头部
```