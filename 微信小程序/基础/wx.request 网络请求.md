## HTTPS 网络请求

### `wx.request`详细参数

| **参数名** | **类型**      | **必填** | **默认值** | **描述**                                                     |
| :--------- | :------------ | :------- | :--------- | :----------------------------------------------------------- |
| url        | String        | 是       |            | 开发者服务器接口地址                                         |
| data       | Object/String | 否       |            | 请求的参数                                                   |
| header     | Object        | 否       |            | 设置请求的 header，header 中不能设置 `referer`，默认`header['content-type'] = 'application/json'` |
| method     | String        | 否       | GET        | （需大写）有效值： `GET`, `POST`,`PUT`, `DELETE`, `OPTIONS`, `HEAD`, `TRACE`, `CONNECT` |
| dataType   | String        | 否       | json       | 响应数据的内容格式，如果设为`json`，会尝试对返回的数据做一次 JSON解析 |
| success    | Function      | 否       |            | 收到开发者服务成功返回的回调函数，其参数是一个Object         |
| fail       | Function      | 否       |            | 接口调用失败的回调函数                                       |
| complete   | Function      | 否       |            | 接口调用结束的回调函数（调用成功、失败都会执行）             |

**success返回参数：**

| **参数名** | **类型**      | **描述**                                |
| :--------- | :------------ | :-------------------------------------- |
| data       | Object/String | 开发者服务器返回的数据                  |
| statusCode | Number        | 开发者服务器返回的 HTTP 状态码          |
| header     | Object        | 开发者服务器返回的 HTTP Response Header |

**在`app.json`指定`wx.requset`超时时间为3秒**

```javascript
{
  "networkTimeout": {
    "request": 3000
  }
}
```

#### 请求前后的状态处理

大部分场景可能是这样的，用户点击一个按钮，界面出现“加载中...”的Loading界面，然后发送一个请求到后台，后台返回成功直接进入下一个业务逻辑处理，后台返回失败或者网络异常等情况则显示一个“系统错误”的Toast，同时一开始的Loading界面会消失：

**为了防止用户极快速度触发两次tap回调，我们还加了一个hasClick的“锁”，在开始请求前检查是否已经发起过请求，如果没有才发起这次请求，等到请求返回之后再把锁的状态恢复回去。**

```javascript
var hasClick = false;
Page({
  tap: function() {
    if (hasClick) {
      return
    }
    hasClick = true
    wx.showLoading()
    wx.request({
      url: 'https://test.com/getinfo',
      method: 'POST',
      header: { 'content-type':'application/json' },
      data: { },
      success: function (res) {
        if (res.statusCode === 200) {
          console.log(res.data) // 响应数据
        }
      },
      fail: function (res) {
        wx.showToast({ title: '系统错误' })
      },
      complete: function (res) {
        wx.hideLoading()
        hasClick = false
      }
    })
  }
})
```

## 排查`wx.request`请求异常时的方法

在使用`wx.request`接口时我们会经常遇到 **无法发起请求** 或者 **服务器无法收到请求 **的情况，我们罗列排查这个问题的一般方法：

1. 检查手机网络状态以及wifi连接点是否工作正常。
2. 检查小程序是否为开发版或者体验版，因为**开发版**和**体验版**的小程序不会**校验域名**。
3. 检查对应请求的HTTPS证书是否有效，同时TLS的版本必须支持1.2及以上版本，可以在开发者工具的`console`面板输入**`showRequestInfo()`**查看相关信息。
4. 域名不要使用**IP地址**或者**`localhost`**，并且不能带**端口号**，同时域名需要经过ICP备案。
5. 检查`app.json`配置的超时时间配置是否太短，超时时间太短会导致还没收到回报就触发`fail`回调。
6. 检查发出去的请求是否`302`(重定向)到其他域名的接口，这种`302`的情况会被视为请求别的域名接口导致无法发起请求。

