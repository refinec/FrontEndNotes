# axios在模块中的封装

## 第一种

**模块中request.js：**

```javascript
import axios from 'axios'
export function request(config, success, fail){
	axios({
		url:config
	}).then(res=>{
		success(res)
	}).catch(err=>{
		fail(err)
	})
}
```

**调用者位置：**

```javascript
import {request} from './request'
request('http://....', res=>{
    console.log(res)
},err=>{
    console.log(err)
})
```

## 第二种

**模块中request.js：**

```javascript
export function request(config){
    axios.defaults.baseURL="http://localhost:8080/student";
    axios(config.url).then(res=>{
        config.sucess(res)
    }).catch(err=>{
        config.fail(err)
    })
}
```

**调用者位置：**

```javascript
import {request} from './request'
request({
    url:'getAl',
    success:res=>{
        console.log(res)
    },
    fail:err=>{
        console.log(err)
    }
})
```

## 第三种（Promise）

**模块中request.js：**

```javascript
export function request(config){
    return new Promise((resolve, reject)=>{
        let newVar = axios.create({
            baseUrl:'http://...',
            timeout:5000
        });
        newVar(config).then(res=>{
            resolve(res)
        }).catch(err=>{
            reject(err)
        })
    })
}
```

**调用者位置：**

```javascript
import {request} from './request'
request({
    url:'getAl'
}).then(res=>{
    console.log(res)
}).catch(err=>{
    console.log(err)
})
```

## 第四种：async

```js
# contactApi.js
const CONTACT_API = {
    getContactList: {
        method: 'get',
        url: '/list'
    }
}

# service/http.js
import axios from 'axios'
import service from './contactApi'

// service 循环遍历输出不同的请求方法
let instance = axios.create({
    baseURL: 'http://localhost:9000/api',
    timeout: 1000,
})

const Http = {}; // 包裹请求方法的容器
for (const key in service) {
    let api = service[key]; // url method
    // async 作用：避免进入回调地狱
    Http[key] = async function (
        params, // 请求参数 get: url, put, post, patch(data), delete: url
        config = {}, // 配置参数
        isFormData = false // 标识是否是form-data 请求
    ) {
        let newParams = {};

        // content-type是否是form-data的判断
        if (params && isFormData) {
            newParams = new FormData();
            for (const i in params) {
                newParams.append(i, params[i])
            }
        } else {
            newParams = params;
        }

        // 不同请求的判断
        let response = {}; // 请求的返回值
        if (api.method === 'put' || api.method === 'post' || api.method === 'patch') {
            try {
               response =  await instance[api.method](api.url, newParams, config)
            } catch (error) {
                response = error;
            }
        } else if (api.method === 'delete' || api.method === 'get') {
            config.params = newParams;
            try {
                response = await instance[api.method](api.url, config);
            } catch (error) {
                response = error;
            }
        }
        return response;
    }
}

instance.interceptors.request.use(config => {
    // 请求前
    
    /* 1.开启显示 loading */

    return config;
}, error => {
    // 请求错误

    /* 2.关闭显示 loading */
    /* 3.开启显示请求错误提示 */
});
instance.interceptors.response.use(response => {
    // 请求成功

    /* 2.关闭显示 loading */
    return response.data;
}, error => {
    /* 2.关闭显示 loading */
    /* 3.开启显示响应错误提示 */
});
export default Http;


# main.js
import Http from 'service/http'
Vue.prototype.$Http = Http;
```



## 第五种：直接调用

**模块中request.js：**

```javascript
export function request(config){
        let newVar = axios.create({
            baseUrl:'http://...',
            timeout:5000
        });
        return newVar(config);
}
```

**调用者位置：**

```javascript
import {request} from './request'
request({
    url:'getA'
}).then(res=>{
    console.log(res)
})
```

