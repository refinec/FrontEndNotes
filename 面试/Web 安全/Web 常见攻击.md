##  `XSS` 跨站脚本

> XSS 全称是 `Cross Site Scripting` ,为了与`CSS`区分开来，故简称 `XSS`，翻译过来就是“***跨站脚本***”
>
> **XSS是指黑客往 HTML 文件中或者 DOM 中注入恶意脚本**，**从而当用户浏览页面时利用注入的恶意脚本对用户实施攻击的一种手段**。

注入恶意脚本可以完成这些事情：

1. **窃取Cookie**
2. **监听用户行为**，比如输入账号密码后之间发给黑客服务器
3. **在网页中生成浮窗广告**
4. **修改DOM伪造登入表单**

### `XSS`攻击的三种实现方式

一般的情况下，`XSS`攻击有三种实现方式:

- **存储型 `XSS` 攻击**
- **反射型 `XSS ` 攻击**
- **基于 DOM 的 `XSS` 攻击**

#### 1.存储型 `XSS` 攻击(存数据库)

**存储型 XSS 攻击大致步骤如下**：

1. 首先黑客利用站点漏洞将一段恶意 JavaScript 代码提交到网站的**数据库**中；
2. 然后用户向网站请求包含了恶意 JavaScript 脚本的页面；
3. 当用户浏览该页面的时候，恶意脚本就会将用户的 Cookie 信息等数据上传到服务器。

比如常见的场景：

> 在评论区提交一份脚本代码，假设前后端没有做好转义工作，那内容上传到服务器，在页面渲染的时候就会`直接执行`，相当于执行一段未知的JS代码。这就是存储型 XSS 攻击。

#### 2.反射型 `XSS ` 攻击(作为网络请求一部分)

> **反射型 XSS 攻击**指的就是恶意脚本作为**「网络请求的一部分」**，随后网站又把恶意的JavaScript脚本返回给用户，当恶意 JavaScript 脚本在用户页面中被执行时，黑客就可以利用该脚本做一些恶意操作。

举个例子:

```http
http://xxx.com?query=<script>alert("你受到了XSS攻击")</script>
```

如上，服务器拿到后解析参数query，最后将内容返回给浏览器，浏览器将这些内容作为HTML的一部分解析，发现是Javascript脚本，直接执行，这样子被XSS攻击了。

这也就是反射型名字的由来，将恶意脚本作为参数，通过网络请求，最后经过服务器，在反射到HTML文档中，执行解析。

#### 3.基于 DOM 的 `XSS` 攻击(网络数据包劫持)

> **基于 DOM 的 `XSS` 攻击是不牵涉到页面 Web 服务器的**。具体来讲，黑客通过各种手段将恶意脚本注入用户的页面中，在数据传输的时候劫持网络数据包

常见的劫持手段有：

- WIFI路由器劫持
- 本地恶意软件

### 如何预防？阻止 XSS 攻击的策略

> 除了以下策略之外，我们还可以通过添加**<u>验证码</u>**防止脚本冒充用户提交危险操作。
>
> 而对于一些不受信任的输入，还可以**<u>限制其输入长度</u>**，这样可以增大 XSS 攻击的难度。

#### 1. **对输入脚本进行过滤或转码**

#### 2. 使用Cookie的`HttpOnly`属性

由于很多 XSS 攻击都是来盗用 Cookie 的，因此还可以通过使用 HttpOnly 属性来保护我们 Cookie 的安全。这样子的话，JavaScript 便无法读取 Cookie 的值。这样也能很好的防范 XSS 攻击。

通常服务器可以将某些 Cookie 设置为 HttpOnly 标志，HttpOnly 是服务器通过 HTTP 响应头来设置的，下面是打开 Google 时，HTTP 响应头中的一段：

```http
set-cookie: NID=189=M8l6-z41asXtm2uEwcOC5oh9djkffOMhWqQrlnCtOI; expires=Sat, 18-Apr-2020 06:52:22 GMT; path=/; domain=.google.com; HttpOnly
```

#### 3. 利用 `CSP`(浏览器中的内容安全策略)

该安全策略的实现基于一个称作 `Content-Security-Policy`的 [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP) 首部。

`CSP`的核心思想大概就是服务器决定浏览器加载哪些资源，具体来说有几个功能：**<u>限制加载其他域下的资源文件</u>**，这样即使黑客插入了一个 JavaScript 文件，这个 JavaScript 文件也是无法被加载的；

#### 4. 禁止向第三方域提交数据，这样用户数据也不会外泄；

#### 5. 提供上报机制，能帮助我们及时发现 XSS 攻击

#### 6. 禁止执行内联脚本和未授权的脚本

## `CSRF` 跨站请求伪造

> CSRF 英文全称是 `Cross-site request forgery`，所以又称为“***跨站请求伪造***”，是指***黑客引诱用户打开黑客的网站，在黑客的网站中，利用用户的登录状态发起的跨站请求***。简单来讲，**CSRF 攻击就是黑客利用了用户的登录状态，并通过第三方的站点来做一些坏事**

### `CSRF`攻击的三种方式

#### 1. 自动发起 Get 请求

黑客网页里面可能有一段这样的代码👇

```html
 <img src="http://bank.example/withdraw?amount=10000&for=hacker" /> 
```

在受害者访问含有这个`img`的页面后，浏览器会自动向`http://bank.example/withdraw?account=xiaoming&amount=10000&for=hacker`发出一次HTTP请求。

`bank.example`就会收到包含受害者登录信息的一次跨域请求

#### 2.自动发起 POST 请求

黑客网页中有一个表单，自动提交的表单👇

```html
<html>
  <body>
    <form action="http://bank.example/withdraw" method=POST>
      <input type="hidden" name="account" value="xiaoming" />
      <input type="hidden" name="amount" value="10000" />
      <input type="hidden" name="for" value="hacker" />
    </form>
    <script> document.forms[0].submit(); </script> 
  </body>
</html>
```

访问该页面后，表单会自动提交，相当于模拟用户完成了一次POST操作。

同样也会携带相应的用户 cookie 信息，让服务器误以为是一个正常的用户在操作，让各种恶意的操作变为可能。

#### 3.引诱用户点击链接

这种需要诱导用户去点击链接才会触发，这类的情况比如在论坛中发布照片，照片中嵌入了恶意链接，或者是以广告的形式去诱导，比如：

```html
 <a href="http://test.com/csrf/withdraw.php?amount=1000&for=hacker" taget="_blank">
  重磅消息！！！
  <a/>
```

点击后，自动发送 get 请求，接下来和`自动发 GET 请求`部分同理。

以上三种情况，就是CSRF攻击原理，跟XSS对比的话，<u>CSRF攻击并不需要将恶意代码注入HTML中，而是跳转新的页面，利用**服务器的验证漏洞**和**用户之前的登录状态**来模拟用户进行操作</u>

### 如何预防？防护策略

其实我们可以想到，黑客只能借助受害者的`cookie`骗取服务器的信任，但是黑客并不能凭借拿到**cookie**，也看不到 **cookie**的内容。另外，对于服务器返回的结果，由于浏览器**同源策略**的限制，黑客也无法进行解析。

> 这就告诉我们，我们要保护的对象是那些可以直接产生数据改变的服务，而对于读取数据的服务，则不需要进行`CSRF`的保护。而保护的关键，是 **在请求中放入黑客所不能伪造的信息**

**防范措施**：1. 验证码机制，2. 验证来源站点，3. 利用Cookie的`SameSite`属性，4.CSRF Token

#### 1.用户操作限制——验证码机制

方法：**添加验证码来识别是不是用户主动去发起这个请求**，由于一定强度的验证码机器无法识别，因此危险网站不能伪造一个完整的请求。

#### 2.验证来源站点

**在服务器端验证请求来源的站点**，由于大量的CSRF攻击来自第三方站点，因此服务器跨域禁止来自第三方站点的请求，主要通过HTTP请求头中的两个Header

- ***`Origin Header`***
- ***`Referer Header`***

这两个Header在浏览器发起请求时，大多数情况会自动带上，并且不能由前端自定义内容。

服务器可以通过解析这两个Header中的域名，确定请求的来源域。

其中，**`Origin`**只包含域名信息，而**`Referer`**包含了具体的 URL 路径。

在某些情况下，这两者都是可以伪造的，通过`AJAX`中自定义请求头即可，安全性略差。

#### 3.利用Cookie的`SameSite `属性

`SameSite`可以设置为三个值，`Strict`、`Lax`和`None`。

1. 在`Strict`模式下，浏览器完全禁止第三方请求携带Cookie。比如请求`sanyuan.com`网站只能在`sanyuan.com`域名当中请求才能携带 Cookie，在其他网站请求都不能。
2. 在`Lax`模式，就宽松一点了，但是只能在 `get方法提交表单`况或者`a标签发送get请求`的情况下可以携带 Cookie，其他情况均不能。
3. 在`None`模式下，Cookie将在所有上下文中发送，即允许跨域发送。

#### 4.CSRF Token

前面讲到CSRF的另一个特征是，攻击者无法直接窃取到用户的信息（Cookie，Header，网站内容等），仅仅是冒用Cookie中的信息。

那么我们可以使用Token，在不涉及XSS的前提下，一般黑客很难拿到Token。

Token(令牌)做为Web领域验证身份是一个不错的选择，当然了，JWT有兴趣的也可以去了解一下。

**第一步:将CSRF Token输出到页面中**

> 首先，用户打开页面的时候，服务器需要给这个用户生成一个Token，该Token通过加密算法对数据进行加密，一般Token都包括随机字符串和时间戳的组合，显然在提交时Token不能再放在Cookie中了（XSS可能会获取Cookie），否则又会被攻击者冒用。因此，为了安全起见Token最好还是存在服务器的Session中，之后在每次页面加载时，使用JS遍历整个DOM树，**对于DOM中所有的a和form标签后加入Token**。这样可以解决大部分的请求，但是对于在页面加载之后动态生成的HTML代码，这种方法就没有作用，还需要程序员在编码时手动添加Token。

**第二步:页面提交的请求携带这个Token**

> 对于GET请求，Token将附在请求地址之后，这样URL 就变成 [http://url](https://link.zhihu.com/?target=http%3A//url)?csrftoken=tokenvalue。 而对于 POST 请求来说，要在 form 的最后加上： `<input type=”hidden” name=”csrftoken” value=”tokenvalue”/>` 这样，就把Token以参数的形式加入请求了。

**第三步：服务器验证Token是否正确**

> 当用户从客户端得到了Token，再次提交给服务器的时候，服务器需要判断Token的有效性，验证过程是先解密Token，对比加密字符串以及时间戳，如果加密字符串一致且时间未过期，那么这个Token就是有效的。

