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

## 第四种：直接调用

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

