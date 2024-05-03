关于`webpack`热模块更新的总结如下：

1. 通过`webpack-dev-server`创建两个服务器：提供静态资源的服务(`express`) 和 `Socket`服务
   * `express server` 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
   * `socket server` 是一个`websocket`的长连接，双方可以通信
2. 当`socker server` 监听到对应的模块发生变化时，会生成两个文件:
   * `xxxx.hot-update.json`(`manifest`文件，包含了文件内容`hash`和`chundId`，用来说明变化的内容) 
   *  `index.xxxx.hot-update.js`（`update chunk`文件）

3. 通过长连接，`socket server` 可以直接将这两个文件主动发送给浏览器
4. 浏览器拿到两个新的文件后吗，通过`HMR runtime`机制，加载这两个文件，并且针对修改的模块进行更新