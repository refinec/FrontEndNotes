# axios与ajax的区别及优缺点

**区别：**

axios是通过Promise实现对ajax技术的一种封装，就像jquery对ajax的封装一样，简单来说就是ajax技术实现了局部数据的刷新，axios实现了对ajax的封装，axios有的ajax都有，ajax有的axios不一定有，总结一句话就是axios是ajax，ajax不止axios

**优缺点：**

* **ajax：**

1. 本身是针对MVC编程，不符合前端MVVM的浪潮

2. 基于原生XHR开发，XHR本身的架构不清晰，已经有了fetch的替代方案，jquery整个项目太大，单纯使用ajax却要引入整个jquery非常不合理（采取个性化打包方案又不能享受cdn服务）

3. ajax不支持浏览器的back按钮

4. 安全问题ajax暴露了与服务器交互的细节

5. 对搜索引擎的支持比较弱

6. 破坏程序的异常机制

7. 不容易调试

* **axios：**

1. 从浏览器中创建 XMLHttpRequests
2. 从node.js创建http请求

2. 支持Promise API
3. 拦截请求和响应
4. 转换请求数据和响应数据
5. 取消请求
6. 自动转换 JSON 数据

3. 客户端防止CSRF（网站恶意利用）

4. 提供了一些并发请求的接口