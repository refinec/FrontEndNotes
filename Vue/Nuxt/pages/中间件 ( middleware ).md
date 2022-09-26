## middleware 定义

> 中间件允许您定义一个**自定义函数**运行在一个**页面**或一组页面**渲染之前**。

每一个中间件应放置在 **`middleware/`** 目录。**文件名称**将成为中间件名称 (**`middleware/auth.js`**将成为 **`auth`** 中间件)。

一个中间件接收 [context](https://www.nuxtjs.cn/api#上下文对象) 作为第一个参数：

```javascript
export default function (context) {  // context：当前页面的上下文对象
  context.userAgent = process.server
    ? context.req.headers['user-agent']
    : navigator.userAgent
}
```

## 中间件执行流程顺序

1. `nuxt.config.js`
2. 匹配布局
3. 匹配页面

### 异步执行

中间件可以**异步执行**,只需要返回一个 `Promise` 或使用第 2 个 `callback` 作为第一个参数：

```js
// middleware/stats.js
import axios from 'axios'

export default function ({ route }) {
  return axios.post('http://my-stats-api.com', {
    url: route.fullPath
  })
}
```

然后在你的 **`nuxt.config.js`** 、 **layouts** 或者 **pages** 中使用中间件:

```js
// nuxt.config.js
module.exports = {
  router: {
    middleware: 'stats'
  }
}
```

**`stats`** 中间件将在每个路由改变时被调用

您也可以将 **middleware** 添加到**指定的布局**或者**页面**:

```js
// pages/index.vue 
// 或者
// layouts/default.vue
export default {
  middleware: 'stats'
}
```

如果你想看到一个使用中间件的真实例子，请参阅在 GitHub 上的[example-auth0](https://github.com/nuxt/example-auth0)。

## 哪些地方可添加`middleware`?

- **`nuxt.config.js`**
- **Layout布局**
- **Pages页面**