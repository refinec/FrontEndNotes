> Nuxt.js 允许你扩展默认的布局，或在 `layout` 目录下创建自定义的布局。

## 默认布局

可通过添加 `layouts/default.vue` 文件来扩展应用的默认布局。

**提示:** 别忘了在布局文件中添加 `<nuxt/>` 组件用于显示页面的主体内容。

默认布局的源码如下：

```html
<template>
  <nuxt />
</template>
```

## 自定义布局

`layouts` 目录中的每个文件 (*顶级*) 都将创建一个可通过页面组件中的 `layout` 属性访问的自定义布局。

假设我们要创建一个 *博客布局* 并将其保存到`layouts/blog.vue`:

```html
<template>
  <div>
    <div>我的博客导航栏在这里</div>
    <nuxt />
  </div>
</template>
```

然后我们必须告诉页面 (即`pages/posts.vue`) 使用您的自定义布局：

```html
<template>
  <!-- Your template -->
</template>
<script>
  export default {
    layout: 'blog'
    // page component definitions
  }
</script>
```

更多有关 `layout` 属性信息: [API 页面 `布局`](https://www.nuxtjs.cn/api/pages-layout)

## 错误页面

*通过编辑* `layouts/error.vue` 文件来定制化错误页面

> **警告:** 虽然此文件放在 `layouts` 文件夹中, 但应该将它看作是一个 **页面(page)**.

这个布局文件不需要包含 `<nuxt/>` 标签。你可以把这个布局文件当成是显示应用错误（404，500 等）的组件。

**默认的错误页面源码如下：**

```vue
<template>
  <div class="__nuxt-error-page">
    <div class="error">
      <svg xmlns="http://www.w3.org/2000/svg" width="90" height="90" fill="#DBE1EC" viewBox="0 0 48 48">
        <path d="M22 30h4v4h-4zm0-16h4v12h-4zm1.99-10C12.94 4 4 12.95 4 24s8.94 20 19.99 20S44 35.05 44 24 35.04 4 23.99 4zM24 40c-8.84 0-16-7.16-16-16S15.16 8 24 8s16 7.16 16 16-7.16 16-16 16z" />
      </svg>

      <div class="title">{{ message }}</div>
      <p v-if="statusCode === 404" class="description">
        <a v-if="typeof $route === 'undefined'" class="error-link" href="/"><% messages.back_to_home %></a>
        <NuxtLink v-else class="error-link" to="/"><%= messages.back_to_home %></NuxtLink>
      </p>
      <% if(debug) { %>
      <p class="description" v-else><%= messages.client_error_details %></p>
      <% } %>

      <div class="logo">
        <a href="https://nuxtjs.org" target="_blank" rel="noopener"><%= messages.nuxtjs %></a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'NuxtError',
  props: {
    error: {
      type: Object,
      default: null
    }
  },
  computed: {
    statusCode () {
      return (this.error && this.error.statusCode) || 500
    },
    message () {
      return this.error.message || '<%= messages.client_error %>'
    }
  },
  head () {
    return {
      title: this.message,
      meta: [
        {
          name: 'viewport',
          content: 'width=device-width,initial-scale=1.0,minimum-scale=1.0'
        }
      ]
    }
  }
}
</script>

<style>
.__nuxt-error-page {
  padding: 1rem;
  background: #F7F8FB;
  color: #47494E;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  font-family: sans-serif;
  font-weight: 100 !important;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -webkit-font-smoothing: antialiased;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.__nuxt-error-page .error {
  max-width: 450px;
}
.__nuxt-error-page .title {
  font-size: 1.5rem;
  margin-top: 15px;
  color: #47494E;
  margin-bottom: 8px;
}
.__nuxt-error-page .description {
  color: #7F828B;
  line-height: 21px;
  margin-bottom: 10px;
}
.__nuxt-error-page a {
  color: #7F828B !important;
  text-decoration: none;
}
.__nuxt-error-page .logo {
  position: fixed;
  left: 12px;
  bottom: 12px;
}
</style>
```

举一个个性化错误页面的例子 `layouts/error.vue`:

```vue
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">页面不存在</h1>
    <h1 v-else>应用发生错误异常</h1>
    <nuxt-link to="/">首 页</nuxt-link>
  </div>
</template>

<script>
  export default {
    props: ['error'],
    layout: 'blog' // 你可以为错误页面指定自定义的布局
  }
</script>
```