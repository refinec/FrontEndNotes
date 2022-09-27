> Nuxt.js 为页面组件添加了一些特殊的配置项

```vue
<template>
  <h1 class="red">Hello {{ name }}!</h1>
</template>

<script>
  export default {
    asyncData (context) {
      // called every time before loading the component
      return { name: 'World' }
    },
    fetch () {
      // The fetch method is used to fill the store before rendering the page
    },
    head () {
      // Set Meta Tags for this Page
    },
    // and more functionality to discover
    ...
  }
</script>

<style>
  .red {
    color: red;
  }
</style>
```

Nuxt.js 为页面提供的特殊配置项：

| 属性名      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| asyncData   | 最重要的一个键, 支持 [异步数据处理](https://www.nuxtjs.cn/guide/async-data)，在设置组件的数据之前能异步获取或处理数据。另外该方法的第一个参数为当前页面组件的 [上下文对象](https://www.nuxtjs.cn/api#上下文对象)。 |
| fetch       | 与 `asyncData` 方法类似，用于在渲染页面之前获取数据填充应用的状态树（store）。不同的是 `fetch` 方法不会设置组件的数据。详情请参考 [关于 fetch 方法的文档](https://www.nuxtjs.cn/api/pages-fetch)。 |
| head        | 配置当前页面的 Meta 标签, 详情参考 [页面头部配置 API](https://www.nuxtjs.cn/api/pages-head)。 |
| layout      | 指定当前页面使用的布局（`layouts` 根目录下的布局文件）。详情请参考 [关于 布局 的文档](https://www.nuxtjs.cn/api/pages-layout)。 |
| loading     | 如果设置为`false`，则阻止页面自动调用`this.$nuxt.$loading.finish()`和`this.$nuxt.$loading.start()`,您可以手动控制它,请看[例子](https://nuxtjs.org/examples/custom-page-loading),仅适用于在 nuxt.config.js 中设置`loading`的情况下。请参考[API 配置 `loading` 文档](https://www.nuxtjs.cn/api/configuration-loading)。 |
| transition  | 指定页面切换的过渡动效, 详情请参考 [页面过渡动效](https://www.nuxtjs.cn/api/pages-transition)。 |
| scrollToTop | 布尔值，默认: `false`。 用于判定渲染页面前是否需要将当前页面滚动至顶部。这个配置用于 [嵌套路由](https://www.nuxtjs.cn/guide/routing#嵌套路由)的应用场景。 |
| validate    | 校验方法用于校验 [动态路由](https://www.nuxtjs.cn/guide/routing#动态路由)的参数。 |
| middleware  | 指定页面的中间件，中间件会在页面渲染之前被调用， 请参考 [路由中间件](https://www.nuxtjs.cn/guide/routing#中间件)。 |

关于页面配置项的详细信息，请参考 [页面 API](https://www.nuxtjs.cn/api)。