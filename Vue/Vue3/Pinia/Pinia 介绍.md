> Pinia主要用于代替vuex

[Pinia.js](https://pinia.vuejs.org/) 有如下特点：

- 完整的 ts 的支持；

- 足够轻量，压缩后的体积只有1kb左右;
- 去除 mutations，只有 state，getters，actions；
- actions 支持同步和异步；
- 代码扁平化没有模块嵌套，只有 store 的概念，store 之间可以自由使用，每一个store都是独立的
- 无需手动添加 store，store 一旦创建便会自动添加；
- 支持Vue3 和 Vue2

**安装**

```
yarn add pinia
 
npm install pinia
```

**引入注册**

Vue3

```js
import { createApp } from 'vue'
import App from './App.vue'
import {createPinia} from 'pinia'
 
const store = createPinia()
let app = createApp(App)
 
 
app.use(store)
 
app.mount('#app')
```

Vue2

```js
import { createPinia, PiniaVuePlugin } from 'pinia'
 
Vue.use(PiniaVuePlugin)
const pinia = createPinia()
 
new Vue({
  el: '#app',
  // other options...
  // ...
  // note the same `pinia` instance can be used across multiple Vue apps on
  // the same page
  pinia,
})
```

