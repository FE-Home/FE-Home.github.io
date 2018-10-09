---
title: 从零开始node（一）：模块机制
date: 2018-10-09 19:18:10
categories:
  - node
---

# 从零开始 node（一）：模块机制

<blockquote class="blockquote-center">朱耀华</blockquote>

之前粗略的阅读过《nodejs 权威指南》，但是这本书有点类似于字典或者文档，比较重视核心模块和接口的使用。

这次从头开始阅读[《深入浅出 Node.js》](https://www.amazon.cn/dp/B00GOM5IL4)，也许这本书比较久远了，但是比较重视原理层面，适合仔细阅读。

<!--more-->

node 中引入模块，需要经历三个步骤

1. 路径分析
2. 文件定位
3. 编译执行

在 Node 中，模块分为两类：`核心模块`和`文件模块`。

核心模块是 node 提供的模块，在 node 源代码编译的过程中，编译进了二进制执行文件。在 node 进程启动时，部分核心模块就被直接加在今内存中，所以这部分核心模块引入时，文件定位和编译执行这两个步骤可以省略掉。核心模块的加载速度是最快的。

文件模块是用户编写的模块在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程。

require()方法对相同模块的二次加载采用缓存优先的方式。

## 路径分析

require()的参数——模块标识符分为以下几类

- 核心模块，如 http、fs
- 相对路径/绝对路径的文件模块
- 自定义的文件模块

自定义的文件模块查找策略是：

- 当前文件目录下的 node_modules
- 父目录下的 node_modules
- 逐级递归，知道根目录下的 node_modules

## 文件定位

Node 会按照.js .json .node 的顺序给模块标识符添加扩展名，在文件定位的过程中是同步阻塞的，所以为标识符带上扩展名会加快速度。

有时候文件定位后得到一个目录，这时首先查找 package.json，从中取出 main 属性指定文件名定位。以上步骤失败则会把 index.js index.json index.node 作为默认文件名。

## 模块编译

对于不同的文件扩展名，其载入方法也不同。

- js 文件：通过 fs 模块同步读取文件后编译执行
- node 文件：这是用 C/C++编写扩展文件，用 dlopen()方法加载最后编译生成的文件
- json 文件：fs 同步读取后，JSON.parse()解析结果

编译成功的模块会将其文件路径作为索引缓存在 Module.\_cache 上。

### JavaScript 模块的编译

在编译的过程中，Node 对 JavaScript 文件内容进行了头尾包装，一个正常的 JavaScript 文件包装成如下：

```javascript
;(function(exports, require, module, __filename, __dirname) {
  var math = require("math")
  exports.area = function(radius) {
    return Math.PI * radius * radius
  }
})
```

CommonJS 规范中存在着 require、exports、module、\_\_filename、\_\_dirname 这几个变量存在，就是由此而来，同事还对每个模块文件之间进行了作用域隔离。

\*关于 exports 和 module.exports，exports 是对 module.exports 的引用，所以直接对 exports 赋值会造成指针丢失，并不会导出模块。

### C/C++模块的编译

node 文件是 C/C++编写编译后生成的，所以这一部分只有加载和执行，执行效率较高。

dlopen()方法在 windows 和\*nix 平台有不同的实现，通过 libuv 兼容层进行封装。

### JSON 文件的编译

读取到内容之后，调用 JSON.parse()得到对象，将其赋给 exports。

内建模块：使用 C/C++编写，主要供其他模块使用的模块，比如 fs 模块。JavaScript 核心模块主要扮演的职责有两类：一类是作为 C/C++内建模块的封装层和桥接层，
供文件模块调用；一类是纯粹的功能模块，它不需要跟底层打交道，但是又十分重要。

扩展模块：用户使用 C/C++编写的模块

这两种模块均属于核心模块。

\*有关内建模块和扩展模块的编译和加载以后再讨论……
