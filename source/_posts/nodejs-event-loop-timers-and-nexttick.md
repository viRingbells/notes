---
title: Nodejs的事件循环
categories:
  - 工程
tags:
  - 学习
date: 2018-06-04 16:44:33
---


最近想深入了解一下Nodejs的event loop，发现了官网上的[一篇文章][event-loop-timers-and-nexttick]。在Nodejs的[中文官网][nodejs-cn]上也没有收录。考虑日后也要常来翻阅，英语阅读起来确实也不直观，所以尝试翻译一下。（原汁原味是不可能的，肯定会有自己的理解参杂在里面）

> 正文来自Nodejs官网[https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/]

------分割线------

## 什么是事件循环(Event Loop)

事件循环通过下放IO任务到操作系统层面，使单线程的Nodejs具备执行非阻塞IO任务的能力。

绝大多数现代操作系统都支持多进程，它们可以在后台并行执行多个任务。当一个任务运行结束后，操作系统会通知Nodejs。Nodejs然后在**轮询**队列中添加对应的回调，并最终执行回调。这部分细节后面再详细说明。

## 事件循环流程

当Nodejs启动时，它就会初始化事件循环，执行目标脚本（或者进入REPL模式，这个不在本文讨论范围之内）。在目标脚本内可能会有调用异步API、Timers、或者`process.nextTick()`等行为。然后Nodejs就会进入事件循环。

下图简要的展示了事件循环流程中的各环节。

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

_说明：每一个方框对应到事件循环的一个"环节"_

每一个环节都有其特有的行为和一个先进先出的回调队列。当一个环节开始时，它会执行这个环节特有的行为，并执行回调队列里的所有回调函数，直到数量达到一个上限值。之后，事件循环会进入下一个环节并重复这一过程。

由于所有执行的操作都可能引起_更多_的操作和事件，这些事件又会被内核添加到队列并在**轮询**阶段处理，因此在处理轮询事件的同时，也会有新的轮询事件添加到队列中。于是，长时间运行的回调会使轮询阶段执行得比定时器的阈值长久许多。查阅[定时器](#定时器)和[轮询](#轮询)了解更多细节

_说明：在Windows和Linux/Unix上的实现上有一些对这个流程展示不太重要的微小差异。虽然实际上有七、八个环节，但是值得关心的，也就是Nodejs实际上用到的，只有上面示例的那些。_

## 各环节概览

* **定时器(timers)**: 这个环节执行通过`setTimeout`和`setInterval`设置的回调。
* **待执行的回调(pending callbacks)**: 执行一些系统I/O的回调。
* **空闲、准备(idle, prepare)**: 仅系统内部使用的环节。
* **轮询(poll)**: 获取新的I/O事件；执行I/O回调（除了关闭回调和通过定时器以及`setImmediate`设置的回调以外的绝大多数回调）。Nodejs在必要的时候会在这个环节阻塞。
* **检查(check)**: 执行`setImmediate`回调。
* **关闭回调(close callbacks)**: 一些负责收尾清理资源的回调，比如`socket.on('close', ...)`。

在每一次事件循环之间，Nodejs会检查是否还存在等待中的异步I/O或者定时器，如果没有，则会退出循环结束运行。

## 各环节详解

### 定时器

通过定时器可以设置阈值，回调函数会在时间满足这个阈值后某个时间执行，但不能保证这个时间与阈值精准一致。定时器在事件a满足阈值后，会尽可能早的执行回调，但可能因操作系统的调度和其他的回调而有所延迟。

_说明：技术上来说，定时器的执行时间受[轮询阶段](#轮询)的直接影响_

例如，用定时器设置一个100毫秒后执行的回调，然后又开始异步读取一个文件，读取过程需要95毫秒。

```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // 假设读取文件需要95毫秒
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// 执行一个需要95毫秒的异步操作
someAsyncOperation(() => {
  const startCallback = Date.now();

  // 执行一个需要10毫秒的同步操作
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

当事件循环进入轮询阶段，回调队列为空（`fs.readFile()`还没有执行完毕）。因此系统会等待直到时间满足最近的一个定时器阈值。等到第95毫秒的时候，`fs.readFile()`执行完毕，并且将一个耗时10毫秒的回调添加到轮询阶段队列中并执行。当这个回调执行完毕后，轮询阶段队列再一次为空。这时系统发现时间已经满足最近的定时器阈值了，就返回到定时器阶段执行其回调。所以，在这个示例里，回调真正执行的时间是在设置回调后的105毫秒后。

说明：为了防止事件循环因为轮询阶段而进入[饥饿][wikipedia-starvation]，[**libuv**][libuv]（实现Nodejs事件循环以及所有异步行为的C语言库）也会有一个轮询的固定（由系统决定）上限值。

### 待执行的回调

这个阶段执行一些系统操作的回调，像处理TCP错误之类的。比如一个TCP的socket在尝试连接时收到了`ECONNREFUSED`错误，一些\*inx的操作系统会等待报告这个错误。那么它会被放到队列里，并且在这个阶段来处理。

### 轮询

轮询阶段有两个主要的工作：

1. 计算轮询I/O和阻塞等待的时间
2. 处理轮询回调队列里的事件

当事件循环进入到这个阶段，_并且没有设置定时器_，那么可能会出现下面两种情况：

* _如果**轮询**回调队列**不为空**_，事件循环会遍历整个回调队列，同步的执行这些回调，直到全部执行完毕，或者执行数量达到系统设置的固定上限值为止。
* _如果**轮询**回调队列**为空**_，那么还会有两种情况：
  * 如果设置了`setImmediate()`回调，那么事件循环会退出**轮询**阶段，进入到**检查**阶段并执行这些回调。
  * 如果没有设置`setImmediate()`，那么事件循环会等待新的回调添加到回调队列中，并立刻执行。

一旦**轮询**回调队列为空，事件循环就会检查当前时间_满足哪些定时器的时间阈值_。如果至少有一个定时器已经满足条件，那么事件循环会进入**定时器**阶段去执行那些定时器回调。

### 检查

这个阶段允许用户在轮询阶段结束后立刻执行回调。当**轮询**阶段空闲，并且存在通过`setImmediate()`设置的回调，那么事件循环不再等待，而是进入这个阶段。

`setImmediate`实际上是一个在事件循环单独的阶段执行的定时器。它通过libuv的API，设置在轮询阶段结束后立刻执行的回调。

通常，虽然代码的执行，系统一定会进入**轮询**阶段，然后等待新的请求、连接之类的。但是如果存在通过`setImmediate()`设置的回调并且**轮询**阶段空闲，那么事件循环就不在等待**轮询**事件而是进入**检查**阶段处理这些回调。

### 关闭回调

如果一个socket或者句柄突然关闭（如`socket.destroy()`），那么`error`事件会在这个阶段触发。除此之外这些事件通过`process.nextTick()`来触发。

## `setImmediate()` vs `setTimeout()`

`setImmediate()`和`setTimeout()`很像，但根据它们在不同的阶段调用，会有不同表现。

* `setImmedate()`的设计用途是在**轮询**阶段结束后立刻执行回调。
* `setTimeout()`的设计用途是在满足指定的时间阈值后执行回调。

调用它们的时机会影响对应回调的执行顺序。如果它们都在主模块中调用，那么回调的顺序与脚本执行的速度直接相关（可能受到机器上其他应用的影响）。

例如，我们在非IO循环（比如主模块）中运行下面这个脚本，那么脚本内的两个定时任务执行的顺序，受到脚本运行速度的影响而无法确定。

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```console
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

但是如果我们在IO循环内执行上面那个脚本，那么总是会先执行`setImmediate()`对应的回调。

```javascript
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```console
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

因此相对于`setTimeout()`，`setImmediate()`最大的优势就是，只要是在IO循环内，无论存在多少个定时器，`setImmediate()`的回调总是会最先执行。

## `process.nextTick()`

### 理解`process.nextTick()`

你或许已经注意到`process.nextTick()`作为一个异步API，却并没有出现在上面的流程示意图中。这是因为`process.nextTick()`技术上来说并不是事件循环的一部分。无论事件循环处于哪个阶段，`process.nextTick()`总是会在当前的操作结束后立刻执行。

再回过头看看异步循环的示意图，在某个阶段调用`process.nextTick()`后，所有`process.nextTick()`设置的回调都会在下个阶段开始前执行。这种特性可能会导致一些严重的问题。如果递归的调用`process.nextTick()`，那么就是因为无法进入轮询阶段而导致IO饥饿。

### 为什么允许使用这个API

那么为什么要在Nodejs中开放对这个API的使用呢。原因之一就是在Nodejs的设计理念中，API应该总是异步的，即使没有那个必要。比如：

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback,
                            new TypeError('argument should be string'));
}
```

这段代码作用是检查参数是否正确，如果不正确那么通过回调回传一个错误。`process.nextTick()`这个API最近有一个更新，使它能够接收`callback`以外的参数，并将这些参数传递给`callback`回调函数。这样就不用再给`callback`加一层嵌套。

那么这个代码要做的是，在用户其他代码执行完毕后再向用户返回这个错误。通过`process.nextTick()`，可以确保`apiCall`的回调一定能够在用户的代码执行完毕后，并且在进入事件循环的下一个环节之前执行。为了实现这个功能，系统允许展开JS调用栈，并且直接执行对应的回调。因此用户可以无限嵌套的调用`process.nextTick()`而不会触发`RangeError: Maximum call stack size exceeded from v8`的错误。

Nodejs的这个设计理念，可能会导致一些麻烦的情况。可以看一看下面这个例子：

```javascript
let bar;

// 这个函数有异步函数的特征，却以同步的方式调用回调函数
function someAsyncApiCall(callback) { callback(); }

// 回调函数在someAsyncApiCall结束之前调用了
someAsyncApiCall(() => {
  // someAsyncApiCall结束了，但是bar还没有被赋值
  console.log('bar', bar); // undefined
});

bar = 1;
```

用户定义了一个具有异步特征的函数，但实际上确实同步执行的。当调用`someAsyncApiCall()`时，因为没有任何异步行为，其回调会在同一个事件循环的环节里执行。那么由于这个脚本还没有运行到后面，所以当在回调里读取`bar`时，其值并不存在。

使用`process.nextTick()`后，脚本就能在回调执行前完成初始化变量、函数之类的工作。`process.nextTick()`同时也有防止事件循环进入下一个环节的能力。有时候我们需要在进入事件循环的下一个环节之前就把当前环节出现的错误通知给用户。

这是上面使用了`process.nextTick()`的例子：

```javascript
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

这是实际工作中的一个例子：

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

当只传端口参数时，系统会立刻绑定端口号，并且立刻执行`listen()`回调。但是此时`.on('listening')`还没有执行，其回调也不会在绑定端口号后触发执行。

为了绕过这个问题，`listen()`使用`nextTick()`将事件放到队列里，等脚本同步执行完毕后才会触发事件回调。这样，用户就可以按上面的方式设置事件回调函数了。

## `process.nextTick()` vs `setImmediate()`

那么对于用户来说，就有了两个功能类似但是名字容易让人混淆的API了。

* `process.nextTick()`: 在当前事件循环环节立即触发回调。
* `setImmediate()`: 在当前事件循环环节后面的环节触发回调。

本质上来说，这两个API互换名称才更符合他们的表现，`process.nextTick()`比`setImmediate()`要『更加立即』的执行回调。但是由于历史原因也不太可能更改这两个API的名称，将它们对调可能会引起npm上的库大面积失效。每天都会有新的包加入到npm中，也就是说每过一天，互换这两个API名称带来的潜在危险也就越大。

_我们建议用户使用`setImmediate()`，因为这个API名称释义性更强。（并且它的兼容性也更强，比如兼容浏览器端的js）_

## 为什么使用`process.nextTick()`?

有以下两个主要的原因。

1. 允许用户处理错误、清理后续不再使用的资源，以及可能的在事件循环进入下一环节之前发起一个新的请求。
2. 有时候需要在栈展开结束后，下一个事件循环环节之前执行回调。 

如下就是一个满足用户预期的例子：

```javascript
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```

假设`listen()`在事件循环之初执行，而对应的回调通过`setImmediate()`设置。绑定端口会立刻执行，除非参数中有hostname。那么对于事件循环来说，它必须进入**轮询**环节之后，才会执行`listen`回调。在**轮询**环节，那么就可能收到一个连接，并在`listen`事件之前触发`connection`事件。

另外一种情况就是，在继承了`EventEmitter`的类的构造函数中触发事件。

```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

由于用户还没有给对象设置事件的回调，在构造函数中触发事件不会触发任何回调。因此，可以通过`process.nextTick()`赋予在构造函数中触发回调的能力。

```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // 使用nextTick来触发事件
  process.nextTick(() => {
    this.emit('event');
  });
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```


[event-loop-timers-and-nexttick]:https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/
[libuv]:http://libuv.org
[nodejs-cn]:http://nodejs.cn
[wikipedia-starvation]:https://en.wikipedia.org/wiki/Starvation_(computer_science)
