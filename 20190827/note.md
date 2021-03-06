## node

### EventLoop
> 主线程从任务队列中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop(事件循环)

1. V8引擎解析JavaScript脚本。
2. 解析后的代码，调用Node API。
3. libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。
4. V8引擎再将结果返回给用户

### 同步/异步 & 阻塞/非阻塞

#### 同步与异步
同步和异步关注的是消息通知机制
- 同步就是发出调用后，没有得到结果之前，该调用不返回，一旦调用返回，就得到返回值了。 简而言之就是调用者主动等待这个调用的结果
- 而异步则相反，调用者在发出调用后这个调用就直接返回了，所以没有返回结果。换句话说当一个异步过程调用发出后，调用者不会立刻得到结果，而是调用发出后，被调用者通过状态、通知或回调函数处理这个调用。

#### 阻塞与非阻塞
阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态.
- 阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- 非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

#### 组合
同步异步取决于被调用者，阻塞非阻塞取决于调用者
- 同步阻塞
- 异步阻塞
- 同步非阻塞
- 异步非阻塞

### 控制台
在Node.js中，使用console对象代表控制台(在操作系统中表现为一个操作系统指定的字符界面，比如 Window中的命令提示窗口)。
- console.log
- console.info
- console.error 重定向到文件
- console.warn
- console.dir
- console.time
- console.timeEnd
- console.trace
- console.assert

### 全局作用域
- 全局作用域(global)可以定义一些不需要通过任何模块的加载即可使用的变量、函数或类
- 定义全局变量时变量会成为global的属性。
- 永远不要不使用var关键字定义变量，以免污染全局作用域
- setTimeout clearTimeout
- setInterval clearInterval
- unref和ref

### 函数
- require
- 模块加载过程
- require.resolve
- 模板缓存(require.cache)
- require.main
- 模块导出

### process

#### process对象
> 在node.js里，process 对象代表node.js应用程序，可以获取应用程序的用户，运行环境等各种信息

#### process.nextTick & setImmediate
- process.nextTick()方法将 callback 添加到"next tick 队列"。 一旦当前事件轮询队列的任务全部完成，在next tick队列中的所有callbacks会被依次调用。
- setImmediate预定立即执行的 callback，它是在 I/O 事件的回调之后被触发
