# Cookie/Session/Token 的区别

## Cookie

> http 是无状态协议，浏览器和服务器不可能凭协议的实现辨别请求的上下文。于是 cookie 登场，既然协议本身不能分辨链接，那就在请求头部手动带着上下文信息吧。

### 设置方式

> cookie 都是依靠 `set-cookie` 头设置，且不允许 JavaScript 设置。

```http
Set-Cookie: <cookie-name>=<cookie-value>
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
Set-Cookie: <cookie-name>=<cookie-value>; Secure
Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly

Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=None; Secure

// Multiple attributes are also possible, for example:
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```

- Expires 设置 cookie 的过期时间（时间戳），这个时间是**客户端时间**。
- Max-Age 设置 cookie 的保留时长（秒数），同时存在 Expires 和 Max-Age 的话，Max-Age 优先
- Domain 设置生效的域名，默认就是当前域名，不包含子域名
- Path 设置生效路径，`/` 全匹配
- Secure 设置 cookie 只在 https 下发送，防止**中间人攻击**
- HttpOnly 设置禁止 JavaScript 访问 cookie，防止**XSS**
- SameSite 设置跨域时不携带 cookie，防止**CSRF**

**Secure 和 HttpOnly 是强烈建议开启的。**

### 发送方式

**cookie 的发送格式如下:**

```http
Cookie: <cookie-list>
Cookie: name=value
Cookie: name=value; name2=value2; name3=value3
Cookie: PHPSESSID=298zf09hf012fh2; csrftoken=u32t4o3tb3gg43; _gat=1
```

在发送 cookie 时，并不会传上面提到的配置到服务器，因为服务器在设置后就不需要关心这些信息了，只要现代浏览器运作正常，收到的 cookie 就是没问题的。

## Session

 session 才是真正的“信息”，cookie 是容器，里面装着 `PHPSESSID=298zf09hf012fh2;`(上面提到)，这就是一个 session ID。 session 可以理解为信息，而 session id 是获取信息的钥匙，通常是一串唯一的哈希码。

**session 信息可以储存在客户端，如 [cookie-session](https://github.com/expressjs/cookie-session)，也可以储存在服务器，如 [express-session](https://github.com/expressjs/session)。使用 session ID 就是把 session 放在服务器里，用 cookie 里的 id 寻找服务器的信息。**

### 客户端储存

对于 cookie-session 库，其实就是把所有信息加密后塞到 cookie 里。其中涉及到 [cookies](https://github.com/pillarjs/cookies/blob/master/index.js) 库。

在设置 session 时其实就是调用 cookies.set，把信息写到 set-cookie 里，再返回浏览器。**换言之，取值和赋值的本质都是操作 cookie**。

浏览器在接收到 set-cookie 头后，会把信息写到 cookie 里。在下次发送请求时，信息又通过 cookie 原样带回来，所以服务器什么东西都不用存，只负责获取和处理 cookie 里的信息，这种实现方法不需要 session ID。

```js
const express = require('express')
var cookieSession = require('cookie-session')
const app = express()
app.use(
  cookieSession({
    name: 'session',
    keys: [
      /* secret keys */
      'key',
    ],
    // Cookie Options
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
  })
)
app.get('/', function(req, res) {
  req.session.test = 'hey'
  res.json({
    wow: 'crazy',
  })
})

app.listen(3001)
```

在通过 `app.use(cookieSession())` 使用中间件之前，请求是不会设置 cookie 的，添加后再访问（并且在设置 req.session 后，若不添加 session 信息就没必要写、也没内容写到 cookie 里），就能看到服务器响应头部新增了下面两行，分别写入 session 和 session.sig：

```
Set-Cookie: session=eyJ0ZXN0IjoiaGV5In0=; path=/; expires=Tue, 23 Feb 2021 01:07:05 GMT; httponly
Set-Cookie: session.sig=QBoXofGvnXbVoA8dDmfD-GMMM6E; path=/; expires=Tue, 23 Feb 2021 01:07:05 GMT; httponly
```

session 的值 `eyJ0ZXN0IjoiaGV5In0=` 通过 base64 解码。即可得到 `{"test":"hey"}`，这就是所谓的“将 session 信息放到客户端”，因为 base64 编码并不是加密，这就跟明文传输没啥区别，所以**请不要在客户端 session 里放用户密码之类的机密信息**。

第二个值 **session.sig**，它是一个 27 字节的 SHA1 签名，用以校验 session 是否被篡改，是 cookie 安全的又一层保障。

### 服务器储存

session储存在服务器，那么` express-session` 就需要一个容器 store，它可以是内存、redis、mongoDB 等等等等，内存应该是最快的，但是重启程序就没了，redis 可以作为备选，用数据库存 session 的场景感觉不多。

` express-session`在监测到客户端送来的 cookie 之后，可以从 cookie 获取 sessionID，再使用 id 在 store 中获取 session 信息，挂到 `req.session`，经过这个中间件，就能顺利地使用 req 中的 session。

在请求没有 session id 的情况下，` express-session`通过 `store.generate` 创建新的 session，在你写 session 的时候，cookie 可以不改变，只要根据原来的 cookie 访问内存里的 session 信息就可以了。

```js
var express = require('express')
var parseurl = require('parseurl')
var session = require('express-session')

var app = express()

app.use(
  session({
    secret: 'keyboard cat',
    resave: false,
    saveUninitialized: true,
  })
)

app.use(function(req, res, next) {
  if (!req.session.views) {
    req.session.views = {}
  }

  // get the url pathname
  var pathname = parseurl(req).pathname

  // count the views
  req.session.views[pathname] = (req.session.views[pathname] || 0) + 1

  next()
})

app.get('/foo', function(req, res, next) {
  res.json({
    session: req.session,
  })
})

app.get('/bar', function(req, res, next) {
  res.send('you viewed this page ' + req.session.views['/bar'] + ' times')
})

app.listen(3001)
```

### 两种储存方式的对比

储存在客户端的情况，解放了服务器存放 session 的内存，但是每次都带上一堆 base64 处理的 session 信息，如果量大的话传输就会很缓慢。

储存在服务器相反，用服务器的内存拯救了带宽。

另外，在退出登录的实现和结果，也是有区别的。

储存在服务器的情况就很简单，如果 `req.session.isLogin = true` 是登录，那么 `req.session.isLogin = false` 就是退出。

但是状态存放在客户端要做到真正的“即时退出登录”就很困难了。你可以在 session 信息里加上过期日期，也可以直接依靠 cookie 的过期日期，过期之后，就当是退出了。

但是如果你不想等到 session 过期，现在就想退出登录！怎么办？认真想想你会发现，仅仅依靠客户端储存的 session 信息真的没有办法做到。

即使你通过 `req.session = null` 删掉客户端 cookie，那也只是删掉了，但是如果有人曾经把 cookie 复制出来了，那他手上的 cookie 直到 session 信息里的过期时间前，都是有效的。

## Token

本质上 token 的功能就是和 session id 一模一样。你把 session id 说成 session token 也没什么问题（Wikipedia 里就写了这个别名）。

**其中的区别在于：**

session id **一般**存在 cookie 里，自动带上；token **一般**是要你主动放在请求中，例如设置请求头的 `Authorization` 为 `bearer:<access_token>`。



















