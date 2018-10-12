---
title: 从零开始 node（二）：Event Loop
date: 2018-10-11 17:37:19
categories:
  - node
---

# 从零开始 node（二）：Event Loop

<blockquote class="blockquote-center">朱耀华</blockquote>

第三、四章讲的是异步 I/O、异步编程，而实现异步的工具就是 Event Loop。在这里可以将浏览器环境和 Node 环境下的 Event Loop 对比来学习。


<!--more-->

首先看一个栗子：

```javascript
setTimeout(() => {
  console.log("timer1")

  Promise.resolve().then(function() {
    console.log("promise1")
  })
}, 0)

setTimeout(() => {
  console.log("timer2")

  Promise.resolve().then(function() {
    console.log("promise2")
  })
}, 0)

// 浏览器环境
// VM82:2 timer1
// VM82:5 promise1
// VM82:10 timer2
// VM82:13 promise2

// Node环境 v8.5.0
// tick2.js:2  timer1
// tick2.js:10 timer2
// tick2.js:5  promise1
// tick2.js:13  promise2
```

产生原因就在于两种环境下的 Event Loop 策略和过程不同。

## 浏览器环境下的 Event Loop

浏览器环境下的异步操作分为两种：宏任务（macrotask）和微任务（microtask）。

宏任务包括 script ， setTimeout ，setInterval ，setImmediate ，I/O ，UI rendering

微任务包括 process.nextTick ，promise ，Object.observe ，MutationObserver

重点来了：  
在每一次 Event Loop 中，从宏任务队列中取出一个任务执行，该任务执行的过程中微任务的回调函数加入到微任务队列中，当宏任务执行结束，开始执行微任务，直到队列清空。再取出下一个宏任务重复以上操作。

举个栗子：

```javascript
setTimeout(function() {
  console.log(1)
}, 0)
new Promise(resolve => {
  console.log(2)
  for (var i = 0; i < 1000; i++) {
    i == 999 && resolve()
  }
  console.log(3)
}).then(function() {
  console.log(4)
})
console.log(5)

// 2 3 5 4 1
```

看懂这个你才算真正理解了浏览器端的 Event Loop。

流程分析：

1. 首先从宏任务队列中取出一个任务即 script，顺序执行。
2. setTimeOut 立即执行（注意宏任务微任务区分的是回调），将回调函数加入宏任务队尾，继续执行。
3. new Promise 立即执行构造函数，输出 2，3，在循环 1000 次之后执行了 resolve，将 then 的回调函数加入微任务队尾。
4. 输出 5，该宏任务执行完成，开始执行微任务队列，取出队首 then 的回调函数执行，输出 4。
5. 微任务队列清空，取出宏任务队首的 setTimeOut 回调，输出 1。

这时我们来重新分析一下开始的栗子：

1. setTimeOut1，2 分别加入宏任务队列。
2. script 执行结束，微任务队列为空，取出宏任务队列 setTimeOut1 的回调，输出 timer1，promise 加入微任务队列。
3. 取出微任务队列队首，输出 promise1。
4. 取出 setTimeOut2 的回调，输出 timer2，promise 加入微任务队列。
5. 取出微任务队列队首，输出 promise2。

至此，浏览器端的 Event Loop 分析完成

## Node 环境下的 Event Loop

分析 Node.js [libuv 库源码](https://github.com/libuv/libuv/blob/v1.x/src/unix/core.c#L348-L397):

```c
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
  int timeout;
  int r;
  int ran_pending;

  r = uv__loop_alive(loop);
  if (!r)
    uv__update_time(loop);

  while (r != 0 && loop->stop_flag == 0) {
    uv__update_time(loop);
    // timers阶段
    uv__run_timers(loop);
    // I/O callbacks阶段
    ran_pending = uv__run_pending(loop);
    // idle阶段
    uv__run_idle(loop);
    // prepare阶段
    uv__run_prepare(loop);

    timeout = 0;
    if ((mode == UV_RUN_ONCE && !ran_pending) || mode == UV_RUN_DEFAULT)
      timeout = uv_backend_timeout(loop);
    // poll阶段
    uv__io_poll(loop, timeout);
    // check阶段
    uv__run_check(loop);
    // close callbacks阶段
    uv__run_closing_handles(loop);

    if (mode == UV_RUN_ONCE) {
      uv__update_time(loop);
      uv__run_timers(loop);
    }

    r = uv__loop_alive(loop);
    if (mode == UV_RUN_ONCE || mode == UV_RUN_NOWAIT)
      break;
  }

  if (loop->stop_flag != 0)
    loop->stop_flag = 0;

  return r;
}
```

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<──connections───     │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

- timers 阶段：执行 timer（setTimeout、setInterval）的回调
- I/O callbacks 阶段：执行一些系统调用错误，比如网络通信的错误回调
- idle, prepare 阶段：仅 node 内部使用
- poll 阶段：获取新的 I/O 事件, 适当的条件下 node 将阻塞在这里
- check 阶段：执行 setImmediate() 的回调
- close callbacks 阶段：执行 socket 的 close 事件回调

可以这样理解：  
在 node 中每一个阶段都维护一个宏任务和微任务队列。在宏任务执行结束之后，执行一次微任务队列中的任务。在等待 I/O 的时候，node 线程会阻塞在 poll 阶段。

### timers 阶段

timers 阶段 Node 会检查 timer 是否已过期，如果有就把它的回调压入任务队列中执行。但是 js 的定时器并不靠谱，因为在定时器过期的时候，js 的线程可能会在执行另外一个任务。

setTimeOut 和 setImmediate 的执行顺序也是不固定的。如下：

```javascript
setTimeout(() => {
  console.log("timeout")
}, 0)

setImmediate(() => {
  console.log("immediate")
})
```

看起来任务是从 timers 阶段开始的，但是如果我们从 poll 阶段的 I/O 开始执行这段代码，就是 setImmediate 先执行，因为 poll 阶段之后就是 check 阶段，执行 setImmediate 代码。

### poll 阶段

当有已超时的 timer，执行它的回调函数。同步执行 poll 队列里的回调，直到队列为空或执行的回调达到系统上限，而后检查有无预设的 setImmediate()，分两种情况：

- 有预设的 setImmediate(),poll 阶段进入 check 阶段，并执行 check 阶段的任务队列；
- 无预设的 setImmediate()，阻塞在该阶段等待。如果 timer 队列非空，则开始下一轮事件循环，重新进入到 timer 阶段。

### check 阶段

setImmediate()的回调加入到 check 队列中顺序执行。

这时候文章开始的例子就比较容易理解了。我们对照着来看其他几个类似案例：

```javascript
// 加入两个setImmediate()的回调函数
setImmediate(function() {
  console.log("setImmediate延迟执行1")
  // 进入下次循环
  Promise.resolve().then(function() {
    console.log("promise1")
  })
  process.nextTick(function() {
    console.log("nextTick1")
  })
  process.nextTick(function() {
    console.log("nextTick2")
  })
}, 0)

setImmediate(function() {
  console.log("setImmediate延迟执行2")
}, 0)

console.log("正常执行")

//正常执行
// setImmediate延迟执行1
// setImmediate延迟执行2
// nextTick1
// nextTick2
// promise1
```

两个 setImmediate 先执行，且`nextTick`队列优先清空，然后再执行其他队列。

---

至此，js 执行的两个环境的异步 Event Loop 执行顺序已经介绍清楚。大家可以自己动手来试试执行一下。

---

## 参考资料

1. [深入浅出 Node.js](https://www.amazon.cn/dp/B00GOM5IL4)
2. [深入理解 js 事件循环机制（Node.js 篇）](http://lynnelv.github.io/js-event-loop-nodejs)
3. [知乎专栏](https://zhuanlan.zhihu.com/p/33090541)
4. [Node-libuv 源码](https://github.com/libuv/libuv/blob/v1.x/src/unix/core.c#L348-L397)
5. [掘金技术征文](https://juejin.im/post/5ba34e54e51d450e5162789b#heading-21)
