# WebSocket 知识点

> HTTP 协议有一个缺陷：通信只能由客户端发起。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用["轮询"](https://www.pubnub.com/blog/2014-12-01-http-long-polling/)：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。
>
> 轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的

## 应用场景

>  **在服务器需要主动向浏览器发送消息、数据的场景**

## 特点

1. 建立在 TCP 协议之上，服务器端的实现比较容易。
2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
3. 允许浏览器和服务器之间建立持久连接
4. 数据格式比较轻量，性能开销小，通信高效。
5. 可以发送文本，也可以发送二进制数据。
6. 没有同源限制，客户端可以与任意服务器通信。
7. 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

![](https://www.ruanyifeng.com/blogimg/asset/2017/bg2017051503.jpg)

## 客户端简单示例

```javascript
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};
```

## WebSocket 构造函数 (客户端的API)

```js
var ws = new WebSocket(url [, protocols]);
```

- **url:** 

  要连接的URL；这应该是WebSocket服务器将响应的URL。

- **`protocols` 可选**

  一个协议字符串或者一个包含协议字符串的数组。这些字符串用于指定子协议，这样单个服务器可以实现多个WebSocket子协议（例如，您可能希望一台服务器能够根据指定的协议（`protocol`）处理不同类型的交互）。如果不指定协议字符串，则假定为空字符串

可能抛出的异常： **SECURITY_ERR** 正在尝试连接的端口被阻止

### webSocket.readyState（只读）

`readyState`属性返回实例对象的当前状态，共有四种：

- **CONNECTING**：值为0，表示正在连接。
- **OPEN**：值为1，表示连接成功，可以通信了。
- **CLOSING**：值为2，表示连接正在关闭。
- **CLOSED**：值为3，表示连接已经关闭，或者打开连接失败。

```js
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```

### webSocket.onopen

> **实例对象的`onopen`属性，用于指定连接成功后的回调函数**。

 ```javascript
 ws.onopen = function () {
   ws.send('Hello Server!');
 }
 ```

如果要指定多个回调函数，可以使用`addEventListener`方法。

 ```javascript
 ws.addEventListener('open', function (event) {
   ws.send('Hello Server!');
 });
 ```

### webSocket.onclose

> 实例对象的`onclose`属性，用于指定连接关闭后的回调函数

实例对象的`onclose`属性，用于指定连接关闭后的回调函数。

 ```javascript
 ws.onclose = function(event) {
   var code = event.code;
   var reason = event.reason;
   var wasClean = event.wasClean;
   // handle close event
 };
 # 或者
 ws.addEventListener("close", function(event) {
   var code = event.code;
   var reason = event.reason;
   var wasClean = event.wasClean;
   // handle close event
 });
 ```

### webSocket.onmessage

> 实例对象的`onmessage`属性，用于指定收到服务器数据后的回调函数

 ```javascript
 ws.onmessage = function(event) {
   var data = event.data;
   // 处理数据
 };
 
 ws.addEventListener("message", function(event) {
   var data = event.data;
   // 处理数据
 });
 ```

注意，服务器数据可能是文本，也可能是二进制数据（`blob`对象或`Arraybuffer`对象）。

 ```javascript
 ws.onmessage = function(event){
   if(typeof event.data === String) {
     console.log("Received data string");
   }
 
   if(event.data instanceof ArrayBuffer){
     var buffer = event.data;
     console.log("Received arraybuffer");
   }
 }
 ```

除了动态判断收到的数据类型，也可以使用`binaryType`属性，显式指定收到的二进制数据类型。

 ```javascript
 // 收到的是 blob 数据
 ws.binaryType = "blob";
 ws.onmessage = function(e) {
   console.log(e.data.size);
 };
 
 // 收到的是 ArrayBuffer 数据
 ws.binaryType = "arraybuffer";
 ws.onmessage = function(e) {
   console.log(e.data.byteLength);
 };
 ```

### webSocket.onerror

> **实例对象的`onerror`属性，用于指定报错时的回调函数。**

```js
socket.onerror = function(event) {
  // handle error event
};
socket.addEventListener("error", function(event) {
  // handle error event
});
```

### webSocket.binaryType

> 返回websocket连接所传输二进制数据的类型

```js
var binaryType = ws.binaryType;
```

- **"blob"**

  如果传输的是 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 类型的数据。

- **"arraybuffer"**

  如果传输的是 [`ArrayBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 类型的数据。

### webSocket.close([code[, reason]])

> 该方法关闭 [`WebSocket`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket) 连接或连接尝试（如果有的话）。 如果连接已经关闭,则此方法不执行任何操作。

- **`code` 可选**

  一个数字状态码，它解释了连接关闭的原因。如果没有传这个参数，默认使用1005。[`CloseEvent`](https://developer.mozilla.org/zh-CN/docs/Web/API/CloseEvent)的允许的状态码见[状态码列表](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent#Status_codes) 。

- **`reason` 可选**

  一个人类可读的字符串，它解释了连接关闭的原因。这个UTF-8编码的字符串不能超过123个字节。

**可能抛出的异常：**

* **INVALID_ACCESS_ERR**

  一个无效的`code`

* **SYNTAX_ERR**

  `reason` 字符串太长（超过123字节）

### webSocket.send()

> **实例对象的`send()`方法用于向服务器发送数据**

1. 发送文本的例子

   ```js
   ws.send('your message');
   ```


2. 发送 Blob 对象的例子

   ```js
   var file = document
     .querySelector('input[type="file"]')
     .files[0];
   ws.send(file);
   ```


3. 发送 ArrayBuffer 对象的例子

   ```js
   // Sending canvas ImageData as ArrayBuffer
   var img = canvas_context.getImageData(0, 0, 400, 320);
   var binary = new Uint8Array(img.data.length);
   for (var i = 0; i < img.data.length; i++) {
     binary[i] = img.data[i];
   }
   ws.send(binary.buffer);
   ```

### webSocket.bufferedAmount(只读)

> **实例对象的`bufferedAmount`属性，表示还有多少字节的二进制数据没有发送出去，它可以用来判断发送是否结束。但是，若在发送过程中连接被关闭，则属性值不会重置为0。另外，不断地调用`send()`，则该属性值会持续增长**

```js
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

###  webSocket.extensions(只读)

> 返回服务器已选择的扩展值。目前，链接可以协定的扩展值只有空字符串或者一个扩展列表。

```js
var extensions = ws.extensions;
```

### webSocket.protocol(只读)

> 回服务器端选中的子协议的名字；这是一个在创建[`WebSocket`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket) 对象时，在参数`protocols`中指定的字符串，当没有已建立的链接时为空串

```js
var protocol = ws.protocol;
```

### webSocket.url(只读)

> 返回值为当构造函数创建[`WebSocket`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)实例对象时URL的绝对路径

```js
var url = ws.url;
```

## 服务端的实现

WebSocket 服务器的实现，可以查看维基百科的[列表](https://en.wikipedia.org/wiki/Comparison_of_WebSocket_implementations)。

常用的 Node 实现有以下三种。

- [µWebSockets](https://github.com/uWebSockets/uWebSockets)
- [Socket.IO](http://socket.io/)
- [WebSocket-Node](https://github.com/theturtle32/WebSocket-Node)

# WebSocketd

一款非常特别的 WebSocket 服务器：[Websocketd](http://websocketd.com/)。

它的最大特点，就是后台脚本不限语言，标准输入（stdin）就是 WebSocket 的输入，标准输出（stdout）就是 WebSocket 的输出。

![img](https://www.ruanyifeng.com/blogimg/asset/2017/bg2017051504.png)

举例来说，下面是一个 Bash 脚本`counter.sh`。

> ```bash
> #!/bin/bash
> 
> echo 1
> sleep 1
> 
> echo 2
> sleep 1
> 
> echo 3
> ```

命令行下运行这个脚本，会输出1、2、3，每个值之间间隔1秒。

> ```bash
> $ bash ./counter.sh
> 1
> 2
> 3
> ```

现在，启动`websocketd`，指定这个脚本作为服务。

> ```bash
> $ websocketd --port=8080 bash ./counter.sh
> ```

上面的命令会启动一个 WebSocket 服务器，端口是`8080`。每当客户端连接这个服务器，就会执行`counter.sh`脚本，并将它的输出推送给客户端。

> ```javascript
> var ws = new WebSocket('ws://localhost:8080/');
> 
> ws.onmessage = function(event) {
>   console.log(event.data);
> };
> ```

上面是客户端的 JavaScript 代码，运行之后会在控制台依次输出1、2、3。

有了它，就可以很方便地将命令行的输出，发给浏览器。

> ```bash
> $ websocketd --port=8080 ls
> ```

上面的命令会执行`ls`命令，从而将当前目录的内容，发给浏览器。使用这种方式实时监控服务器，简直是轻而易举（[代码](https://github.com/joewalnes/web-vmstats)）。

![img](https://www.ruanyifeng.com/blogimg/asset/2017/bg2017051505.jpg)

更多的用法可以参考[官方示例](https://github.com/joewalnes/websocketd/tree/master/examples/bash)。

> - Bash 脚本[读取客户端输入](https://github.com/joewalnes/websocketd/blob/master/examples/bash/greeter.sh)的例子
> - 五行代码实现一个最简单的[聊天服务器](https://github.com/joewalnes/websocketd/blob/master/examples/bash/chat.sh)

![img](https://www.ruanyifeng.com/blogimg/asset/2017/bg2017051506.png)

websocketd 的实质，就是命令行的 WebSocket 代理。只要命令行可以执行的程序，都可以通过它与浏览器进行 WebSocket 通信。下面是一个 Node 实现的回声服务[`greeter.js`](https://github.com/joewalnes/websocketd/blob/master/examples/nodejs/greeter.js)。

> ```javascript
> process.stdin.setEncoding('utf8');
> 
> process.stdin.on('readable', function() {
>   var chunk = process.stdin.read();
>   if (chunk !== null) {
>     process.stdout.write('data: ' + chunk);
>   }
> });
> ```

启动这个脚本的命令如下。

> ```bash
> $ websocketd --port=8080 node ./greeter.js
> ```

官方仓库还有其他[各种语言](https://github.com/joewalnes/websocketd/tree/master/examples)的例子。

# 参考链接

- [How to Use WebSockets](http://cjihrig.com/blog/how-to-use-websockets/)
- [WebSockets - Send & Receive Messages](https://www.tutorialspoint.com/websockets/websockets_send_receive_messages.htm)
- [Introducing WebSockets: Bringing Sockets to the Web](https://www.html5rocks.com/en/tutorials/websockets/basics/)
