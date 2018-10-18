---
title: 从零开始 node（三）：多进程
date: 2018-10-17 19:37:58
categories:
  - node
---

# 从零开始 node（三）：多进程和集群

<blockquote class="blockquote-center">朱耀华</blockquote>

在我们日常开发中，对于写好的 node 项目，使用`node app.js`即可启动。但是在大型项目，并发要求较高的情况下，单一的 node 进程不足以应对。

这篇博客从多进程和集群稳定性两个角度来分析 node 项目走向集群的原理。

<!--more-->

## 多进程

Node 提供了 child_process 模块来创建子进程，通过调用 child_process.fork()来实现进程的复制。这种模式叫做`Master-Worker模式`，也叫作主从模式。主进程负责调度，工作进程处理业务。

这里要普及一下`进程`和`线程`的区别，进程占有资源（内存，总线，缓存，管道等），是操作系统进行资源调度的基本单位，是系统中一个互相较为独立的工作空间；线程占有的是 CPU 时间，在 CPU 调度中线程比较常见，线程不占有资源，存在于进程中。

也就是说两个线程互相通信需要额外的操作，实现`进程间通信（IPC, Inter-Process Communication）`的方法有很多，比如：`命名管道`，`匿名管道`，`socket`，`信号量`，`共享内存`，`消息队列`，`Domain Socket`等，Node 采用的是 IPC 管道，通过 libuv 抽象，windows 底层采用命名管道，\*nix 系统采用 Unix Domain Socket 实现。在应用层面表现为 message s 事件和 send()方法。

来看一段创建多进程的例子：

```javascript
// parent.js
var cp = require("child_process")
var child1 = cp.fork("child.js")
var child2 = cp.fork("child.js")

var server = require("net").createServer()
server.listen(80, function() {
  child1.send("server", server)
  child2.send("server", server)
  server.close()
})
```

```javascript
// child.js
var http = require("http")
var server = http.createServer(function(req, res) {
  res.writeHead(200, { "Content-Type": "text/plain" })
  res.end("handled by child, pid is " + process.pid + "\n")
})
process.on("message", function(m, tcp) {
  if (m === "server") {
    tcp.on("connection", function(socket) {
      server.emit("connection", socket)
    })
  }
})
```

在主从模式中，数据可以由主进程获取然后转发到各个进程中，但是这种方式并不高效。另一种方法是各个进程分别监听该端口，以抢占式处理请求。

在日常的开发过程中经常出现端口被占用的情况，所以在我们日常的认知中是一个端口只能由一个进程监听。事实上，多个进程是可以监听同一个端口的，只是由于我们分别启动进程监听端口时，socket 套接字的文件描述符各不相同，所以产生了错误。

> Node 底层对每个端口监听都设置了 SO_REUSEADDR 选项，这个选项的涵义是不同进程可以就相同的网卡和端口进行监听，这个服务器端套接字可以被不同的进程复用。由于独立启动的进程互相之间并不知道文件描述符，所以监听相同端口时就会失败。但对于 send()发送的句柄还原出来的服务而言，它们的文件描述符是相同的，所以监听相同端口不会引起异常。

## 集群

本文从三个角度来简单介绍一下 node 集群管理需要注意的问题：

- 负载均衡
- 平滑重启
- 状态共享

### 负载均衡

由于这些进程对于连接是抢占式的，也就是说，当连接进入端口时，所有进程都会被唤起，最终却只有一个进程能够获得连接，其他进程再次进入休眠。这种现象叫做`惊群`。

这时就需要主进程来决定连接的分配问题。

使用 pm2 和 Nginx 可以方便的进行负载均衡，如：

```sh
$ pm2 start app.js -i 4
```

来启动 4 个 app.js 进程。下一篇会有详细介绍

为此 Node 采用的策略叫`Round-Robin`，又叫轮叫调度。轮叫调度的工作方式是由主进程接受连接，将其依次分发给工作
进程。分发的策略是在 N 个工作进程中，每次选择第 `i = (i + 1) mod n` 个进程来发送连接。在 cluster
模块中启用它的方式如下：

```javascript
// 启用Round-Robin
cluster.schedulingPolicy = cluster.SCHED_RR
// 不启用Round-Robin
cluster.schedulingPolicy = cluster.SCHED_NONE
```

或者在环境变量中设置 `NODE_CLUSTER_SCHED_POLICY` 的值，如下所示：

```javascript
export NODE_CLUSTER_SCHED_POLICY=rr
export NODE_CLUSTER_SCHED_POLICY=none
```

大致类似于 CPU 调度的时间片轮转法。

### 平滑重启

使用 pm2 的 reload 命令可以平滑的重启各个进程，也就是可以保证最少会有一个进程在工作中，而不是全部结束后重启。

_自杀信号_

在进程得知将要停止时，先向主进程发出“suicide”信号，主进程收到信号后会创建新工作进程来代替该进程，随后原进程处理完毕后即关闭。

_限量重启_

如果工作进程由于某些原因不断重启，可能会造成资源枯竭，这种频繁的重启有可能是由我们程序编写的原因造成的。如果在单位时间内进程重启超过一定次数，就会发出"giveup"事件,并放弃重启该工作进程。

### 状态共享

解决数据共享最直接、简单的方式就是通过第三方来进行数据存储，比如将数据存放到数据库、磁盘文件、缓存服务（如 Redis）中，所有工作进程启动时将其读取进内存中。

实现状态同步的一种方案是各个进程通过轮询的方式来获取数据更新，这种方式显然不合适，占用了大量的资源。  
另一种方案是数据发生更改时主动通知各个进程，我们可以独立出来一个进程，通过轮询方式来判断状态的变化，一旦变化就主动通知进程变化。
