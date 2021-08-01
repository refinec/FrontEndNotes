# 异步组件、Router、Vuex懒加载

## 异步组件懒加载

> 通过将`import`函数包装到箭头函数中，Vue仅在请求时执行它，并在那一刻加载模块。

```vue
<script>
    // 全局组件注册
    Vue.component("AsyncCmp", ()=> import("./AsyncCmp");
    // 或者
    Vue.component("AsyncCmp", resolve => {
     	require(['./AsyncCmp'], resolve)             
     })
	// 本地注册
    new Vue({
    	components:{
        	AsyncCmp: ()=> import("./AsyncCmp"),
            // 如果组件导入使用 命名的export，则可以在返回的Promise上使用对象分解。
            UiAlert: () => import('keen-ui').then(({ UiAlert }) => UiAlert)
        }              
    })
</script>
```

## Router懒加载

```javascript
// 代替：import Login from './login'
const Login = () => import("./login")
new VueRouter({
	routes:[{
		path:'/login',
		component: Login
	}]
})
```

## Vuex懒加载

Vuex具有**`registerModule`**使我们能够动态创建Vuex模块的方法。如果考虑到该`import`函数以ES模块作为有效负载返回了promise，则可以使用它来延迟加载模块：

```javascript
const store = new Vuex.Store();
// 假设这有个登录模块要加载
import('./store/login').then( loginModule =>{
    store.registerModule('login', loginModule)
})
```

