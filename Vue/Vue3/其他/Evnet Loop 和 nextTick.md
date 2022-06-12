## Evnet Loop 和 nextTick

### JS 执行机制

都知道js 是单线程的如果是多线程的话会引发一个问题在同一时间同时操作DOM 一个增加一个删除JS就不知道到底要干嘛了，所以这个语言是单线程的但是随着HTML5到来js也支持了多线程webWorker 但是也是不允许操作DOM

单线程就意味着所有的任务都需要排队，后面的任务需要等前面的任务执行完才能执行，如果前面的任务耗时过长，后面的任务就需要一直等，一些从用户角度上不需要等待的任务就会一直等待，这个从体验角度上来讲是不可接受的，所以JS中就出现了异步的概念。

### 同步任务

代码从上到下按顺序执行

### 异步任务

#### 1.宏任务

`script`(整体代码)、`setTimeout`、`setInterval`、`UI交互事件`、`postMessage`、`Ajax`

#### 2.微任务

`Promise.then` `catch` `finally`、`MutaionObserver`、`process.nextTick`(Node.js 环境)