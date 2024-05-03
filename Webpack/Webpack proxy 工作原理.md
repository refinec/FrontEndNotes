> `webpack proxy`，即`webpack`提供的代理服务。基本行为就是接收客户端发送的请求后转发给其他服务器。`webpack`中提供服务器的工具为`webpack-dev-server`

在开发阶段，`webpack-dev-server`会启动一个本地开发服务器，所以我们的应用在开发阶段是独立运行在`localhost`的一个端口上。通过设置`webpack proxy`实现代理请求后，相当于浏览器与服务端之间添加了一个代理者。

`proxy` 工作原理实质上是利用`http-proxy-middleware`这个`http`代理中间件，实现请求转发给其他服务器。

举个例子：在开发阶段，本地地址为`http://localhost:3000`，该浏览器发送一个前缀带有`/api`标识的请求到服务端获取数据，但响应这个请求的服务器只是将请求转发到另一台服务器中

```js
const express = require('express')
const proxy = require('http-proxy-middleware')

const app = express()

app.use('/api', proxy({
  target: 'http://www.example.org',
  changeOrigin: true
}))

app.listen(3000)

// http://localshot:300/api/foo/bar -> http://www.example.org/api/foo/bar
```

