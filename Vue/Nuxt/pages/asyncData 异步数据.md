## asyncData 方法

> `asyncData` 的方法，使得我们可以在设置组件的数据之前能**异步获取**或**处理数据**。

`asyncData`方法会在组件（**限于页面组件**）每次加载之前被调用。它可以在服务端或路由更新之前被调用。

在这个方法被调用的时候，第一个参数被设定为当前页面的[上下文对象](https://www.nuxtjs.cn/api#上下文对象)，你可以利用 `asyncData`方法来获取数据，Nuxt.js 会将 `asyncData` 返回的数据融合组件 `data` 方法返回的数据一并返回给当前组件。

> 注意：由于`asyncData`方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 `this` 来引用组件的实例对象。

### 使用方式

#### 1. Promise

> 返回一个 `Promise`, nuxt.js 会等待该`Promise`被解析之后才会设置组件的数据，从而渲染组件.

```js
export default {
  asyncData({ params }) {
    return axios.get(`https://my-api/posts/${params.id}`).then(res => {
      return { title: res.data.title }
    })
  }
}
```

#### 2. Async/await

```js
export default {
  async asyncData({ params }) {
    const { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
```

#### 3. callback 回调函数返回数据

> 除了return 返回数据，也可以使用`callback`

```js
export default {
  asyncData({ params }, callback) {
    axios.get(`https://my-api/posts/${params.id}`).then(res => {
      callback(null, { title: res.data.title })
    })
  }
}
```

### 数据的展示

`asyncData` 方法返回的数据在融合 `data` 方法返回的数据后，一并返回给模板进行展示，如：

```html
<template>
  <h1>{{ title }}</h1>
</template>
```

### context 上下文对象

context 变量的可用属性一览:

| 属性字段               | 类型                                                         | 可用            | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ |
| `app`                  | Vue 根实例                                                   | 客户端 & 服务端 | 包含所有插件的 Vue 根实例。例如：在使用 `axios` 的时候，你想获取 `$axios` 可以直接通过 `context.app.$axios` 来获取 |
| `isClient`             | `Boolean`                                                    | 客户端 & 服务端 | 是否来自客户端渲染（废弃。请使用 `process.client` ）         |
| `isServer`             | `Boolean`                                                    | 客户端 & 服务端 | 是否来自服务端渲染（废弃。请使用 `process.server` ）         |
| `isStatic`             | `Boolean`                                                    | 客户端 & 服务端 | 是否来自 `nuxt generate` 静态化（预渲染）（废弃。请使用 `process.static` ） |
| `isDev`                | `Boolean`                                                    | 客户端 & 服务端 | 是否是开发 dev 模式，在生产环境的数据缓存中用到              |
| `isHMR`                | `Boolean`                                                    | 客户端 & 服务端 | 是否是通过模块热替换 `webpack hot module replacement` (*仅在客户端以 dev 模式*) |
| `route`                | [Vue Router 路由](https://router.vuejs.org/zh/api/#路由对象属性) | 客户端 & 服务端 | Vue Router 路由实例                                          |
| `store`                | [Vuex 数据](https://vuex.vuejs.org/zh/api/)                  | 客户端 & 服务端 | `Vuex.Store` 实例。**只有[vuex 数据流](https://www.nuxtjs.cn/guide/vuex-store)存在相关配置时可用** |
| `env`                  | `Object`                                                     | 客户端 & 服务端 | `nuxt.config.js` 中配置的环境变量，见 [环境变量 api](https://www.nuxtjs.cn/api/configuration-env) |
| `params`               | `Object`                                                     | 客户端 & 服务端 | `route.params` 的别名                                        |
| `query`                | `Object`                                                     | 客户端 & 服务端 | `route.query` 的别名                                         |
| `req`                  | [`http.Request`](https://nodejs.org/api/http.html#http_class_http_incomingmessage) | 服务端          | Node.js API 的 Request 对象。如果 Nuxt 以中间件形式使用的话，这个对象就根据你所使用的框架而定。*`nuxt generate` 不可用* |
| `res`                  | [`http.Response`](https://nodejs.org/api/http.html#http_class_http_serverresponse) | 服务端          | Node.js API 的 Response 对象。如果 Nuxt 以中间件形式使用的话，这个对象就根据你所使用的框架而定。*`nuxt generate` 不可用* |
| `redirect`             | `Function`                                                   | 客户端 & 服务端 | 用这个方法重定向用户请求到另一个路由。状态码在服务端被使用，默认 302 `redirect([status,] path [, query])` |
| `error`                | `Function`                                                   | 客户端 & 服务端 | 用这个方法展示错误页：`error(params)` 。`params` 参数应该包含 `statusCode` 和 `message` 字段 |
| `nuxtState`            | `Object`                                                     | 客户端          | Nuxt 状态，在使用 `beforeNuxtRender` 之前，用于客户端获取 Nuxt 状态，仅在 `universal` 模式下可用 |
| `beforeNuxtRender(fn)` | `Function`                                                   | 服务端          | 使用此方法更新 `__NUXT__` 在客户端呈现的变量，`fn` 调用 (可以是异步) `{ Components, nuxtState }` ，参考 [示例](https://github.com/nuxt/nuxt.js/blob/cf6b0df45f678c5ac35535d49710c606ab34787d/test/fixtures/basic/pages/special-state.vue) |

#### 使用 `req`/`res`对象

在服务器端调用`asyncData`时，您可以访问用户请求的`req`和`res`对象。

```js
export default {
  async asyncData({ req, res }) {
    // 请检查您是否在服务器端
    // 使用 req 和 res
    if (process.server) {
      return { host: req.headers.host }
    }

    return {}
  }
}
```

#### 访问动态路由数据

您可以使用`注入`asyncData 属性的`context`对象来访问动态路由数据。

例如，可以使用配置它的文件或文件夹的名称访问动态路径参数。所以，如果你定义一个名为`_slug.vue`的文件，您可以通过`context.params.slug`来访问它。

```js
export default {
  async asyncData({ params }) {
    const slug = params.slug // When calling /abc the slug will be "abc"
    return { slug }
  }
}
```

#### 监听 query 参数改变

默认情况下，query 的改变不会调用`asyncData`方法。

如果要监听这个行为，例如，在构建分页组件时，您可以设置应通过页面组件的`watchQuery`属性监听参数。

#### 错误处理

通过调用 `error(params)` 方法来显示错误信息页面。`params.statusCode` 可用于指定服务端返回的请求状态码。

```js
export default {
  asyncData({ params, error }) {
    return axios
      .get(`https://my-api/posts/${params.id}`)
      .then(res => {
        return { title: res.data.title }
      })
      .catch(e => {
        error({ statusCode: 404, message: 'Post not found' })
      })
  }
}
```

如果你使用 `回调函数` 的方式, 你可以将错误的信息对象直接传给该回调函数， Nuxt.js 内部会自动调用 `error` 方法：

```js
export default {
  asyncData({ params }, callback) {
    axios
      .get(`https://my-api/posts/${params.id}`)
      .then(res => {
        callback(null, { title: res.data.title })
      })
      .catch(e => {
        callback({ statusCode: 404, message: 'Post not found' })
      })
  }
}
```
