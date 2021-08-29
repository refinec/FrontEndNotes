# 使用 application/x-www-form-urlencoded 格式

默认情况下，axios 将 JavaScript 对象序列化为 JSON。若要以application/x-www-form-urlencoded 格式发送数据，可以使用下列选项之一。

* 在浏览器中，你可以使用**URLSearchParams ** API 如下:

  注意 URLSearchParams 并不是所有的浏览器都支持(请参阅  [caniuse.com](http://www.caniuse.com/#feat=urlsearchparams)) ，但是有一个可用的[polyfill](https://github.com/WebReflection/url-search-params)(请确保填充全局环境)。

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

* 在 node.js 中，你可以使用如下的 **querystring**  模块:

  ```javascript
  const queryString = require('querystring');
  axios.post('http://something.com/', querystring.stringify({foo:'bar'}))
  ```

  或‘ url 模块’中的‘ **URLSearchParams**’:

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

  