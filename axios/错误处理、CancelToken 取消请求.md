# 错误处理、取消请求

## 错误处理

```javascript
axios.get('/user/123').catch(function(error){
    if(error.response){
        // 请求已发出，但服务器响应的状态码不在 2xx 范围内
        console.log(error.response.data)
        console.log(error.response.status)
        console.log(error.response.headers)
    }else{
        console.log(error.message)
    }
    console.log(error.config)
})
```

使用**`validateStatus `** 配置选项定义一个HTTP 状态码的错误范围

```javascript
axios.get('/user/123',{
    validateStatus:function(status){
        return status < 500; // 状态码大于或等于500才会 reject
    }
})
```

## 取消请求

使用 `CancelToken` 取消请求，可以使用同一个 `CancelToken` 取消多个请求

1. 使用 `CancelToken.source` 工厂方法创建 cancel token

   ```javascript
   var CancelToken = axios.CancelToken;
   var source = CancelToken.source();
   
   axios.get('/user/123',{
       cancelToken:source.token
   }).catch(function(thrown){
       if(axios.isCancel(thrown)){
           console.log(throw.message)
       }else{
           // 处理错误
       }
   });
   
   // 取消请求(message 参数可选)
   source.cancel('Operation canceled by the user')
   ```

2. 还可以通过传递一个 `executor `函数到 `CancelToken `的构造函数来创建 cancel token

   ```javascript
   var CancelToken = axios.CancelToken;
   var cancel;
   axios.get('/user/123', {
       cancelToken: new CancelToken(function executor(c){
           // executor 函数接收一个 cancel函数作为参数
           cancel = c;
       })
   });
   
   // 取消请求
   cancel();
   ```

   