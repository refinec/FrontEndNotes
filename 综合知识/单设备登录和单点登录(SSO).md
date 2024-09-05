## 单设备登录

### 实现思路一

> WebSocket + Redis

1. 在后台socket里面创建两个Map，`sessionPool`和`sessionIds`，分别用来存放**客户端会话池**和**客户端会话标记**

2. 用户登录账号成功，进入应用首页，和服务端建立socket连接，将 `账号+"_"+UUID` 格式的字符串作为`state`参数的值

3. 在OnOpen连接时，以`state`为`key`，当前`session`为`value`存入`sessionPool`，以`sessionId`为`key`，`state`为`value`存入`sessionIds`

4. 以"`_`"分隔`state`成数组，取第一个元素即获取到当前登录的账号

5. 在缓存(Redis)里模糊查询含有该账号的`key`集合，如果存在，那么就取出对应的`value`值，其实这个`value`存的就是首先登陆这个账号的那个`state`，就可以根据这个`state`给先登录的账号设备推送消息并做`logout`的操作，并清除缓存

6. 把当前登录的`state`作为`key`和`value`存入缓存，失效时间设置与否都可以，如果设置的话需超过登录态失效的时长

```java
 /**
 * 连接时触发
 * @param state
 * @param session
 */
@OnOpen
public void onOpen(@PathParam(value = "state")String state,Session session) {
    this.session = session;
    sessionPool.put(state, session);
    sessionIds.put(session.getId(), state);     
    String[] arry = state.split("_");
    Set<String> keys = redisService.keys(arry[0] + "*");
    if (keys != null) {
        List<String> list = redisService.multiGet(keys);
        for (String value : list) {
            sendMessage("您的账号于"+ DateUtils.pageformat(DateUtils.getCurrentTime()) + "在另一台设备登录，如果这不是您的操作，那么您的登录密码已泄露，请尽快修改",value);
            redisService.delete(value);
        }
    }
    redisService.set(state,state,7200);
}

/**
 * 自定义发送消息的方法
 * @param message
 * @param state
 */
public static void sendMessage(String message,String state) {
    Session session = sessionPool.get(state);
    if (session != null) {
        try {
            session.getBasicRemote().sendText(message);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

前端js连接WebSocket方法

```javascript
function getUUID() {
  return 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
      var r = Math.random() * 16 | 0,
          v = (c === 'x' ? r : (r & 0x3 | 0x8));
      return v.toString(16);
  });
}

function loginSocket(userName) {
  var websocket = null;
  if ('WebSocket' in window) {
      websocket = new 		  WebSocket("ws://localhost:8080/mobile/socketServer/"+userName+"_"+getUUID());
  } else {
      alert('当前浏览器 不支持 websocket')
  }
  //连接成功建立的回调方法
  websocket.onopen = function () {
      console.log('websocket连接成功');
  };
  //连接发生错误的回调方法
  websocket.onerror = function () {
      console.log('websocket连接发生错误');
  };
  //接收到消息的回调方法
  websocket.onmessage = function (event) {
      $.getJSON("logout", function(r){
          console.log('logout：'+event.data);
      });
      confirm(event.data, {
          btn: ['确定'] //按钮
      }, function(){
          location.href = 'login.html';
      });
  };
  //连接关闭的回调方法
  websocket.onclose = function () {
      console.log("websocket连接关闭");
  };
  //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
  window.onbeforeunload = function () {
      websocket.close();
  };
}
```

### 实现思路二

用户首次登录时，**将用户信息存入Redis，key是用户id，value是token**。

当用户在其他设备登录时，会重新生成token，这个时候原先的token已经被覆盖了。

所以用户在访问需要登录账号的操作时，系统会拦截请求判断token是否存在。当然是不存在的，所以我们就实现了单个设备登录的需求。



## 单点登录(Single Sign On, 又称SSO)

> SSO 用于多个应用系统间，用户只需要登录一次就可以访问所有相互信任的应用系统。

- `HTTP` 是无状态协议，所以服务器单从网络连接上无从知道客户身份。 那要如何才能识别客户端呢？最常见的方案就是 `Cookie`。

- `Cookie` 是**客户端保存用户信息**的一种机制，保存在客户机硬盘上。可以由服务器响应报文`Set-Cookie`的首部字段信息或者客户端 `document.cookie`来设置，并随着每次请求发送到服务器。子域名可以获取父级域名 `Cookie`。

  cookie保存在HTTP 响应头的`set-cookie`字段和请求头的**`cookie`**字段

- `Session` 其实是一个抽象概念，**用于跟踪会话**，识别多次 HTTP 请求来自同一个客户端。`Cookie` 只是通用性较好的一种实现方案，通常是设置一个名为 `SessionID`（名称可自定义，便于描述，本文均使用此名称）的 `Cookie`，每次请求时携带该 `Cookie`，后台服务即可依赖此 `SessionID` 值识别客户端。

### 普通单系统登录

<img src="../assets/%E9%9D%A2%E8%AF%95/single system.jpeg" alt="登录流程" style="zoom:50%;" />

#### 后台是如何通过 SessionID 知道是哪个用户呢？

1. **数据库存储关联**：将 `SessionID` 与数据信息关联，存储在 `Redis`、`mysql` 等数据库中；
2. **数据加密直接存储**：比如 `JWT` 方式，用户数据直接从 `SessionID` 值解密出来（此方式时 `Cookie` 名称以 `Token` 居多）。

### 1.多系统登录

#### 要实现多系统登录，面临的问题？

##### 不同子域名

**<u>子域名间 Cookie 是不共享的，但各子域名均可获取到父级域名的 Cookie</u>**，即`app.demo.com`与`news.demo.com`均可以获取 `demo.com`域名下的 Cookie。

所以可以通过将 Cookie 设置在父级域名上，可以达到子域名共享的效果，即当用户在 `app.demo.com` 域名下登录时，在`demo.com`域名下设置名为 SessionID 的 Cookie，当用户之后访问`news.demo.com`时，后台服务也可以获取到该 SessionID，从而识别用户。

##### 完全不同域名

> **不同域名无法直接共享 Cookie **。

**前端跨域带 Cookie**，如果只是期望异步请求时获取当前用户的登录态，可以通过发送跨域请求到已经登录过的域名，并配置属性：

```
xhrFields: {
  withCredentials: true
}
```

这样可在请求时携带目标域名的 Cookie，目标域名的服务即可识别当前用户。

但是，这要求目标域名的接口支持 [CORS](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FAccess_control_CORS) 访问（出于安全考虑，CORS 开启 withCredentials 时，浏览器不支持使用通配符`*`，需明确设置可跨域访问的域名名单）。

如果只是为了规避浏览器的限制，实现与通配`*`同样的效果，到达所有域名都可以访问的目的，可根据访问的 `Referrer` 解析请求来源域名，作为可访问名单。但是出于安全考虑，不推荐使用，请设置明确的可访问域名。



#### CAS（Central Authentication Service）中央认证服务

> 使用CAS实现单点登录，它通过跳转中间域名的方式来实现登录

1. 所有的登录过程都依赖于 CAS 服务，包含用户登录页面、ST 生成、验证；
2. 为了保证 ST 的安全性，一般 ST 都是随机生成的，没有规律性。CAS 规定 ST 只能保留一定的时间，之后 CAS 服务会让它失效，而且，CAS 协议规定 ST 只能使用一次，无论 ST 验证是否成功，CAS 服务都会清除服务端缓存中的该 ST，从而规避同一个 ST 被使用两次或被窃取的

**关于令牌注意点**：

1. 授权令牌是一串随机字符，以什么样的方式生成都没有关系，只要不重复、不易伪造即可
2. 用户在sso认证中心登录成功后，认证中心创建授权令牌并存储该令牌。子系统会接收并存储sso认证中心发送的授权令牌，并且子系统的session ID 会与令牌绑定

![流程图](../assets/%E9%9D%A2%E8%AF%95/multi system.jpeg)

举例：普通网站 A.com、B.com；统一登录网站S.com,
未登录时候: 访问A.com, A站发现你没有登录, 就把网址重定向到S.com/?server=A.com, 此时浏览器在S站, S站发现没有登录,会让你输入 账号密码, 然后确认, 好, 此时S站已经登录了, S站会生成1个session-对应的浏览器生成一个cookie(S站的), 下次访问S站, 是用这个cookie,
S站存完cookie后, 又会重定向到A.com/?ticket=asdsadad, 到此,又回到了A站, A站还是没有登录状态, 接下来A站后台发现你传了ticket参数, 后台会拿着 ticket 和A站的域名(A.com) 这两个东西 去S站的后台去验证,这个是ticket是否有效,以及ticket这个人的身份信息, 如果正确, 则A站会生成一个session,给浏览器发放一个cookie(A站的), 此时A站完成登录状态, 浏览器会有2个不同的cookie (A站的, 和S站的).
B.com 免登效果:
然后用户访问B.com, B站(不是bilibili...) B站发现没有登录, 重定向到S.com/?server=B.com , 此时S站有cookie, S站发现用户已经登录, 不用输入密码了, 直接重定向到B.com/ticket=wqeipofvijwrpt, 接下和A站的流程一样了.... B站后台发现你传了ticket参数, 后台会拿着 ticket 和B站的域名(B.com) 这两个东西.....
用户只输入了B.com 会发现已经是登录状态

### 2. 多系统注销

> 在一个子系统中注销，所有子系统的会话都将被销毁，用下面的图来说明

![image.png](../assets/%E9%9D%A2%E8%AF%95/logout.jpg)

sso认证中心一直监听全局会话的状态，一旦全局会话销毁，监听器将通知所有注册系统执行注销操作

下面对上图简要说明:

1. 用户向系统1发起注销请求
2. 系统1根据用户与系统1建立的会话id拿到令牌，向sso认证中心发起注销请求
3. sso认证中心校验令牌有效，销毁全局会话，同时取出所有用此令牌注册的系统地址
4. sso认证中心向所有注册系统发起注销请求
5. 各注册系统接收sso认证中心的注销请求，销毁局部会话
6. sso认证中心引导用户至登录页面