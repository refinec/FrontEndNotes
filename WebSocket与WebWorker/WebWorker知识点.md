> `Web Worker`分为`Dedicated Web Worker`和`Shared Web Worker`两类.

1. `Dedicated Web Worker`仅为创建它的JSVM进程服务，当其所属的JSVM进程结束该`Dedicated Web Worker`线程也将结束；
2. `Shared Web Worker`为创建它的JSVM进程所属页面的域名服务，当该域名下的所有JSVM进程均结束时该`Shared Web Worker`线程才会结束;

## 怎么使用 `web worker`

主线程使用new命令调用Worker()构造函数创建一个Worker线程

- `var worker = new Worker('xxxxx.js')`
- Worker构造函数接收参数为脚本文件路径

主线成指定监听函数监听Worker线程的返回消息

- `worker.onmessage = function (event) {console.log(event.data)}`
- `data`为Worker发来的数据

由于主线程与Worker线程存在通信限制,不再同一个上下文中,所以只能通过消息完成

- `worker.postMessage("hello world")`
- `worker.postMessage({action: "ajax", url: "xxxxx", method: "post"})`

当使用完成后，如果不需要再使用可以在主线程中关闭Worker

- `worker.terminate()`
- Worker也可以关闭自身,在Worker的脚本中执行 `self.close()`

错误的处理，主线程可以监听Worker是否错误,如果有错误则会触发主线程的error事件

- `  worker.onerror(function (evet) {        // ...    })`

### 在Worker中使用其他脚本

如果需要引用其他脚本可以使用`importScripts`

- `importScripts('scripts1.js')`
- 该方法可以同时加载多个脚本 `importScripts('scripts1.js'，'scripts2.js')`

### 数据通信

主线程与Worker之间通信时拷贝的方式进行,即是传值而不是传址。Worker中对通信数据的修改并不会影响到主线程。

> 事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给 Worker，后者再将它还原

但是拷贝的形式做数据传输会造成性能问题，比如主线程向Worker发送几百MB的数据,默认情况下,浏览器会生成一份拷贝。为了解决这个问题,JAVASCRIPT允许主线程将二进制数据直接转移给Worker,但是转移控制权后，主线程就不再能使用这些数据。这是为了防止多个线程同时修改数据的情况发生。

这种转移数据控制权的方法叫做 [Transferable Objects](https://link.juejin.cn?target=http%3A%2F%2Fw3c.github.io%2Fhtml%2Finfrastructure.html%23transferable-objects) 这使得主线程可以快速的把数据移交给子线程。不会产生性能负担

```js
interface Worker extends EventTarget, AbstractWorker {
    onmessage: ((this: Worker, ev: MessageEvent) => any) | null;
    onmessageerror: ((this: Worker, ev: MessageEvent) => any) | null;
    postMessage(message: any, transfer: Transferable[]): void;
    postMessage(message: any, options?: StructuredSerializeOptions): void;
    terminate(): void;
    addEventListener<K extends keyof WorkerEventMap>(type: K, listener: (this: Worker, ev: WorkerEventMap[K]) => any, options?: boolean | AddEventListenerOptions): void;
    addEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | AddEventListenerOptions): void;
    removeEventListener<K extends keyof WorkerEventMap>(type: K, listener: (this: Worker, ev: WorkerEventMap[K]) => any, options?: boolean | EventListenerOptions): void;
    removeEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | EventListenerOptions): void;
}
```

## `web worker`能力边界

在使用`Web Worker`前我们要了解它的能力边界

1. **同源限制**

   1. 以`http(s)://`协议加载给`Web Worker`线程运行的脚本时，其`URL`必须和**UI线程**所属页面的`URL`同源； 
   2. 不能加载客户端本地脚本给WebWorker线程运行（即采用`file://`协议），即使UI线程所属页面也是本地页面； 

2. **DOM和BOM限制**

   1. 无法访问UI线程所属页面的任何DOM元素；

   2.  可访问如下BOM元素:

      1. ` XMLHttpRequest/fetch`

      2. `setTimeout/clearTimeout`

      3. `setInterval/clearInterval`

      4. `location`

         注意该location指向的是WebWorker创建时以UI线程所属页面的当前Location为基础创建的WorkerLocation对象，即使此后页面被多次重定向，该location的信息依然保持不变。 1.2.5. navigator，注意该navigator指向的是WebWorker创建时以UI线程所属页面的当前Navigator为基础创建

      5. `navigator`

         注意该navigator指向的是WebWorker创建时以UI线程所属页面的当前Navigator为基础创建的WorkerNavigator对象，即使此后页面被多次重定向，该navigator的信息依然保持不变

3. **通信限制**

   UI线程和 `Web Worker`线程间必须通过消息机制进行通信。

## 一、`Dedicated Web Worker`详解

### 1. 基本使用

<span style="color:red;">**UI线程**</span>

```js
const worker = new Worker('work.js') // 若下载失败如404，则会默默地失败不会抛异常，即无法通过try/catch捕获。                                                                                                                      
const workerWithName = new Worker('work.js', {
    name: 'worker2'
}) // 为Worker线程命名，那么在Worker线程内的代码可通过 self.name 获取该名称。                                                                                        


worker.postMessage('Send message to worker.') // 发送文本消息                                                                                                                                                                     
worker.postMessage({
    type: 'message',
    payload: ['hi']
}) // 发送JavaScript对象，会先执行序列化为JSON文本消息再发送，然后在接收端自动反序列化为JavaScript对象。                                                                      
const uInt8Array = new Uint8Array(new ArrayBuffer(10))
for (let i = 0; i < uint8array.length; ++i) {
    uInt8Array[i] = i * 2
}
worker.postMessage(uInt8Array) // 以先序列化后反序列化的方式发送二进制数据，发送后主线程仍然能访问uInt8Array变量的数据，但会造成性能问题。                                                                                        
worker.postMessage(uInt8Array, [uInt8Array]) // 以Transferable Objets的方式发送二进制数据，发送后主线程无法访问uInt8Array变量的数据，但不会造成性能问题，适合用于影像、声音和3D等大文件运算。                                     


// 接收worker线程向主线程发送的消息                                                                                                                                                                                               
worker.onmessage = event => {
    console.log(event.data)
}
worker.addEventListener('message', event => {
    console.log(event.data)
})


// 当发送的消息序列化失败时触发该事件。                                                                                                                                                                                           
worker.onmessageerror = error => console.error(error)
// 捕获Worker线程发生的异常                                                                                                                                                                                                       
worker.onerror = error => {
    console.error(error)
}

// 关闭worker线程                                                                                                                                                                                                                 
worker.terminate()
```

<span style="color:red;">**Worker线程**</span>

```js
// Worker线程的全局对象为WorkerGlobalScrip，通过self或this引用。调用全局对象的属性和方法时可以省略全局对象。                                                                                                                      

// 接收主线程向worker线程发送的消息                                                                                                                                                                                               
self.addEventListener('message', event => {
    console.log(event.data)
})
addEventListener('message', event => {
    console.log(event.data)
})
this.onmessage = event => {
    console.log(event.data)
}
// 当发送的消息序列化失败时触发该事件。                                                                                                                                                                                           
self.onmessageerror = error => console.error(error)
// 向主线程发送消息                                                                                                                                                                                                               
self.postMessage('send text to main worker')

// 结束自身所在的Worker线程                                                                                                                                                                                                       
self.close()

// 导入其他脚本到当前的Worker线程，不要求所引用的脚本必须同域。                                                                                                                                                                   
self.importScripts('script1.js', 'script2.js')
```

### 2. 通过`Web Worker`运行本页脚本

#### 方式1：`Blob`和`URL.createObjectURL`

> **限制**：UI线程所属页面不是本地页面，即必须为`http(s)://`协议。

```js
const script = `addEventListener('message', event => {                                                                                                                                                                                    
  console.log(event.data)                                                                                                                                                                                                               
  postMessage('echo')                                                                                                                                                                                                                   
}`


const blob = new Blob([script])
const url = URL.createObjectURL(blob)
const worker = new Worker(url)
worker.onmessage = event => console.log(event.data)
worker.postMessage('main thread')
setTimeout(() => {
    worker.terminate()
    URL.revokeObjectURL(url) // 必须手动释放资源，否则需要刷新Browser Context时才会被释放。                                                                                                                                               
}, 1000)
```

#### 方式2：`Data URL`

> **限制**：无法利用JavaScript的ASI机制少写分号。 
>
> **优点**：即使UI线程所属页面是本地页面也可以执行

```js
// 由于Data URL的内容为必须压缩为一行，因此JavaScript无法利用换行符达到分号的效果。                                                                                                                                                       
const script = `addEventListener('message', event => {                                                                                                                                                                                    
  console.log(event.data);                                                                                                                                                                                                              
  postMessage('echo');                                                                                                                                                                                                                  
}`

const worker = new Worker(`data:,${script}`)
// 或 const worker = new Worker(`data:application/javascript,${script}`)                                                                                                                                                                  
worker.onmessage = event => console.log(event.data)
worker.postMessage('main thread')
```

## 二、`Shared Web Worker` 详解

共享线程可以和多个同域页面间通信，当所有相关页面都关闭时共享线程才会被释放。
这里的多个同域页面包括：

1. iframe之间
2. 浏览器标签页之间

### 简单示例

**UI主线程**

```js
const worker = new SharedWorker('./worker.js')
worker.port.addEventListener('message', e => {
    console.log(e.data)
}, false)
worker.port.start()  // 连接worker线程                                                                                                                                                                                                     
worker.port.postMessage('hi')

setTimeout(() => {
    worker.port.close() // 关闭连接                                                                                                                                                                                                          
}, 10000)
```

`Shared Web Worker`线程

```js
let conns = 0

// 当UI线程执行worker.port.start()时触发建立连接                                                                                                                                                                                           
self.addEventListener('connect', e => {
    const port = e.ports[0]
    conns += 1

    port.addEventListener('message', e => {
        console.log(e.data)  // 注意console对象指向第一个创建Worker线程的UI线程的console对象。即如果A先创建Worker线程，那么后续B、C等UI线程执行worker.port.postMessage时回显信心依然会发送给A页面。                                            
    })

    // 建立双向连接，可相互通信                                                                                                                                                                                                              
    port.start()
    port.postMessage('hey')
})
```

### 示例——广播

**UI主线程**

```js
const worker = new SharedWorker('./worker.js')
worker.port.addEventListener('message', e => {
    console.log('SUM:', e.data)
}, false)
worker.port.start() // 连接worker线程

const button = document.createElement('button')
button.textContent = 'Increment'
button.onclick = () => worker.port.postMessage(1)
document.body.appendChild(button)
```

`Shared Web Worker`线程

```js
let sum = 0
const conns = []
self.addEventListener('connect', e => {
    const port = e.ports[0]
    conns.push(port)

    port.addEventListener('message', e => {
        sum += e.data
        conns.forEach(conn => conn.postMessage(sum))
    })

    port.start()
})
```

## 工程化

### 通过Webpack的worker-loader打包代码

`web worker`实际项目中应该怎么使用呢？或者说如何更好的集成到工程自动化工具——Webpack呢？` worker-loader`和`shared-worker-loader`就是我们想要的。 通过`worker-loader`将代码转换为Blob类型，并通过`URL.createObjectURL`创建`url`分配给WebWorker线程执行。

1. 安装

   `npm install worker-loader -D`

2. 配置`Webpack.config.js`

   ```json
   // 处理worker代码的loader必须位于js和ts之前                                                                                                                                                                                              
   {
       test: /\.worker\.ts$/,
       use: {
           loader: 'worker-loader',
           options: {
               name: '[name]:[hash:8].js', // 打包后的chunk的名称                                                                                                                                                                                 
               inline: true // 开启内联模式，将chunk的内容转换为Blob对象内嵌到代码中。                                                                                                                                                            
           }
       }
   }， {
       test: /\.js$/,
       use: {
           loader: 'babel-loader'
       },
       exclude: [path.resolve(__dirname, 'node_modules')]
   }, {
       test: /\.ts(x?)$/,
       use: [{
               loader: 'babel-loader'
           },
           {
               loader: 'ts-loader'
           } // loader顺序从后往前执行                                                                                                                                                                                    
       ],
       exclude: [path.resolve(__dirname, 'node_modules')]
   }
   ```

3. UI线程代码

   ```js
   import MyWorker from './my.worker'
   
   const worker = new MyWorker('');
   worker.postMessage('hi')
   worker.addEventListener('message', event => console.log(event.data))
   ```

4. Worker线程代码

   ```ts
   const worker: Worker = self as any
   worker.addEventListener('message', event => console.log(event.data))
   
   export default null as any // 标识当前为TS模块，避免报xxx.ts is not a module的异常
   ```

### RPC类库Comlink

一般场景下我们会这样使用`Web Worker`，

1. UI线程传递参数并调用运算函数；
2. 在不影响用户界面响应的前提下等待函数返回值；
3. 获取函数返回值继续后续代码。

翻译为代码就是

```js
let arg1 = getArg1()
let arg2 = getArg2()
const result = await performCalcuation(arg1, arg2)
doSomething(result)
```

而UI线程和`Web Worker`线程的消息机制通信机制显然会加大代码复杂度，而`Comlink`类库恰好能抚平这道伤疤

1. UI线程

   ```js
   import * as Comlink from 'comlink'
   
   async function init() {
       const cl = Comlink.wrap(new Worker('worker.js'))
       console.log(`Counter: ${await cl.counter}`)
       await cl.inc()
       console.log(`Counter: ${await cl.counter}`)
   }
   ```

2. Worker线程

   ```js
   import * as Comlink from 'comlink'
   
   const obj = {
       counter: 0,
       inc() {
           this.counter += 1
       }
   }
   Comlink.expose(obj)
   ```

## Electron中使用WebWorker

Electron中使用Web Worker的同源限制中开了个口——**UI线程所属页面URL为本地文件时，所分配给Web Worker的脚本可为本地脚本**。
 其实Electron打包后读取的HTML页面、脚本等都是本地文件，如果不能分配本地脚本给Web Worker执行，那就进入死胡同了。

```js
const path = window.require('path')
const worker = new Worker(path.resolve(__dirname, 'worker.js'))
```

上述代码仅表示Electron可以分配本地脚本给WebWorker线程执行，但实际开发阶段一般是通过 ~~http(s)://~~ 协议加载页面资源，而发布时才会打包为本地资源。
 所以这里还要分为开发阶段用和发布用代码，还涉及资源的路径问题，所以还不如直接转换为Blob数据内嵌到UI线程的代码中更便捷。

## 前端项目内使用

方式一：

```js
import MyWorker from './delayWorker.js?worker&inline'

const worker = new MyWorker()
```

方式二：

```js
import MyWorker from './delayWorker.js?raw'

const blob = new Blob([MyWorker], { type: 'application/javascript' });
const workUrl = window.URL.createObjectURL(blob);
const worker = new Worker(workUrl);
```

方式三：

worker导出为函数；需要一个转换函数把 worker 转换为URL；

```js
export function hilitorWorker() {
  self.addEventListener(
    'message',
    function (e) {
      const {
        data: { type, payload },
      } = e;
    });
}

export function createWorker(fn: any) {
  return new Worker(URL.createObjectURL(new Blob([`(${fn})()`], { type: 'text/javascript' })));
}

const worker = createWorker(hilitorWorker);
```

