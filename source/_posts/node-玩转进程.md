---
title: Node-玩转进程
date: 2018-12-16 20:31:14
tags:
---
## 背景
Node 在实际开发中面临的两个问题：

1. 如何充分利用多核CPU服务器？
2. 如何保证进程的健壮性和稳定性？ - Node 执行在单线程上，一旦线程异常未被捕获，会引起整个进程崩溃。

## 解决方案演进
####多进程架构

每个进程使用一个CPU，以此实现多核CPU利用。 Node提供了child_process 模块，以及child_process.fork()函数供实现进程的复制。实例： Demo01

work.js

```
const http = require('http');
http.createServer((req,res)=>{
  res.writeHead(200,{'Content-Type':'text/plain'});
  res.end('Hello World\n');
}).listen(Math.round(1+Math.random())*1000,'127.0.0.1');

```

Master.js

```
const fork = require('child_process').fork;

const cpus = require('os').cpus();
for(var i =0;i<cpus.length;i++){
  fork('./worker.js');
}
```

1. 如何创建子进程
	
	child_process 提供了四种方法：
	- spawn() 启动一个子进程来执行命令
	- exec() 启动一个子进程来执行命令。 与spawn 不同点：有回调函数获知子进程的状况
	- execFile() 启动一个子进程来执行可执行文件。
	- fork() 无spawn 不同点，只需指定执行的 Js 文件模块即可。
	
	```
	const cp = require('child_process');

	cp.spawn('node',['worker.js']);
	cp.exec('node worker.js',(err,stdout,stderr)=>{});
	cp.execFile('worker.js',(err,stdout,stderr)=>{});
	cp.fork('./work.js');
	```
	
2. 进程间通信
	
	JavaScript 主线程和UI 渲染线程共用一个线程，相互阻塞。为了解决这个问题HTML5 提出了WebWorker API：允许创建工作线程在后台运行，不影响主线程和UI 渲染，通过onmessage() 和 postMessage() 进行通信。Node 中对应的实现如实例：Demo02
	
	```
	process.on('message',(m)=>{});
	process.send({foo:'bar'});
	```
	
	父子进程之间会创建IPC(Inner-Process Communication, 即进程间通信），其目的是让不同进程能够互相访问资源并进行协调工作（有多种方式：管道、socket、信号量、共享内存、消息队列等），Node采用了管道技术
	
	
3. 消息传递内容——句柄传递

	IPC 出来传递数据，还能传递句柄
			
	**1）What?是什么？**
	
	一种可以用来标识资源的引用，其内部可以包含指向对象的文件描述符。例如句柄可以用来标识一个服务器端socket 对象、一个客户端socket 对象、一个UDP套接字、一个管道等等
	
	**2) How?怎么使用？**
	
	```
	// parent.js
	const child = cp.fork('child.js');
	const server = require('net').createServer();
	server.on('connection',(socket)=>{
  		socket.end('handled by parent\n');
	});

	server.listen(1337,()=>{
  		child.send('server',server);
	})
```

	```
	// 句柄 child.js
	process.on('message',(m,server)=>{
  		if(m==='server'){
    		server.on('connection',(socket)=>{
      			socket.end('handled by child, pid is '+process.pid);
    		})
  		}
	})
	```
	
	现象：客户端发起请求可能被父进程或者子进程处理；
	处理方法：主进程将句柄发送给子进程后，关闭服务器的监听
	### 集群之路

父子进程之间的事件：error / exit / close / disconnect / kill 等，这些事件关联着父子进程，所以就需要在这些关系之间建立机制，搭建完整的多进程架构。
	
1. 自动重启

	子进程关闭，重新启动一个工作进程来继续服务
 * Condition1: 业务代码中有隐藏的bug导致工作进程退出，所以需要在工作进程中处理这种异常
 * Condition2: 所有工作进程都停止接收新的连接，全处于等待退出的状态。但等到进程完全退出才重启过程中，没有进程接收用户服务
 * Condition3: 等待长连接需要较久的时间，所以需要为连接的断开设置一个超时时间，限定之间里强制退出
 * Condition4: 优化日志记录
 * Condition5: 无限重启：如果启动的过程中发生错误，导致工作进程频繁重启，所以采用队列标记，判断是否重启过于频繁

2. 负载均衡

	Node而言繁忙由CPU 和 I/O 两个部分构成，而影响抢占是CPU的繁忙度，所以可能出现I/O繁忙但是CPU空闲，造成进程间负载不均衡。Node在v0.11 中提供Round-in 轮叫调度的策略

