## 一、`axios`配置默认值

1. **全局的 `axios `默认值**

   ```javascript
   axios.defaults.baseURL = 'http://api.example.com';
   axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
   axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'
   ```

2. **自定义实例默认值**

   ```javascript
   // 创建实例时设置配置的默认值
   var instance = axios.create({
       baseURL:'http://api.example.com'
   });
   // 在实例创建后修改默认值
   instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
   ```

3. **配置的优先顺序**

   配置会以一个优先顺序进行合并。这个顺序是：在` lib/defaults.js` 找到的库的默认值，然后是实例的 `defaults `属性，最后是请求的 `config `参数。后者将优先于前者。这里是一个例子：

   ```javascript
   // 使用由库提供的配置的默认值来创建实例
   // 此时超时配置的默认值是0
   var instance = axios.create();
   
   // 覆写库的超时默认值。
   // 在超时前，所有请求都会等待 2.5 秒
   instance.defaults.timeout = 2500;
   
   // 为已知需要花费很长时间的请求覆写超时设置
   instance.get('/longRequest',{
       timeout:5000
   })
   ```

## 二、`axios`配置选项

> **`axios(config)` **或 **`axios(url[,config])`**，只有**`url`**是必需的，若没有指定`method`，默认使用`get`请求。
>
> ```js
> // 发送 GET 请求（默认方法）
> axios('/user/123')
> 
> // 发送POST 请求
> axios({
>     method:'post',
>     url:'/user/123',
>     data:{
>         firstName:'f'
>     }
> });
> ```

### 1. 基础配置项

```json
{
    // url: 用于请求的服务器 URL
    url: '/user',
    
    // baseURL 将自动加在 url 前面，除非 url 是一个绝对 URL。
    // 它可以通过设置一个 baseURL 便于为 axios 实例的方法传递相对 URL
    baseURL: 'https://some-domain.com/api/',
    
    // method: 请求方法。默认是 get
    method: 'get',
    
    // params 是与请求一起发送的 URL 参数
    // 必须是一个：无格式对象(plain object) 或 URLSearchParams 对象
    params: {
        ID: 12345
    },
    
    // paramsSerializer 是一个负责 params 序列化的函数
    // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
    paramsSerializer: function(params) {
        return Qs.stringify(params, {arrayFormat: 'brackets'})
    },
    
    // data 是作为请求主体被发送的数据, 只适用于这3个请求方法： 'PUT'、 'POST'、 'PATCH'
    // 在没有设置 transformRequest 时，必须是以下类型之一：
    // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
    // - 浏览器专属：FormData, File, Blob
    // - Node 专属： Stream
    data: {
        firstName: 'Fred'
    },
}
```

### 2. `request`请求配置项

```json
{
    // timeout 指定请求超时的毫秒数(0 表示无超时时间)
    // 如果请求话费了超过 timeout 的时间，请求将被中断
    timeout: 3000,
    
    // headers 是被发送的自定义请求头
    headers: {'X-Requested-With': 'XMLHttpRequest'},
    
    // withCredentials 表示跨域请求时是否需要使用凭证, 默认 false
    withCredentials: false,
    
    // auth 表示使用 HTTP 基础验证，并提供凭据
    // 这将设置一个 Authorization 头，覆写掉现有的任意使用 headers 设置的自定义 Authorization 头
    auth: {
        username: 'janedoe',
        password: 's00pers3cret'
    },
    
    // xsrfCookieName 是用作 xsrf token 的值的 cookie 的名称
    xsrfCookieName: 'XSRF-TOKEN', // 默认的
    
    // xsrfHeaderName 是承载 xsrf token 的值的 HTTP 头的名称
    xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的
    
    // transformRequest 允许在向服务器发送前，修改请求数据。只能用在这3个请求方法：'PUT'、 'POST' 、 'PATCH' 
    // 后面数组中的函数必须返回一个：字符串、ArrayBuffer、Stream
    transformRequest: [function (data) {
        // 对 data 进行任意转换处理
        return data;
    }],
    
    // cancelToken 指定用于取消请求的 cancel token
	// （查看后面的 Cancellation 这节了解更多）
	cancelToken: new CancelToken(function (cancel) { })
}
```

### 3. `response `响应配置项

```json
{
    // responseType 表示服务器响应的数据类型，可以是:
    // - 'json'
    // - 'text'
    // - 'document'
    // - 'blob' (默认)
    // - 'arraybuffer'
    // - 'stream'
    responseType: 'json',
    
    // maxContentLength 定义允许的响应内容的最大尺寸
    maxContentLength: 2000,
    
    // validateStatus 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise。
    // 如果 validateStatus 返回 true (或者设置为 null 或 undefined)，promise 将被 resolve; 
    // 否则，promise 将被 reject
    validateStatus: function (status) {
        return status >= 200 && status < 300; // 默认的
    },
    
    // transformResponse 在传递给 then/catch 前，允许修改响应数据
    transformResponse: [function (data) {
        // 对 data 进行任意转换处理
        return data;
    }],
}
```

### 4. `upload`上传 /`download`下载进度配置项

```json
// onUploadProgress 允许为上传处理进度事件
onUploadProgress: function (progressEvent) {
    // 对原生进度事件的处理
},

// onDownloadProgress 允许为下载处理进度事件
onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
},
```

### 5. 其他配置项

```json
{
    // adapter 允许自定义处理请求，以使测试更轻松
    // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
    adapter: function (config) {
        /* ... */
    },
    
    // maxRedirects 定义在 node.js 中 follow 的最大重定向数目
    // 如果设置为0，将不会 follow 任何重定向
    maxRedirects: 5, // 默认的
    
    // httpAgent 和 httpsAgent 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。
    // 允许像这样配置选项：
    // keepAlive 默认没有启用
    httpAgent: new http.Agent({ keepAlive: true }),
    httpsAgent: new https.Agent({ keepAlive: true }),
    
    // proxy 定义代理服务器的主机名称和端口
    // auth 表示 HTTP 基础验证应当用于连接代理，并提供凭据
    // 这将会设置一个 Proxy-Authorization 头，覆写掉已有的通过使用 header 设置的自定义 Proxy-Authorization 头。
    proxy: {
        host: '127.0.0.1',
        port: 9000,
        auth: : {
        	username: 'mikeymike',
        	password: 'rapunz3l'
    	}
	},

}
```

## 三、请求方法

使用自定义配置新建一个 `axios `实例 ：`axios.create([config])`

```javascript
const instance = axios.create({
    baseURL:'http://api.example.com',
    timeout:1000,
    headers:{
        'X-custom-Header':'foo'
    }
})
```

**实例方法**

1. instance.**`request(config)` **
2. instance.**`get(url[, config]) `**
3. instance.**`delete(url[, config])`**
4. instance.**`head(url[, config])`**
5. instance.**`post(url[, data[, config]])`**
6. instance.**`put(url[, data[, config]])`**
7. instance.**`patch(url[, data[, config]])`**

> 在使用别名方法时，**url、method、data** 这些属性都不必在**`config配置`**中指定

### 1. **`instance.request(config)`**

### 2. **`instance.get(url[, config])`  **

```javascript
// 为给定 ID 的user 创建请求
instance.get('/user?ID=12345').then(function(res){
    console.log(res)
}).catch(function(err){
    console.log(err)
})
```

或者

```js
instance.get('/user',{
    params:{
        ID:12345
    }
}).then(function(res){
    console.log(res)
}).catch(function(err){
    console.log(err)
})
```

或者 使用 `async`

```javascript
async function getUser(){
    try{
        const res = await instance.get('/user?ID=123')
    }catch(err){
        console.error(err)
    }
}
```

### 3.**`instance.post(url[, data[, config]])`**

> 提交数据（表单提交+文件上传）

```javascript
instance.post('/user',{
    firstName:'f'
}).then((res)=>{
    ...
}).catch((err)=>{
    ...
})
```

### 4. **`instance.put(url[, data[, config]])`**

> 更新数据（所有数据推送到后端）

### 5. **`instance.patch(url[, data[, config]])`**

> 更新数据（只将修改的数据推送到后端）

### 6.**`instance.delete(url[, config])`**

> 删除数据

### 7. **`instance.head(url[, config])` **

## 四、请求数据使用 `application/x-www-form-urlencoded` 格式

默认情况下，axios 将 JavaScript 对象序列化为 JSON。若要以`application/x-www-form-urlencoded` 格式发送数据，可以使用下列选项之一。

* 在浏览器中，你可以使用**`URLSearchParams `** API 如下:

  注意 `URLSearchParams `并不是所有的浏览器都支持(请参阅  [caniuse.com](http://www.caniuse.com/#feat=urlsearchparams)) ，但是有一个可用的[polyfill](https://github.com/WebReflection/url-search-params)(请确保填充全局环境)。

  ```javascript
  const params = new URLSearchParams();
  params.append('params','value');
  axios.post('/foo', params);
  
  // 使用qs库对数据进行编码
  const qs = require('qs');
  axios.post('/foo', qs.stringify({ 'bar':123}))
  ```

  或以另一种方式(ES6)

  ```javascript
  import qs from 'qs';
  const data = { 'bar':123};
  const options = {
      method:'POST',
      headers:{
          'content-type':'application/x-www-form-urlencoded'
      },
      data: qs.stringify(data);
      url
  }
  axios(options)
  ```

* 在 node.js 中，你可以使用如下的 **`querystring`**  模块:

  ```javascript
  const queryString = require('querystring');
  axios.post('http://something.com/', querystring.stringify({foo:'bar'}))
  ```

  或 url 模块中的 **`URLSearchParams`**:

  ```javascript
  const url = require('url');
  const params = new url.URLSearchParams({foo:'bar'});
  axios.post('http://something.com/', params.toString())
  ```

  在 node.js 中，可以使用[form-data](https://github.com/form-data/form-data)库如下:

  ```javascript
  const FormData = require('form-data');
  
  const form = new FormData()；
  form.append('my_buffer', new Buffer(10));
  form.append('my_file', fs.createReadStream('1.jpg'))
  axios.post('http://something.com/', form, { headers: form.getHeaders() })
  ```

  或者使用拦截器:

  ```javascript
  axios.interceptors.request.use(config=>{
      if(config.data instanceof FormData) {
          Object.assign(config.headers, config.data.getHeaders())
      }
      return config;
  })
  ```


## 五、response响应结构

```json
{
    // 'data'由服务器提供的响应
    data:{},
    // 'status'来自服务器响应的 HTTP 状态码
    status:200,
    // 'statusText'来自服务器响应的HTTP状态信息
    statusText:'OK',
    //'headers'服务器响应的头
    headers:{}
    //'config'是为请求提供的配置信息
    config:{}
}
```

使用 `then `时，你将接收下面这样的响应：

```javascript
axios.get('/user/123').then((res)=>{
    console.log(res.data);
    console.log(res.status);
    console.log(res.statusText);
    console.log(res.headers);
    console.log(res.config);
})
```

在使用 `catch `时，或传递 [rejection callback](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 作为 then 的第二个参数时，**响应可以通过 `error `对象被使用**

