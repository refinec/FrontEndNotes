# JavaScript 执行机制

1. 同步和异步任务分别进入不同的执行"场所"，**同步的进入主线程**，**异步的进入Event Table并注册回调函数**。
2. 当指定的事情完成时，**Event Table会将这个函数移入Event Queue **。
3. 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
4. 上述过程会不断重复，也就是常说的**Event Loop(事件循环) **。

js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。

```javascript
let data = [];
$.ajax({
    url:www.javascript.com,
    data:data,
    success:() => {
        console.log('发送成功!');
    }
})
console.log('代码执行结束');
```

上面是一段简易的`ajax`请求代码：

1. ajax进入Event Table，注册回调函数`success`。
2. 执行`console.log('代码执行结束')`。
3. ajax事件完成，回调函数`success`进入Event Queue。
4. 主线程从Event Queue读取回调函数`success`并执行。

## 拿setTimeout来讲

```javascript
setTimeout(() => {
    task()
},3000)
sleep(10000000)
```

我们把这段代码在chrome执行一下，却发现控制台执行`task()`需要的时间远远超过3秒，说好的延时三秒，为啥现在需要这么长时间啊？

这时候我们需要重新理解`setTimeout`的定义。我们先说上述代码是怎么执行的：

1. `task()`进入Event Table并注册,计时开始。
2. 执行`sleep`函数，很慢，非常慢，计时仍在继续。
3. 3秒到了，计时事件`timeout`完成，`task()`进入**Event Queue**，但是`sleep`也太慢了吧，还没执行完，只好等着。
4. `sleep`终于执行完了，`task()`终于从Event Queue进入了主线程执行。

上述的流程走完，我们知道**`setTimeout`这个函数，是经过指定时间后，把要执行的任务(本例中为`task()`)加入到Event Queue中**，又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着，导致真正的延迟时间远远大于3秒。

### setTimeout(fn,0)

我们还经常遇到`setTimeout(fn,0)`这样的代码，0秒后执行又是什么意思呢？是不是可以立即执行呢？

答案是不会的，`setTimeout(fn,0)`的含义是，**指定某个任务在主线程最早可得的空闲时间执行**，意思就是不用再等多少秒了，**只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行 **。举例说明：

```javascript
//代码1
console.log('先执行这里');
setTimeout(() => {
    console.log('执行啦')
},0);
//先执行这里
//执行啦


//代码2
console.log('先执行这里');
setTimeout(() => {
    console.log('执行啦')
},3000);  
//先执行这里
// ... 3s later
// 执行啦
```

关于`setTimeout`要补充的是，即便主线程为空，0毫秒实际上也是达不到的。根据HTML的标准，最低是4毫秒

## 拿setInterval来讲

对于执行顺序来说，`setInterval`会**每隔指定的时间将注册的函数置入Event Queue **，如果前面的任务耗时太久，那么同样需要等待。

唯一需要注意的一点是，对于`setInterval(fn,ms)`来说，我们已经知道不是每过`ms`秒会执行一次`fn`，而是每过`ms`秒，会有`fn`进入Event Queue。一旦**`setInterval`的回调函数`fn`执行时间超过了延迟时间`ms`，那么就完全看不出来有时间间隔了**。

## Promise与process.nextTick(callback)

`process.nextTick(callback)`类似node.js版的"setTimeout"，在事件循环的**下一次循环中调用 callback 回调函数 **。

除了广义的同步任务和异步任务，我们对任务有更精细的定义：

- **macro-task(宏任务)：包括整体代码script，setTimeout，setInterval**
- **micro-task(微任务)：Promise，process.nextTick**

不同类型的任务会进入对应的Event Queue，比如`setTimeout`和`setInterval`会进入相同的Event Queue。

**事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。**

```javascript
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
    resolve();
}).then(function() {
    console.log('then');
})

console.log('console');
```

1. 这段代码作为宏任务，进入主线程。
2. 先遇到`setTimeout`，那么将其回调函数注册后分发到**宏任务Event Queue**。(注册过程与上同，下文不再描述)
3. 接下来遇到了`Promise`，`new Promise`立即执行，`then`函数分发到**微任务Event Queue**。
4. 遇到`console.log()`，立即执行。
5. 好啦，整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了`then`在微任务Event Queue里面，执行。
6. ok，第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event Queue开始。我们发现了宏任务Event Queue中`setTimeout`对应的回调函数，立即执行。
7. 结束。

事件循环，宏任务，微任务的关系如图所示：

```javascript
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})

process.nextTick(function() {
    console.log('6');
})

new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})

```

第一轮事件循环流程分析如下：

- 整体script作为第一个宏任务进入主线程，遇到`console.log`，输出1。
- 遇到`setTimeout`，其回调函数被分发到**宏任务Event Queue**中。我们暂且记为`setTimeout1`。
- 遇到`process.nextTick()`，其回调函数被分发到**微任务Event Queue**中。我们记为`process1`。
- 遇到`Promise`，`new Promise`直接执行，输出7。`then`被分发到**微任务Event Queue**中。我们记为`then1`。
- 又遇到了`setTimeout`，其回调函数被分发到宏任务Event Queue中，我们记为`setTimeout2`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
| setTimeout1       | process1          |
| setTimeout2       | then1             |

- 上表是第一轮事件循环宏任务结束时各Event Queue的情况，此时已经输出了1和7。
- 我们发现了`process1`和`then1`两个微任务。
- 执行`process1`,输出6。
- 执行`then1`，输出8。

好了，第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮时间循环从`setTimeout1`宏任务开始：

- 首先输出2。接下来遇到了`process.nextTick()`，同样将其分发到微任务Event Queue中，记为`process2`。`new Promise`立即执行输出4，`then`也分发到微任务Event Queue中，记为`then2`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
| setTimeout2       | process2          |
|                   | then2             |

- 第二轮事件循环宏任务结束，我们发现有`process2`和`then2`两个微任务可以执行。
- 输出3。
- 输出5。
- 第二轮事件循环结束，第二轮输出2，4，3，5。
- 第三轮事件循环开始，此时只剩setTimeout2了，执行。
- 直接输出9。
- 将`process.nextTick()`分发到微任务Event Queue中。记为`process3`。
- 直接执行`new Promise`，输出11。
- 将`then`分发到微任务Event Queue中，记为`then3`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
|                   | process3          |
|                   | then3             |

- 第三轮事件循环宏任务执行结束，执行两个微任务`process3`和`then3`。
- 输出10。
- 输出12。
- 第三轮事件循环结束，第三轮输出9，11，10，12。

整段代码，共进行了三次事件循环，完整的输出为`1，7，6，8，2，4，3，5，9，11，10，12`。 (请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)

## `async/await`

```js
async function async1() {
       console.log('async1 start')
       await async2()
       console.log('async1 end')         // 1
   }
    
   async function async2() {
       console.log('async2')
   }
    
  console.log('script start')
    
  async1()
    
  setTimeout(() => {                    // 2
      console.log('setTimeout')
  }, 0)
    
  new Promise((resolve) => {
      console.log('promise')
      resolve()
  }).then(function () {                 // 3
      console.log('promise1 start')
      return 'promise1 end'
  }).then(res => {                      // 4
      console.log(res)
  }).then(res => {                      // 5
      console.log(res)
  })
    
  console.log('script end')
```

输出结果：

```
script start
async1 start
async2
promise
script end
async1 end
promise1 start
promise1 end
undefined
setTimeout
```

`await async2()` 相当于

```js
new Promise(resolve => {
    async2()
    resolve()
}).then(() => {
    console.log('async1 end')
})
```

`async`标记的函数返回一个Promise对象，因此，我们又可以看成

```js
await async2().then(() => { // 微任务1
    console.log('async1 end')
})
```

**关于多个`then`:**

1. 上一个then回调代码都是同步执行，执行结束后下一个then可以加入到微任务队列中；
2. 上一个then回调有return关键字时，需要等待return的内容完全执行完毕后，下一个then才能加入到微任务队列中

**再举一个例子：**

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')           // 1
  new Promise(resolve => {
    console.log('async promise')
    resolve('async promise start')
  }).then(res => {                    // 2
    console.log(res)
  })
}

async function async2() {
  console.log('async2')
}

async1()

new Promise((resolve) => {
  console.log('promise')
  resolve()
}).then(function () {                 // 3
  console.log('promise1 start')
  return 'promise1 end'
}).then(res => {                      // 4
  console.log(res)
}).then(res => {                      // 5
  console.log(res)
})
```

输出结果：

```
async1 start
async2
promise
async1 end
async promise
promise1 start
async promise start
promise1 end
undefined
```

## 其他

### 堆（Heap）

**堆**是一种数据结构，是利用**完全二叉树**维护的一组数据，**堆**分为两种，一种为最大**堆**，一种为**最小堆**，将根节点**最大**的**堆**叫做**最大堆**或**大根堆**，根节点**最小**的**堆**叫做**最小堆**或**小根堆**。
**堆**是**线性数据结构**，相当于**一维数组**，有唯一后继。

### 栈（Stack）

**栈**在计算机科学中是限定仅在**表尾**进行**插入**或**删除**操作的线性表。 **栈**是一种数据结构，它按照**后进先出**的原则存储数据，**先进入**的数据被压入**栈底**，**最后的数据**在**栈顶**，需要读数据的时候从**栈顶**开始**弹出数据**。
**栈**是只能在**某一端插入**和**删除**的**特殊线性表**。

### 队列（Queue）

特殊之处在于它只允许在表的前端（`front`）进行**删除**操作，而在表的后端（`rear`）进行**插入**操作，和**栈**一样，**队列**是一种操作受限制的线性表。

进行**插入**操作的端称为**队尾**，进行**删除**操作的端称为**队头**。 队列中没有元素时，称为**空队列**。

### Event Loop

在`JavaScript`中，任务被分为两种，一种宏任务（`MacroTask`）也叫`Task`，一种叫微任务（`MicroTask`）

#### MacroTask（宏任务）

`script`全部代码、`setTimeout`、`setInterval`、`Ajax`、`I/O`、`UI Rendering`、`setImmediate`（浏览器暂时不支持，只有IE10支持，具体可见[`MDN`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setImmediate)）、`MessageChannel`。

#### MicroTask（微任务）

`Process.nextTick（Node独有）`、`Promise`、`Object.observe(废弃)`、`MutationObserver`（具体使用方式查看[这里](http://javascript.ruanyifeng.com/dom/mutationobserver.html)）

### 浏览器中的Event Loop

`Javascript` 有一个 `main thread` 主线程和 `call-stack` 调用栈(执行栈)，所有的任务都会被放到调用栈等待主线程执行。

**JS调用栈 **

JS调用栈采用的是**后进先出**的规则，当函数执行的时候，会被添加到栈的顶部，当执行栈执行完成后，就会从栈顶移出，直到栈内被清空。