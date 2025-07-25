## H5 是什么？-->>移动端的网页

**h5一般指的是开一个WebView来加载页面。WebView是一种控件，它基于webkit引擎，因此具备渲染Web页面的功能**

基于Webview的混合开发，就是在 Anddroid (安卓)/(苹果)原生APP里，通过WebView控件嵌入Web页面。
很多APP都是外边套原生APP的壳，内容是H5页面(基于html+css+js的Web页面)。现在的移动端混合开发软件，如果对于交互渲染要求不是特别高的项目，基本都是这么玩的。

**WebView作用：**

- 显示和渲染Web页面
- 直接使用html文件（网络上或本地assets中）作布局
- 可和JavaScript交互调用

**HTML5新特性：**

1. 本地存储特性
2. 设备兼容特性 HTML5提供了前所未有的数据与应用接入开放接口
3. 连接特性 WebSockets
4. 网页多媒体特性 支持Audio Video SVG Canvas WebGL CSS3
5. CSS3特性

## DOM

### 事件冒泡

事件会从最内层的元素开始发生，一直向上传播，直到document对象

### **事件捕获**

与事件冒泡相反，事件会从最外层开始发生，直到最具体的元素

#### `addEventListener`

addEventListener方法用来为一个特定的元素绑定一个事件处理函数，是JavaScript中的常用方法。

```
 element.addEventListener(event, function, useCapture)
```

重点来看看第三个参数`useCapture`

- true - 事件句柄在捕获阶段执行（即在事件捕获阶段调用处理函数）
- false- false- 默认。事件句柄在冒泡阶段执行（即表示在事件冒泡的阶段调用事件处理函数）

#### `attachEvent`

兼容IE的写法，默认是事件冒泡阶段调用处理函数，写**事件名时候要加上"on"前缀 **（"onload"、"onclick"等）。

```
object.attachEvent(event, function)
```

### 阻止事件冒泡和默认事件

```javascript
event.preventDefault()   // 阻止默认事件
event.stopPropagation() //阻止冒泡
```

### 实现一个可以拖拽的DIV

```html
<div id="xxx"></div>
```

```javascript
var dragging = false
var position = null

xxx.addEventListener('mousedown',function(e){
  dragging = true
  position = [e.clientX, e.clientY]
})
document.addEventListener('mousemove', function(e){
  if(dragging === false) return null
  console.log('hi')
  const x = e.clientX
  const y = e.clientY
  const deltaX = x - position[0]
  const deltaY = y - position[1]
  const left = parseInt(xxx.style.left || 0)
  const top = parseInt(xxx.style.top || 0)
  xxx.style.left = left + deltaX + 'px'
  xxx.style.top = top + deltaY + 'px'
  position = [x, y]
})
document.addEventListener('mouseup', function(e){
  dragging = false
})
```



## src与href的区别

### src的特性

1. 引用外部资源

   如`script`元素、`img`元素、`iframe`元素、`video`元素

2. 会替换元素本身的内容比

   **如下面这两段代码，会打印出2，而不是打印1**。原因就是`test.js`的代码嵌入到了当前`script`元素中，导致原本的内容被替换。

   ```js
   // test.js
   console.log(2)
   ```

   ```html
   <script src="./test.js">
       console.log(1)
   </script>
   ```

3. 会暂停其他资源的下载

   当浏览器解析到使用src的元素时，会暂停其他资源的下载，直到src引用资源加载、编译、执行完毕。这也是为什么script元素推荐放在html结构的底部

### href的特性

1. 表示超链接

   如`a`标签、`link`标签，表示**外部资源与该页面的联系**

2. 不会替换元素本身的内容

   如下面这段代码，点击a标签跳转到另外一个页面，但图片内容没有被替换

   ```html
   <a href="www.baidu.com">
       <img src="xxx"/>
   </a>
   ```

3. 不会暂停其他资源的下载

   像CSS那样影响页面观感的可以放在html结构的头部优先加载

### 区别

- 为可替换元素使用`src`属性, 而使用`href`属性建立参考文档与外部资源之间的联系
- `src`代表的是网站的一部分，没有则会对网站的使用造成影响
- `href`代表网站的附属资源，没有不会对网站的核心逻辑和结构造成影响