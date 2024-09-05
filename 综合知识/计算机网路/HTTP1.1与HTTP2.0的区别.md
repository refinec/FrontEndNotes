

# (重要)HTTP/2的多路复用和HTTP/1.X中的长连接复用的区别

- HTTP/1.X 一次请求-响应，建立一个连接，用完关闭；每一个请求都要建立一个连接；
- HTTP/1.1 Pipeling解决方式为，若干个请求排队**串行化单线程处理**，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；
- HTTP/2多个请求可同时在一个连接上**并行执行**。某个请求任务耗时严重，不会影响到其它连接的正常执行；

# HTTP1.1与HTTP2.0的区别

## 1.协议解析采用二进制格式（Binary Format）

HTTP1.x解析是基于文本的，基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

## 2.**多路复用**（MultiPlexing）

> 即连接共享，即每一个请求都是是用作连接共享机制的。
>

HTTP/2 添加了**二进制分帧层**，将请求或响应数据经过二进制分帧处理，转化为一个个带有**请求 ID 编号**的帧，服务器或者浏览器接收到响应帧后，把具有相同 ID 的帧合并为一条完整信息。

## 3.**首部压缩**

> 对请求头和响应头进行压缩
>

- HTTP1.x的header带有大量信息，而且每次都要重复发送

- HTTP2.0使用encoder来减少需要传输的header大小，**通讯双方各自缓存一份header fields表**，既避免了重复header的传输，又减小了需要传输的大小。

例如有如下两个请求：

```http
:authority: unpkg.zhimg.com
:method: GET
:path: /za-js-sdk@2.16.0/dist/zap.js
:scheme: https
accept: */*
accept-encoding: gzip, deflate, br
accept-language: zh-CN,zh;q=0.9
cache-control: no-cache
pragma: no-cache
referer: https://www.zhihu.com/
sec-fetch-dest: script
sec-fetch-mode: no-cors
sec-fetch-site: cross-site
user-agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36
```

```http
:authority: zz.bdstatic.com
:method: GET
:path: /linksubmit/push.js
:scheme: https
accept: */*
accept-encoding: gzip, deflate, br
accept-language: zh-CN,zh;q=0.9
cache-control: no-cache
pragma: no-cache
referer: https://www.zhihu.com/
sec-fetch-dest: script
sec-fetch-mode: no-cors
sec-fetch-site: cross-site
user-agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36
```

从上面两个请求可以看出来，有很多数据都是重复的。如果可以**把相同的首部存储起来，仅发送它们之间不同的部分**，就可以节省不少的流量，加快请求的时间。

HTTP/2 在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键－值对，对于相同的数据，不再通过每次请求和响应发送。

下面再来看一个简化的例子，假设客户端按顺序发送如下请求首部：

```http
Header1:foo
Header2:bar
Header3:bat
```

当客户端发送请求时，它会根据首部值创建一张表：

| 索引 | 首部名称 | 值   |
| ---- | -------- | ---- |
| 62   | Header1  | foo  |
| 63   | Header2  | bar  |
| 64   | Header3  | bat  |

如果服务器收到了请求，它会照样创建一张表。 当客户端发送下一个请求的时候，如果首部相同，它可以直接发送这样的首部块：**`62 63 64`**

服务器会查找先前建立的表格，并把这些数字还原成索引对应的完整首部。

## 4.**服务端推送**（server push）

同SPDY一样，HTTP2.0也具有server push功能。

**服务器可以对一个客户端请求发送多个响应。换句话说，除了对最初请求的响应外，服务器还可以额外向客户端推送资源，而无需客户端明确地请求。**

例如当浏览器请求一个网站时，除了返回 HTML 页面外，服务器还可以根据 HTML 页面中的资源的 URL(引用了哪些 JavaScript 和 CSS 文件)，附带一起发送给浏览器，来提前推送资源。

## 5.设置请求优先级

发送请求可以设置请求优先级，服务器可以优先处理；

## 6.解析速度快

服务器解析 HTTP1.1 的请求时，必须不断地读入字节，直到遇到分隔符 CRLF 为止。

而解析 HTTP2 的请求就不用这么麻烦，因为 HTTP2 是基于帧的协议，每个帧都有表示帧长度的字段。

## 7.流量控制

由于一个 TCP 连接流量带宽（根据客户端到服务器的网络带宽而定）是固定的，当有多个请求并发时，一个请求占的流量多，另一个请求占的流量就会少。流量控制可以对不同的流的流量进行精确控制。



# HTTP/1.0与HTTP/1.1的区别

## 1.长连接

* HTTP 1.1支持**长连接**（PersistentConnection）和**请求的管道**（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启`Connection: Keep-Alive`； 
* HTTP1.0默认使用短连接，规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪 每个客户也不记录过去的请求。要建立长连接，可以在请求消息中包含`Connection: Keep-Alive`头域，如果服务器愿意维持这条连接，在响应消息中也会包含一个`Connection: Keep-Alive`的头域。

## 2.缓存处理

- HTTP1.0中主要使用header里的`If-Modified-Since`,`Expires`来做为缓存判断的标准；
- HTTP1.1则引入了更多的缓存控制策略例如`Entity tag`，`If-None-Match`；`Last modified`，`If-modified-Since`, 等更多可供选择的缓存头来控制缓存策略

## 3.带宽优化

- HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能;
- HTTP1.1则在请求头引入了range头域，它允许**只请求资源的某个部分**，即返回码是`206`（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

## 4.错误通知的管理（状态码）

- 在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除

## 5.Host头处理

- 在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。
- HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

# **HTTP3：甩掉 TCP、TCL 包袱，构建高效网络**

- 虽然 HTTP/2 解决了应用层面的对头阻塞问题，不过和 HTTP/1.1 一样，HTTP/2 依然是基于 TCP 协议，而 TCP 最初是为了单连接而设计；
- TCP 可以看成是计算机之间的一个虚拟管道，数据从一端发送到另一端会被拆分为一个个按照顺序排列的数据包，如果在传输过程中，有一个数据因为网络故障或者其他原因丢失，那么整个连接会处于暂停状态，只有等到该数据重新传输；
- 由于 TCP 协议僵化，也不可能使用新的协议，HTTP/3 选择了一个折衷的方法，基于现有的 UDP 协议，实现类似 TC 片多路复用，传输可靠等功能，称为 QULC 协议；
- QULC 实现类似 TCP 流量控制，传输可靠功能；集成 TLS 加密功能；实现多路复用功能；