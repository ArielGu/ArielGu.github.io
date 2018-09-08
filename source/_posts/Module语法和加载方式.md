---
title: Module语法和加载方式
date: 2018-09-02 23:18:56
tags:
---

# 背景

Javascript 没有模块的(module)体系，ES6前主要是CommonJs(用于服务器端)、AMD(用于浏览器)

ES6 的加载方式：编译时加载/静态加载（即在编译时就完成模块加载），效率自然是比CommonJs（运行时加载）高。

# ES6
1.import命令具有提升效果；

2.由于import是静态执行，所以不能使用变量和表达式（这种智能在运行时得到结果的语法结构）

```
// 报错
import {'f'+'oo'} from 'my_module;
```

3.import 语句会执行所加载的模块，但不会输入任何值，并且不会加载多次

```
import('../a.js),import()函数功能类似于Node的require 方法，区别在于：前者是异步加载，后者是同步加载
```

# Module 的加载方式
1.浏览器加载js文件会阻塞线程，所以允许异步加载的方式

```
<script src='./myModule.js' defer></script>
<script src='./myModule.js' async></script>
```
 
**区别：**defer 会等到整个页面正常渲染完成后执行；而async 一旦下载完成，渲染引擎终止，执行脚本后继续渲染。

加载es6 模块，也可用\<script\>标签，但是要加type="module"属性。

```
<script src='./myModule.js' type="module"></script>
<!-- 等同于 -->
<script src='./myModule.js' defer></script>

<script src='./myModule.js' type="module" async></script>
<!-- 等同于 -->
<script src='./myModule.js' async></script>
```

2.ES6 模块和CommonJs模块的差异

- CommonJs 模块输出的是值的复制；ES6模块输出的是值的引用

```
<!-- CommonJS lib.js -->
var counter = 3;
function inCounter(){
	counter++;
}
module.exports={
	counter: counter,
	inCounter: inCounter
}

<!-- CommonJS main.js -->
var mod = require('./lib');
mod.counter // 3
mode.inCouter();
mod.counter //3

<!-- ES6 lib.js -->
export let counter = 3;
export function inCounter(){
	counter++;
}

<!-- ES6 main.js -->
import {counter, inCounter} from './lib';
counter //3
inCounter();
counter //4
```

ES6 模块的运行机制和CommonJs不同。ES6模块中JS引擎对脚本静态分析是，遇到模块加载命令（import）就会生成一个只读引用，等脚本真正运行了才会根据引用到被加载的模块中取值。
**所以ES6模块是动态引用，没有缓存**

注：地址是只读的，不能重新赋值

- CommonJS 模块是运行时加载，ES6模块是编译时输出接口

# Node 加载
1. Node 有自己的CommonJS 模块格式，与ES6 模块格式不兼容，目前的解决方案是将两者分开，采用各自的加载方案。静态分析阶段，脚本只要有一行import 或 export 语句，Node就会认为是ES6模块；

## CommonJs模块和ES6模块互相加载
- import 命令加载CommonJS 模块
步骤一： module.exports 等同于 export.default

```
// a.js
module.exports={
	foo:'hello',
	bar:'world'
}

<!-- 等同于 -->
export.default={
	foo:'hello',
	bar:'world'
}

//引用
<!-- 写法一 -->
import baz from './a'
<!-- 写法二 -->
import {default as baz} from './a'
// 上面两种写法
//baz={foo:'hello',bar:'world'}
<!-- 写法三 -->
import * as baz from './a'
// baz={
//	get default(){return module.exports;}
//	get foo(){ return this.default.foo }.bind(baz);
//	get bar(){ return this.default.bar }.bind(baz);
//}
```

步骤二：ES6模块是在编译时确定输出接口，CommonJS模块是在运行时才确定输出接口，所以一下写法❌

```
import {readfile} from 'fs';
```
- require 命令加载ES6
ES6 模块的所有输出接口都会成为输出对象的属性

```
// es.js
let foo = {bar:'my-default'};
export default foo;
foo=null;

// commonJs.js
const es_namespace = require('./es');
es_namespace.deault // {bar:'my-default'}
```

注：es.js 对foo重新赋值没有在模块外反应出来，原因就是CommonJS加载的缓存

---
## 实际的应用
1.单页按需加载

```
const listRoutes = [
    {
        path: 'list/:pageName',
        getComponent: (location, cb) => {
            require.ensure([], (require) => {
                cb(null, require('components/List/ListContainer').default);
            }, 'list');
        },
        onEnter: ({ params }) => {
            pageEnter({ ...params, pageType: 'list' });
        },
    },
];
```

2.Koa 项目中模块引用


