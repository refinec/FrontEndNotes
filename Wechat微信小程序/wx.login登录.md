## `wx.login`

> 调用接口获取登录凭证（`code`）。通过(`code`)凭证进而换取用户登录态信息，包括用户在当前小程序的唯一标识（`openid`）、微信开放平台帐号下的唯一标识（`unionid`，若当前小程序已绑定到微信开放平台帐号）及本次登录的会话密钥（`session_key`）等。用户数据的加解密通讯需要依赖会话密钥完成。

```js
wx.login({
  timeout: xx, // 单位ms
  success(res){
   // code用户登录凭证（有效期五分钟）。
   // 开发者需要在开发者服务器后台调用 code2Session，使用 code 换取 openid、unionid、session_key 等信息
    if (res.code) {
      wx.request({
        url: 'https://example.com/onLogin',
        data: {
          code: res.code
        }
      })
    } else {
      console.log('登录失败！' + res.errMsg)
    }
  },
  fail(){
    
  },
  complete(){
    
  }
})
```

