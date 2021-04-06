#### express-session 的使用

```javascript
// 配置一定在app.use(router)之前
// 当把这个插件配置好之后，就可通过req.session 来访问和设置 session 成员
// 添加 session 数据：req.session.foo = 'bar'
// 访问 session 数据：req.session.foo
app.use(session({
    // 配置加密字符串，它会在原有加密基础上和资格字符串拼起来去加密
    // 目的是为了增加安全性，防止客户端恶意伪造
    secret: 'keyboard cat', // 作为服务端生成session的签名
    name:'key', // 返回客户端的key的名称，默认未connect.sid，也可自己设置
    resave: false,
    saveUninitialized: true, //无论是否使用session，都默认直接分配一把钥匙
	cookie: { // 返回给前端key的属性，默认值为{ path:'/', httpOnly:true,secure:false,maxAge:null}
        maxAge: 66666,
        secure: true // https这样的情况才可以访问cookie
    }
    rolling:false, //在每次请求时强行设置cookie，这将重置cookie过期时间，默认false
}));

// 1.配置好后，通过 req.session 来访问和设置 Session 成员
req.session.foo = 'bar'; // 添加 Session 数据
req.session.foo; // 访问 Session 数据

// 2.注册成功，使用 Session 记录用户的登陆状态
req.session.isLogin = true;
req.session.user = user;

// 3.登出清除登录状态
app.get('/logout', function(req, res){
    // 清除登录状态
    req.session.user = null;
    
    //重定向到登录页
    res.redirect('/login');
})

// 4.对密码进行 md5 重复加密
body.password = md5(md5(body.password) + 'cat')
new User(body).save(function (err, user){
    if(err){
        return res.status(500).json({
            err_code: 500,
            message: 'Internal error.'
        })
    }
})

```

##### 1.安装使用

	* `npm install express-session`
	* `var session = require('express-session')`

##### 2. session(options)

```javascript
app.set('trust proxy', 1); //设置服务器代理时使用
app.use(session({
    //函数调用以生成新的session ID。提供一个函数，该函数返回一个字符串，该字符串将用作会话 ID。如果     //您想在生成 ID 时使用附加到 req 的某个值，那么该函数将作为第一个参数提供 req
    genid: function(req) {
        return genuuid() // use UUIDs for session IDs
    },
    
    //在响应中设置的会话 ID cookie 的名称(并在请求中读取)。默认值是‘ connect.sid’.
    //注意：如果有多个应用程序在同一主机名上运行（这只是名称，即localhost或127.0.0.1；不同的方案和     //端口不会命名不同的主机名），则需要将会话cookie彼此分开。只需为每个应用程序设置不同的名称
    name:'connect.sid',
    
    //默认值为undefined。在设置安全 cookie 时信任反向代理(通过“ X-Forwarded-Proto”头)\
    //true:"X-Forwarded-Proto"头将被使用
    //false:只有在存在直接 TLS/SSL 连接的情况下，才会忽略所有报头，并认为连接是安全的
    //undefined:使用'trust proxy'设置
    proxy:undefined,
    
    //默认值为true。如果它没有实现 touch 方法，并且您的存储在存储会话上设置了过期日期，那么您可能需     //要 reswave: true。每次请求都重新设置session cookie，假设你的cookie是10分钟过期，每次请求
    //都会再设置10分钟
    resave:false,
    
    //注意: 当该选项设置为 true，但是 saveUninitialized 选项设置为 false 时，不会在具有未初始化     //会话的响应上设置 cookie。此选项仅在为请求加载现有会话时修改该行为
    //每个请求都重新设置一个 cookie，默认为 false。
    rolling:false,
    
    //默认值是true强制将“未初始化”的会话保存到存储区。当会话是新的但是没有被修改时，它是未初始化的。     //选择 false 对于实现登录会话、减少服务器存储的使用，或者在设置 cookie 之前遵守要求权限的法律非     //常有用。选择 false 还有助于解决客户机在没有会话的情况下发出多个并行请求的竞争条件。
    saveUninitialized:false,
    
    //对会话 ID cookie 进行签名的密码。这可以是单个秘密的字符串，也可以是多个秘密的数组。若提供了       //机密数组，则只有第1个元素将用于对会话 ID cookie 进行签名，而在验证请求中的签名时将考虑所有元
    //素，通过设置的 secret 字符串，来计算hash值并放在 cookie 中，使产生的signedCookie防篡改。
    secret:'',
    
    //会话存储实例，默认为新的 MemoryStore 实例，也可以使用 redis，mongodb 等。express 生态中都     //有相应模块的支持
    store:'',
    
    //控制撤销 req.session 的结果(通过 delete、设置为 null 等)
    //destory:当响应结束时，会话将被销毁(删除)
    //keep:存储中的会话将被保留，但是在请求期间所做的修改将被忽略并且不保存
    unset:'keep',
    
    // session ID cookie 的设置对象
    cookie{
        //指定Path Set-Cookie 的值，默认'/',表示域的根路径
        path: '/',
        // 默认true，但为 true 时要小心，因为顺从的客户机不允许客户机端 JavaScript 查看
        // document.cookie 中的 cookie
        httpOnly:true, 
        //默认false，若为true，则如果浏览器没有HTTPS连接，则兼容的客户端将来不会将cookie发送回服
        //务器。如果您的node.js位于代理之后并且正在使用secure: true，则需要在express中设置  
        //“trust proxy”。app.set('trust proxy', 1); 信任第一代理
        secure:false,
        //指定计算 Expires Set-Cookie 属性时要使用的数字(毫秒)。这是通过获取当前服务器时间并将   	     // maxAge 毫秒添加到值中来计算 Expires 日期时间来完成的。默认情况下，不设置最大年龄
        maxAge:null,

        //指定 Domain Set-Cookie 属性的值。默认情况下不设置域.
        //大多数客户机将认为 cookie 只应用于当前域。
        domain:'', 

        //指定日期对象作为 Expires Set-Cookie 属性的值。默认情况下没有设置过期
        //大多数客户机会认为这是一个“非持久性 cookie” ，并会在退出 web 浏览器应用程序的情况下删除
        //注意: 如果选项中同时设置 expires 和 maxAge，那么对象中定义的最后一个选项将被使用.
        // expires 选项不应该直接设置，而只使用 maxAge 选项
        expires:'', 

        //指定SameSite Set-Cookie属性的值
        //true会将SameSite属性设置为，Strict以严格执行相同的网站。
        //false不会设置SameSite属性。
        //'lax'会将SameSite属性设置为，以Lax实现宽松的同一网站实施。
        //'none'会将SameSite属性设置None为显式跨站点Cookie。
        //'strict'会将SameSite属性设置为，Strict以严格执行相同的网站。
        sameSite:false,

    }
}));
```

##### 3. 方法

* `req.session`

* `Session.regenerate(callback) `

  重新生成会话,调用该方法后将在 req.Session 中初始化一个新的 SID 和 Session 实例，并调用回调

  ```javascript
  req.session.regenerate(function(err) {
    // will have a new session here
  })
  ```

* `Session.destroy(callback) `

  销毁会话并取消 req.session 属性。完成后，将调用回调

  ```javascript
  req.session.destroy(function(err) {
    // 无法访问此处的会话
  })
  ```

* `Session.reload(callback)`

  从存储中重新加载会话数据并重新填充 req.session 对象

  ```javascript
  req.session.reload(function(err) {
    // session updated
  })
  ```

* `Session.save(callback)`

  将会话保存回存储区，用内存中的内容替换存储区中的内容.如果会话数据已被更改，则在 HTTP 响应结束时自动调用此方法(尽管可以使用中间件构造函数中的各种选项更改此行为)。因此，通常不需要调用此方法。在某些情况下，调用这个方法是有用的，例如，重定向、长期存活的请求或在 websocket 中

  ```javascript
  req.session.save(function(err) {
    // session saved
  })
  ```

* `Session.touch()`

  更新.maxAge 属性。通常不需要调用这个属性，因为会话中间件会为您这样做

* `req.session.id`

  每个会话都有一个与之关联的惟一 ID。此属性是 req.sessionID 的别名，不能修改。添加它是为了使会话 ID 可以从会话对象访问

* `req.session.cookie`

  每个会话都有一个惟一的 cookie 对象伴随它。这允许您为每个访问者更改会话 cookie。例如，我们可以将 req.session.cookie.expires 设置为 false，以使 cookie 只在 user-agent 的持续时间内保留.

* `Cookie.maxAge/req.session.cookie.maxAge`

  返回以毫秒为单位的剩余时间，我们还可以重新分配一个新值来调整:

  ```javascript
  #以下几点基本上是等价的
  var hour = 3600000
  req.session.cookie.expires = new Date(Date.now() + hour)
  req.session.cookie.maxAge = hour
  ```

  当 maxAge 设置为60000(一分钟) ，并且运行了30秒后，它将返回30000，直到当前请求完成，此时 req.session.touch ()被调用以将 req.session.cookie.maxAge 重置为其原始值:

  ```javascript
  req.session.cookie.maxAge // => 30000
  ```

* `Cookie.originalMaxAge`

  Originalmaxage 属性返回会话 cookie 的原始 maxAge (生存时间) ，以毫秒为单位

* `req.sessionID`

  获取已加载会话的 ID,这只是在加载/创建会话时设置的只读值。

* `store.all(callback)`

  此可选方法用于将存储中的所有会话作为数组获取。这个callback应被调用为callback(error，session)

* `store.destroy(sid, callback) `

  用于在给定会话 ID (sid)的情况下从存储中销毁/删除会话。一旦会话被销毁，callback(error)被调用

* `store.clear(callback)`

  用于删除存储区中的所有会话。一旦存储被清除，callback(error)被调用

* `store.length(callback) `

  此可选方法用于获取存储中所有会话的计数,callback(error, len)

* `store.get(sid, callback)`

  用于从给定会话 ID (sid)的存储中获取会话,callback(error, session)

  如果找到session参数，则该session参数应为会话，否则如果找不到会话(并且没有错误) ，则为 null 或undefined。当 error.code = = ‘ ENOENT’起到callback(null，null)的作用时，需要进行特殊处理。

* `store.set(sid, session, callback)`

  在给定会话 ID (sid)和会话(session)对象的情况下将会话更新到存储中。一旦在存储中设置了会话，就调用 callback (error)。

* `store.touch(sid, session, callback)`

  用于“touch”给定session ID (sid)和会话(session)对象，调用回调callback(error)。

  这主要用于当存储将自动删除空闲会话时，并且此方法用于向存储发出给定会话处于活动状态的信号，可能会重置空闲计时器

##### 4. Debugging 调试

此模块在内部使用调试模块来记录有关会话操作的信息。要查看所有内部日志，请在启动应用程序时将 DEBUG 环境变量设置为 express-session (本例中为 npm start) :

```javascript
$ DEBUG=express-session npm start
```

在 Windows 上，使用相应的命令

```javascript
> set DEBUG=express-session & npm start
```

##### 5. 存储方式

session 的 store 有四个常用选项：1）内存 2）cookie 3）缓存 4）数据库

* 用 cookie session 的话，是不用担心状态共享问题的，因为 session 的 data 不是由服务器来保存，而是保存在用户浏览器端，每次用户访问时，都会主动带上他自己的信息。当然在这里，安全性之类的，只要遵照最佳实践来，也是有保证 的。它的弊端是增大了数据量传输，利端是方便。
* 缓存方式是最常用的方式了，即快，又能共享状态。相比 cookie session 来说，当 session data 比较大的时候，可以节省网络传输。推荐使用。
* 数据库 session。除非你很熟悉这一块，知道自己要什么，否则还是老老实实用缓存吧

##### 6.**signedCookie**

假设我的服务器有个秘密字符串，是 this_is_my_secret_and_fuck_you_all，我为用户 cookie 的 dotcom_user 字段设置了个值 alsotang。cookie 本应是：`{dotcom_user: 'alsotang'}`

而如果我们签个名，比如把 dotcom_user 的值跟我的 secret_string 做个 sha1:`sha1('this_is_my_secret_and_fuck_you_all' + 'alsotang') === '4850a42e3bc0d39c978770392cbd8dc2923e3d1d'`

然后把 cookie 变成这样:

```text
{
  dotcom_user: 'alsotang',  'dotcom_user.sig': '4850a42e3bc0d39c978770392cbd8dc2923e3d1d',
}
```

##### 7.**session cookie**

初学者容易犯的一个错误是，忘记了 session_id 在 cookie 中的存储方式是 session cookie。即，当用户一关闭浏览器，浏览器 cookie 中的 session_id 字段就会消失。常见的场景就是在开发用户登陆状态保持。假如用户在之前登陆了你的网站，你在他对应的 session 中存了信息，当他关闭浏览器再次访问时，你还是不懂他是谁。所以我们要在 cookie 中，也保存一份关于用户身份的信息。

比如有这样一个用户:`{username: 'alsotang', age: 22, company: 'alibaba', location: 'hangzhou'}`

我们可以考虑把这四个字段的信息都存在 session 中，而在 cookie，我们用 signedCookies 来存个 username。

登陆的检验过程伪代码如下：

```javascript
if (req.session.user) {  // 获取 user 并进行下一步
  next()
} else if (req.signedCookies['username']) {  // 如果存在则从数据库中获取这个 username 的信息，并保存到 session 中
  getuser(function (err, user) {
    req.session.user = user;
    next();
  });
} else {  // 当做为登陆用户处理
  next();
}
```



#### 2.cookie-parser的使用

##### 1.安装使用

* `npm install cookie-parse`
* `var cookie = require('cookie-parse')`

