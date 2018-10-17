---
title: 从零开始 node（三）：多进程
date: 2018-10-17 19:37:58
categories:
  - node
---

# 从零开始 node（三）：多进程和集群

在我们日常开发中，对于写好的 node 项目，使用`node app.js`即可启动。但是在大型项目，并发要求较高的情况下，单一的 node 进程不足以应对。

这篇博客从多进程和集群稳定性两个角度来分析 node 项目走向集群的原理。

## 多进程

Node 提供了 child_process 模块来创建子进程，通过调用 child_process.fork()来实现进程的复制。这种模式叫做`Master-Worker模式`，也叫作主从模式。主进程负责调度，工作进程处理业务。

这里要普及一下`进程`和`线程`的区别，进程占有资源（内存，总线，缓存，管道等），是操作系统进行资源调度的基本单位，是系统中一个互相较为独立的工作空间；线程占有的是 CPU 时间，在 CPU 调度中线程比较常见，线程不占有资源，存在于进程中。

也就是说两个线程互相通信需要额外的操作，实现`进程间通信（IPC, Inter-Process Communication）`的方法有很多，比如：`命名管道`，`匿名管道`，`socket`，`信号量`，`共享内存`，`消息队列`，`Domain Socket`等，Node 采用的是 IPC 管道，通过 libuv 抽象，windows 底层采用命名管道，\*nix 系统采用 Unix Domain Socket 实现。在应用层面表现为 message s事件和 send()方法。

来看一段创建多进程的例子：

```javascript
// parent.js
var cp = require('child_process')
var child1 = cp.fork('child.js')
var child2 = cp.fork('child.js')

var server = require('net').createServer()
server.listen(80, function () {
  child1.send('server',server)
  child2.send('server',server)
  server.close()
})
```
