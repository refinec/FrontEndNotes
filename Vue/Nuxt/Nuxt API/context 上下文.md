## [`context` 上下文对象](https://www.nuxtjs.cn/api/context)

> **` context`** 提供了一些额外的参数和对象
>
> 并且该对象仅在**`asyncData`**、**`fetch`**、**`plugins`**、**`middleware`**、**`nuxtServerInit`**生命周期中使用

### 通用的keys

> 下面这些key在客户端和服务端都可用

#### app

包含所有插件的 **Vue 根实例**。

#### store

Vuex.Store 实例。只有设置了vuex 时才可用(在src/store目录下，创建store文件)

#### route

Vue Router 路由实例

#### params

`route.params`的别名

#### query

`route.query`的别名

#### env

nuxt.config.js 中配置的环境变量

```js
export default {
  env: {
    baseUrl: process.env.BASE_URL || 'http://localhost:3000'
    // ...
  }
}
```

#### isDev

是否是开发 dev 模式，这对于在生产环境中缓存一些数据非常有用。

#### isHMR

是否是通过模块热替换 webpack hot module replacement (仅在客户端以 dev 模式)

#### redirect

`redirect ([ status，] path [ ，query ]).`

路由重定向，状态码在服务器端使用，默认为302。

```js
redirect(302, '/login')
redirect({ name: 'slug', params: { slug: mySlug } })
redirect('https://vuejs.org')
```

> 注意📢：由于hydration 错误，无法在客户端 Nuxt 插件中使用`redirect`或`error`（客户端内容与服务器的预期内容不同）。有效的解决方法是使用 `window.onNuxtReady(() => { window.$nuxt.$router.push('/your-route') })`

#### error

用这个方法展示错误页：`error(params)` 。`params` 参数应该包含 `statusCode` 和 `message` 字段

如：`error({ status: 404 })`

#### $config

指向`Runtime config`，包括

* [`publicRuntimeConfig`](https://nuxtjs.org/docs/configuration-glossary/configuration-runtime-config/#publicruntimeconfig)：在客户端和服务器使用 $config 访问此对象的值。
* [`privateRuntimeConfig`](https://nuxtjs.org/docs/configuration-glossary/configuration-runtime-config/#privateruntimeconfig)：仅服务器使用 $config 访问此对象的值。

### 仅服务端可用的 keys

#### req(request)

Node.js API 的 Request 对象。如果 Nuxt 以中间件形式使用的话，这个对象就根据你所使用的框架而定。`nuxt generate` 不可用

#### res(response)

Node.js API 的 Response 对象。如果 Nuxt 以中间件形式使用的话，这个对象就根据你所使用的框架而定。`nuxt generate` 不可用

#### beforenuxtrender(fn)

使用此方法更新 `__NUXT__` 在客户端呈现的变量，`fn` 调用 (可以是异步) `{ Components, nuxtState }` .

例如：

```vue
<template>
  <h1>Special state in `window.__NUXT__`</h1>
</template>

<script>
export default {
  fetch ({ isServer, beforeNuxtRender }) {
    if (isServer) {
      beforeNuxtRender(({ nuxtState }) => {
        nuxtState.test = true
      })
    }
  }
}
</script>
```

### 仅客户端可用的 keys

#### from

`route`导航的来源from

#### nuxtState

nuxt 状态，对于使用 beforeNuxtRender 的插件来说非常有用，在客户端获得 Nuxt 状态。

但只能在`universal`模式下使用。

## pages

