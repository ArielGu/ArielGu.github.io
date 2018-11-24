---
title: 理解Buffer
date: 2018-11-24 16:31:58
tags:
---

## 关于Buffer
1. Buffer 是什么?
	- Buffer 是一个像Array的对象，但其主要用于操作字节。
	- Buffer 是典型的JavaScript 和 C++结合的模块（性能相关由C++实现，非性能由JavaScript实现）；
	- Node 在进程启动时已经加载了Buffer 模块
	
	```
	var str='Buffer 结构';
	var buf = Buffer.alloc(20, str);
	```
	
2. Buffer 的应用场景有哪些？
	- 流 的暂存区
	
		流是数据的集合（与数据、字符串类似），但是流的数据不能一次性获取到，数据也不会全部load	到内存中，因此流非常适合大数据处理以及断断续续返回chunk的外部源。流的生产者与消费者之间的速	度通常是不一致的，因此需要buffer来暂存一些数据。buffer大小通过highWaterMark参数指定，默	认情况下是16Kb。

	- 存储需要占用大量内存的数据

		Buffer 对象占用的内存空间是不计算在 Node.js 进程内存空间限制上的，所以可以用来存储大	对象，但是对象的大小还是有限制的。一般情况下32位系统大约是1G，64位系统大约是2G。
		
3. Buffer 的使用？

	- buffer转字符串
	
	```
	buffer.toString();
	Buffer.isEncoding('utf-8'); //编码检查
	Buffer.isBuffer(new Buffer('a'); //Buffer检查
	```
	
	- buffer转JSON

	```
	const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
	console.log(buf.toJSON());    // { type: 'Buffer', data: [ 1, 2, 3, 4, 5 ] }
	```
	
	- buffer 拼接

	```
	var fs = require('fs');
	
	var rs = fs.createReadStream('test.md',{highWaterMark:11});
	var data = '';
	rs.on('data',function(chunk){
		data += chunk;
	});
	rs.on('end',function(chunk){
		console.log(data);
	});
	```
	
	**1. data事件中获取的chunk对象即Buffer对象**
	
	**2. highWaterMark 可将文件可读流每次读取的Buffer长度限制为11**

	
4. Buffer 与性能

	Buffer 在文件I/O和网络I/O中运用广泛，在网络传输中需要转换为 Buffer，以二进制数据传输，可大大提高传输效率。
	
	在实际应用中response 本身就是可写流，所以可将Buffer对象直接传输，例如
	
	```
		var helloworldStr = 'abc';
		var helloworld = new Buffer(helloworldStr);
		http.createServer(function(req,res){
			res.writeHead(200);
			res.end(helloworld);
		});
	```