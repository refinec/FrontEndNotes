# Cookie/Session/Token 的区别

## Cookie

> http 是无状态协议，浏览器和服务器不可能凭协议的实现辨别请求的上下文。于是 cookie 登场，既然协议本身不能分辨链接，那就在请求头部手动带着上下文信息吧。

### 设置方式

> cookie 都是依靠 **`set-cookie` 头设置**，且**不允许 JavaScript 设置**。

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

 **session 才是真正的“信息”，cookie 是容器**，里面装着 `PHPSESSID=298zf09hf012fh2;`(上面提到)，这就是一个 session ID。 session 可以理解为信息，而 session id 是获取信息的钥匙，通常是一串唯一的哈希码。

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

```http
Set-Cookie: session=eyJ0ZXN0IjoiaGV5In0=; path=/; expires=Tue, 23 Feb 2021 01:07:05 GMT; httponly
Set-Cookie: session.sig=QBoXofGvnXbVoA8dDmfD-GMMM6E; path=/; expires=Tue, 23 Feb 2021 01:07:05 GMT; httponly
```

session 的值 `eyJ0ZXN0IjoiaGV5In0=` 通过 base64 解码。即可得到 `{"test":"hey"}`，这就是所谓的“将 session 信息放到客户端”，因为 base64 编码并不是加密，这就跟明文传输没啥区别，所以**请不要在客户端 session 里放用户密码之类的机密信息**。

第二个值 **session.sig**，它是一个 27 字节的 SHA1 签名，用以校验 session 是否被篡改，是 cookie 安全的又一层保障。

### 服务器储存

session储存在服务器，那么` express-session` 就需要一个容器 store，它可以是内存、redis、mongoDB 等等等等，内存应该是最快的，但是重启程序就没了，redis 可以作为备选，用数据库存 session 的场景感觉不多。

**` express-session`在监测到客户端送来的 cookie 之后，可以从 cookie 获取 sessionID，再使用 id 在 store 中获取 session 信息，挂到 `req.session`，经过这个中间件，就能顺利地使用 req 中的 session。**

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

**本质上 token 的功能就是和 session id 一模一样**。你把 session id 说成 session token 也没什么问题（Wikipedia 里就写了这个别名）。

**其中的区别在于：**

**区别在于:**

* session id **一般**存在 cookie 里，自动带上；
* token **一般**是要你主动**放在请求中**，例如设置**请求头**的 `Authorization` 为 `bearer:<access_token>`。

**Token 在权限证明上真的很重要**，不可泄漏，谁拿到 token，谁就是“主人”。

## 两种登录规范

> `Authentication` 和 `Authorization` 前者是**身份验证**，后者是**授权**。在编写登录系统时，要先**验证**用户身份，设置登录状态，给用户发送 token 就是**授权**。

### JWT（JSON Web Token）

> **本质是一个Token，里面放着 session 信息**。使用客户端储存的方式(是**客户端储存 session 信息的一种**)，并且可以随你选择放在 **cookie** 或是手动添加在 **Authorization**！但是他就叫 token！

**（重要）JWT 有着自己的结构**：`Header.Payload.Signature`  通过`.`结合一起

* **Header**

  ```json
  {
    "alg": "HS256", // alg 是签名算法，HMAC、SHA256、RSA 等。然后将其 base64 编码。
    "typ": "JWT" // typ 说明 token 类型是 JWT
  }
  ```

* **Payload**

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

  **Payload 是放置 session 信息的位置**，最后也要将这些信息进行 base64 编码，结果和客户端储存的 session 信息差不多。

  JWT 有一些约定好的属性：

  - iss (issuer)：签发人
  - exp (expiration time)：过期时间
  - sub (subject)：主题
  - aud (audience)：受众
  - nbf (Not Before)：生效时间
  - iat (Issued At)：签发时间
  - jti (JWT ID)：编号

* **Signature（签名）**

  和 `session.sig` 一样是用于**防止篡改**，不过 JWT 把**签名**和**内容**组合到一起罢了。

  JWT 签名的**生成算法**是这样的：

  ```js
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
  ```

  使用 **Header 里 alg 的算法**和自己设定的密钥 **secret** 编码 `base64UrlEncode(header) + "." + base64UrlEncode(payload)`。可以通过 [jwt.io Debugger](https://jwt.io/#debugger-io) 形象地看到 JWT 的组成原理。

#### 使用

在验证用户，顺利登录后，会给用户返回 JWT。因为 JWT 的信息没有加密，所以别往里面放密码，同客户端储存的 cookie。

**用户访问需要授权的连接时，可以把 token 放在 cookie，也可以在请求头带上 `Authorization: Bearer <token>`。（手动放在请求头不受 CORS 限制，不怕 CSRF）**

这样可以用于自家登录，也可以用于**第三方登录**。**单点登录**也是 JWT 的常用领域。

**JWT 也因为信息储存在客户端造成无法让自己失效的问题，这算是 JWT 的一个缺点**。

### HTTP authentication

HTTP authentication 是一种标准化的校验方式，**不会使用 cookie 和 session 相关技术**。请求头带有 **`Authorization: Basic <credentials>`** 格式的授权字段。

其中 **credentials 就是 Base64 编码的用户名 + `:` + 密码（或 token）**，以后看到 Basic authentication，意识到就是**每次请求**都带上用户名密码就好了。

Basic authentication 大概比较适合 serverless，毕竟他没有运行着的内存，无法记录 session，直接每次都带上验证就完事了。

### OAuth 2.0

> OAuth 2.0也是用 token 授权的一种协议，它的特点是你可以在**有限范围内**使用别家接口，如第三方应用登录。

在 OAuth 2.0 中涉及三个角色：**用户**、**应用提供方**、**登录平台**。相互调用关系如下：

     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+

#### 微信举例

1. **应用提供方需要先在登录平台申请好 AppID 和 AppSecret。**

2. **获取 code**

   > 什么是授权临时票据（code）？ 
   >
   > 答：第三方通过 code 进行获取 `access_token` 的时候需要用到，code 的超时时间为 10 分钟，一个 code 只能成功换取一次 `access_token` 即失效。code 的临时性和一次保障了微信授权登录的安全性。第三方可通过使用 https 和 state 参数，进一步加强自身授权登录的安全性。

   在这一步中，**登录用户**先在**登录平台**进行身份校验。

   ```
   https://open.weixin.qq.com/connect/qrconnect?
   appid=APPID&
   redirect_uri=REDIRECT_URI&
   response_type=code&
   scope=SCOPE&
   state=STATE
   #wechat_redirect
   ```

   | 参数          | 是否必须 | 说明                                                         |
   | ------------- | -------- | ------------------------------------------------------------ |
   | appid         | 是       | 应用唯一标识                                                 |
   | redirect_uri  | 是       | 请使用 urlEncode 对链接进行处理                              |
   | response_type | 是       | 填 code                                                      |
   | scope         | 是       | 应用授权作用域，拥有多个作用域用逗号（,）分隔，网页应用目前仅填写 snsapi_login |
   | state         | 否       | 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止 csrf 攻击（跨站请求伪造攻击） |

   注意一下 **scope** 是 OAuth2.0 权限控制的特点，定义了这个 code 换取的 token 可以用于什么接口。

   正确配置参数后，打开这个页面看到的是授权页面，在**用户**授权成功后，**登录平台**会带着 code 跳转到**应用提供方**指定的 `redirect_uri`：

   ```
   redirect_uri?code=CODE&state=STATE
   ```

   授权失败时，跳转到

   ```
   redirect_uri?state=STATE
   ```

   也就是失败时没 code。

3. **获取 token**

   > 在跳转到重定向 URI 之后，应用提供方的**后台**需要使用微信给你的**code**获取 token，同时，你也可以用传回来的 state 进行来源校验。

   要获取 token，**传入正确参数访问这个接口**：

   ```
   https://api.weixin.qq.com/sns/oauth2/access_token?
   appid=APPID&
   secret=SECRET&
   code=CODE&
   grant_type=authorization_code
   ```

   | 参数       | 是否必须 | 说明                                                         |
   | ---------- | -------- | ------------------------------------------------------------ |
   | appid      | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得           |
   | secret     | 是       | 应用密钥 AppSecret，在微信开放平台提交应用审核通过后获得     |
   | code       | 是       | 填写第一步获取的 code 参数                                   |
   | grant_type | 是       | 填 authorization_code，是其中一种授权模式，微信现在只支持这一种 |

   **正确的返回：**

   ```json
   {
     "access_token": "ACCESS_TOKEN",
     "expires_in": 7200,
     "refresh_token": "REFRESH_TOKEN",
     "openid": "OPENID",
     "scope": "SCOPE",
     "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
   }
   ```

   得到 token 之后你就可以根据之前申请 code 填写的 scope 调用接口了

4. **使用 token 调用微信接口**

   | 授权作用域（scope） | 接口                      | 接口说明                                                    |
   | ------------------- | ------------------------- | ----------------------------------------------------------- |
   | snsapi_base         | /sns/oauth2/access_token  | 通过 code 换取 `access_token`、refresh_token 和已授权 scope |
   | snsapi_base         | /sns/oauth2/refresh_token | 刷新或续期 `access_token` 使用                              |
   | snsapi_base         | /sns/auth                 | 检查 `access_token` 有效性                                  |
   | snsapi_userinfo     | /sns/userinfo             | 获取用户个人信息                                            |

   例如获取个人信息就是 `GET` [https://api.weixin.qq.com/sns...](https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN)

   在微信 OAuth 2.0，`access_token` 使用 query 传输，而不是上面提到的 Authorization。

   而使用 Authorization 的例子，如 GitHub 的授权，前面的步骤基本一致，在获取 token 后，这样请求接口：

   ```
   curl -H "Authorization: token OAUTH-TOKEN" https://api.github.com
   ```

   说回微信的 userinfo 接口，返回的数据格式如下：

   ```json
   {
     "openid": "OPENID",
     "nickname": "NICKNAME",
     "sex": 1,
     "province":"PROVINCE",
     "city":"CITY",
     "country":"COUNTRY",
     "headimgurl":"https://thirdwx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46",
     "privilege":[ "PRIVILEGE1" "PRIVILEGE2" ],
     "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
   }
   ```

5. **后续使用**

   在使用 token 获取用户个人信息后，你可以接着用 userinfo 接口返回的 openid，结合 session 技术实现在自己服务器登录。

   ```js
   // 登录
   req.session.id = openid
   if (req.session.id) {
     //   已登录
   } else {
     //   未登录
   }
   // 退出
   req.session.id = null
   // 清除 session
   ```

   **总结一下 OAuth2.0 的流程和重点**：

   1. 为自己的应用申请 ID 和 Secret
   2. 准备好重定向接口
   3. 正确传参获取 code **<- 重要**
   4. code 传入你的重定向接口
   5. 在重定向接口中使用 code 获取 token **<- 重要**
   6. 传入 token 使用微信接口

   OAuth2.0 着重于第三方登录和权限限制。而且 OAuth2.0 不止微信使用的这一种授权方式，其他方式可以看阮老师的[OAuth 2.0 的四种方式](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)。

## 其他方法

JWT 和 OAuth2.0 都是成体系的鉴权方法。

简单登录系统其实就以上面两种 session 储存方式为基础就能做到。

1. 使用服务器储存 session 为基础，可以用类似 `req.session.isLogin = true` 的方法标志该 session 的状态为已登录。
2. 使用客户端储存 session 为基础，设置 session 的过期日期和登录人就基本能用了。

```json
{
  "exp": 1614088104313,
  "usr": "admin"
}
```

**使用上面的知识自己写一个 express 的登录系统**：

- 初始化一个 store，内存、redis、数据库都可以
- 在用户身份验证成功后，随机生成一串哈希码作为 token
- 用 set-cookie 写到客户端
- 再在服务器写入登录状态，以内存为例就是在 store 中添加哈希码作为属性
- 下次请求带着 cookie 的话检查 cookie 带来的 token 是否已经写入 store 中即可

```js
let store = {}

// 登录成功后
store[HASH] = true
cookie.set('token', HASH)

// 需要鉴权的请求钟
const hash = cookie.get('token')
if (store[hash]) {
  // 已登录
} else {
  // 未登录
}

// 退出
const hash = cookie.get('token')
delete store[hash]
```

## 总结

- cookie 是储存 session/session id/token 的容器
- cookie 设置一般通过 `set-cookie` 请求头设置
- session 信息可以存放在浏览器，也可以存放在服务器
- session 存放在服务器时，以 session id 为钥匙获取信息
- token/session/session id 三者的界限是模糊的
- 一般新技术使用 token，传统技术使用 session id
- cookie/token/session/session id 都是用于鉴权的实用技术
- JWT 是浏览器储存 session 的一种
- JWT 常用于单点登录（SSO）
- OAuth2.0 的 token 不是由应用端颁发，存在另外的授权服务器
- OAuth2.0 常用于第三方应用登录

















