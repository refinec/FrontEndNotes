# HTML

## meta viewport 是做什么用的，怎么写

> 通常viewport是指视窗、视口。浏览器上(也可能是一个app中的webview)用来显示网页的那部分区域。在移动端和pc端视口是不同的，pc端的视口是浏览器窗口区域，而在移动端有三个不同的视口概念：布局视口、视觉视口、理想视口

1. **meta有两个属性name 和 http-equiv**

   **name(7个):**

   * keywords(关键字) 告诉搜索引擎，你网页的关键字

   * description(网站内容描述) 用于告诉搜索引擎，你网站的主要内容

   * viewport(移动端的窗口) 

   * robots(定义搜索引擎爬虫的索引方式) robots用来告诉爬虫哪些页面需要索引，哪些页面不需要索引

   * author(作者)

   * generator(网页制作软件）

   * copyright(版权)

   **http-equiv： **http-equiv顾名思义，相当于http的文件头作用

   * content-Type 设定网页字符集

   * X-UA-Compatible(浏览器采用哪种版本来渲染页面)

     //指定IE和Chrome使用最新版本渲染当前页面

   * cache-control（请求和响应遵循的缓存机制）

   * expires(网页到期时间)

```html
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>当下软件园-绿色软件_最新绿色下载软件_免费软件下载网站 - 当下软件园</title>
<meta name="keywords" content="软件下载,下载软件,绿色软件,免费软件,绿色软件园">
<meta name="description" content="当下软件园汇聚当下最新最酷的绿色软件,更新快,种类全,所有软件均经过检测,安全无毒, DOWN下——为中国5亿网民提供贴心,省心,放心的免费软件下载网站。">
<meta http-equiv="mobile-agent" content="format=html5; url=http://m.downxia.com"/>
```

## H5 是什么？-->>移动端页面

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

#### addEventListener

addEventListener方法用来为一个特定的元素绑定一个事件处理函数，是JavaScript中的常用方法。

```
 element.addEventListener(event, function, useCapture)
```

重点来看看第三个参数`useCapture`

- true - 事件句柄在捕获阶段执行（即在事件捕获阶段调用处理函数）
- false- false- 默认。事件句柄在冒泡阶段执行（即表示在事件冒泡的阶段调用事件处理函数）

#### attachEvent

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

## 标签语义化

（1）html结构清晰，代码可读性较好。 

（2）有利于SEO，搜索引擎根据标签来确定上下文和各个关键字的权重。 

（3）无障碍阅读，样式丢失的时候能让页面呈现清晰的结构。 

（4）方便其他设备解析，如盲人阅读器根据语义渲染网页 

（5）便于团队维护和开发，语义化更具可读性，代码更好维护，与CSS3关系更和谐。