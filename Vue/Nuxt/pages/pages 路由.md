## 基础路由

假设 `pages` 的目录结构如下：

```bash
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
```

那么，Nuxt.js 自动生成的路由配置如下：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'user',
      path: '/user',
      component: 'pages/user/index.vue'
    },
    {
      name: 'user-one',
      path: '/user/one',
      component: 'pages/user/one.vue'
    }
  ]
}
```

## 动态路由

定义**带参数**的动态路由，需要创建对应的**以下划线作为前缀**的 **Vue 文件** 或 **目录**。

以下目录结构：

```bash
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```

Nuxt.js 生成对应的路由配置表为：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id?',
      component: 'pages/users/_id.vue'
    },
    {
      name: 'slug',
      path: '/:slug',
      component: 'pages/_slug/index.vue'
    },
    {
      name: 'slug-comments',
      path: '/:slug/comments',
      component: 'pages/_slug/comments.vue'
    }
  ]
}
```

上面名称为 **`users-id`** 的路由路径带有 **`:id?`** 参数，表示该路由是**可选的**。如果你想将它设置为必选的路由，需要在 `users/_id` 目录内创建一个 `index.vue` 文件。

**警告：**`generate` 命令会忽略动态路由: [API Configuration generate](https://www.nuxtjs.cn/api/configuration-generate#routes)

### 动态路由参数校验

> 可以让你在**动态路由组件中**定义参数校验方法，参考 [页面校验 API](https://www.nuxtjs.cn/api/pages-validate)。

举个例子： `pages/users/_id.vue`

```js
export default {
  validate({ params }) {
    // 必须是number类型
    return /^\d+$/.test(params.id)
  }
}
```

如果校验方法返回的值不为 `true`或`Promise`中 resolve 解析为`false`或抛出 Error ， Nuxt.js 将自动加载显示 404 错误页面或 500 错误页面。

```vue
<template>
  <div class="user">
    <h3>{{ name }}</h3>
    <h4>@{{ username }}</h4>
    <p>Email : {{ email }}</p>
    <p>
      <NuxtLink to="/">
        List of users
      </NuxtLink>
    </p>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  validate ({ params }) {
    return !isNaN(+params.id)
  },
  async asyncData ({ params, error }) {
    try {
      const { data } = await axios.get(`https://jsonplaceholder.typicode.com/users/${+params.id}`)
      return data
    } catch (e) {
      error({ message: 'User not found', statusCode: 404 })
    }
  }
}
</script>

<style scoped>
.user {
  text-align: center;
  margin-top: 100px;
  font-family: sans-serif;
}
</style>
```

### SPA fallback

您也可以为动态路由启用`SPA fallback`。在使用`mode:'spa'`模式下，Nuxt.js 将输出一个与`index.html`相同的额外文件。如果没有文件匹配，大多数静态托管服务可以配置为使用 SPA 模板。生成文件不包含头信息或任何 HTML，但它仍将解析并加载 API 中的数据。

我们在`nuxt.config.js`文件中启用它：

```js
export default {
  generate: {
    fallback: true, // if you want to use '404.html'
    fallback: 'my-fallback/file.html' // if your hosting needs a custom location
  }
}
```

#### 在 Surge 上实现

Surge [可以处理](https://surge.sh/help/adding-a-custom-404-not-found-page)`200.html` 和 `404.html`，`generate.fallback`默认设置为`200.html`，因此无需更改它。

#### 在 GitHub Pages 和 Netlify 上实现

GitHub Pages 和 Netlify 自动识别 `404.html`文件，所以我们需要做的就是将 `generate.fallback` 设置为 `true`！

#### 在 Firebase Hosting 上实现

要在 Firebase Hosting 上使用，请将 `generate.fallback` 配置为 `true` 并使用以下配置 ([more info](https://firebase.google.com/docs/hosting/url-redirects-rewrites#section-rewrites))：

```json
{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/404.html"
      }
    ]
  }
}
```

## 嵌套路由(内嵌子路由)

> 创建内嵌子路由，你需要添加一个 **Vue 文件**，同时添加一个**与该文件同名**且同级的**目录**用来**存放子视图组件**。

**警告:** 别忘了在父组件(`.vue`文件) 内增加 **`<nuxt-child/>`** 用于显示子视图内容

假设文件结构如：

```bash
pages/
--| users/
-----| _id.vue
-----| index.vue
--| users.vue
```

Nuxt.js 自动生成的路由配置如下：

```js
router: {
  routes: [
    {
      path: '/users',
      component: 'pages/users.vue',
      children: [
        {
          path: '',
          component: 'pages/users/index.vue',
          name: 'users'
        },
        {
          path: ':id',
          component: 'pages/users/_id.vue',
          name: 'users-id'
        }
      ]
    }
  ]
}
```

### 动态嵌套路由

> 在动态路由下配置动态子路由

假设文件结构如下：

```bash
pages/
--| _category/
-----| _subCategory/
--------| _id.vue
--------| index.vue
-----| _subCategory.vue
-----| index.vue
--| _category.vue
--| index.vue
```

Nuxt.js 自动生成的路由配置如下：

```js
router: {
  routes: [
    {
      path: '/',
      component: 'pages/index.vue',
      name: 'index'
    },
    {
      path: '/:category',
      component: 'pages/_category.vue',
      children: [
        {
          path: '',
          component: 'pages/_category/index.vue',
          name: 'category'
        },
        {
          path: ':subCategory',
          component: 'pages/_category/_subCategory.vue',
          children: [
            {
              path: '',
              component: 'pages/_category/_subCategory/index.vue',
              name: 'category-subCategory'
            },
            {
              path: ':id',
              component: 'pages/_category/_subCategory/_id.vue',
              name: 'category-subCategory-id'
            }
          ]
        }
      ]
    }
  ]
}
```

如果您不知道 URL 结构的深度，您可以使用**`_.vue`**动态匹配嵌套路径。这将处理与*更具体*请求不匹配的情况。

文件结构:

```bash
pages/
--| people/
-----| _id.vue
-----| index.vue
--| _.vue
--| index.vue
```

将处理这样的请求：

| Path                     | File               |
| ------------------------ | ------------------ |
| `/`                      | `index.vue`        |
| `/people`                | `people/index.vue` |
| `/people/123`            | `people/_id.vue`   |
| `/about`                 | `_.vue`            |
| `/about/careers`         | `_.vue`            |
| `/about/careers/chicago` | `_.vue`            |

**Note:** 处理 404 页面，现在符合`_.vue`页面的逻辑。 [有关 404 重定向的更多信息，请点击此处](https://www.nuxtjs.cn/guide/async-data#handling-errors).

### 命名视图

要渲染命名视图，您可以在**`布局(layout) / 页面(page)`**中使用 **`<nuxt name="top"/>`** 或 **`<nuxt-child name="top"/>`** 组件。要指定页面的**命名视图**，我们需要在**`nuxt.config.js`**文件中扩展路由器配置：

[命名视图示例](https://www.nuxtjs.cn/examples/named-views)

```js
export default {
  router: {
    extendRoutes(routes, resolve) {
      const index = routes.findIndex(route => route.name === 'main')
      routes[index] = {
        ...routes[index],
        components: {
          default: routes[index].component,
          top: resolve(__dirname, 'components/mainTop.vue')
        },
        chunkNames: {
          top: 'components/mainTop'
        }
      }
    }
  }
}
```

它需要使用**两个属性** `components` 和 `chunkNames` 扩展路由。此配置示例中的命名视图名称为 `top` 

## 过渡动效

> Nuxt.js 使用 Vue.js 的 [transition](http://vuejs.org/v2/guide/transitions.html#Transitioning-Single-Elements-Components) 组件来实现路由切换时的过渡动效。

### 全局过渡动效设置

> Nuxt.js 默认使用的过渡效果名称为 **`page`**

如果想让每一个页面的切换都有淡出 (fade) 效果，我们需要创建一个所有路由共用的 CSS 文件。所以我们可以在 **`assets/`** 目录下创建这个文件：

```css
/* assets/main.css */
.page-enter-active,
.page-leave-active {
  transition: opacity 0.5s;
}
.page-enter,
.page-leave-active {
  opacity: 0;
}
```

然后添加到 `nuxt.config.js` 文件中：

```js
module.exports = {
  css: ['assets/main.css']
}
```

关于过渡效果 `transition` 属性配置的更多信息可参看 [页面过渡效果 API](https://www.nuxtjs.cn/api/pages-transition)。

### 页面过渡动效设置

如果想给某个页面自定义过渡特效的话，只要在该页面组件中配置 `transition` 字段即可：

在全局样式 `assets/main.css` 中添加一下内容：

```css
.test-enter-active,
.test-leave-active {
  transition: opacity 0.5s;
}
.test-enter,
.test-leave-active {
  opacity: 0;
}
```

然后我们将页面组件中的 `transition` 属性的值设置为 `test` 即可：

```js
export default {
  transition: 'test'
}
```

关于过渡效果 `transition` 属性配置的更多信息可参看 [页面过渡效果 API](https://www.nuxtjs.cn/api/pages-transition)。