## `<nuxt>`组件

> 该组件只适用于在 [布局](https://www.nuxtjs.cn/guide/views#布局) 中显示页面组件（即非布局内容）

例子 (`layouts/default.vue`)：

```vue
<template>
  <div>
    <div>页头</div>
    <nuxt />
    <div>页脚</div>
  </div>
</template>
```

**Props**:

* `name`: **string**(Nuxt v2.4.0 新增)

  - 此 prop 将设置为`<router-view />` 上的name，用于渲染命名视图。

  - 默认值: `default`

- `nuxtChildKey`: **string**
  - 此 prop 将设置为**`<router-view />`**上的key，可用于在动态页面和不同路由内进行过渡。
  - 默认值: **`$route.path`**

有三种方式可以处理 `<router-view />` 内部属性的 `key`：

1. `nuxtChildKey` 属性：

```vue
<template>
  <div>
    <nuxt :nuxt-child-key="someKey" />
  </div>
</template>
```

2. 页面组件中的`key`选项：`string` 或 `function`

```js
export default {
  key(route) {
    return route.fullPath
  }
}
```

3. 页面组件中的`watchQuery`选项：`boolean` 或 `string []`

[watchQuery](https://www.nuxtjs.cn/api/pages-watchquery)选项中指定的查询会被用于构建`key`。如果`watchQuery`为`true`，则默认使用`fullPath`。

## `<nuxt-child>` 组件

> 该组件用于显示[嵌套路由](https://www.nuxtjs.cn/guide/routing#嵌套路由)场景下的页面内容

```
-| pages/
---| parent/
------| child.vue
---| parent.vue
```

上面的目录树结构会生成下面这些路由配置：

```js
;[
  {
    path: '/parent',
    component: '~/pages/parent.vue',
    name: 'parent',
    children: [
      {
        path: 'child',
        component: '~/pages/parent/child.vue',
        name: 'parent-child'
      }
    ]
  }
]
```

1. 为了显示 `child.vue` 组件，我们需要在父级页面组件 `pages/parent.vue` 中加入 `<nuxt-child/>`：

```vue
<template>
  <div>
    <h1>我是父级页面</h1>
    <nuxt-child :foobar="123" />
  </div>
</template>
```

2. `<nuxt-child/>` 接收 **`keep-alive`** 和 **`keep-alive-props`**:

```vue
<template>
  <div>
    <nuxt-child keep-alive :keep-alive-props="{ exclude: ['modal'] }" />
  </div>
</template>

<!-- 将被转换成以下形式 -->
<div>
  <keep-alive :exclude="['modal']">
    <router-view />
  </keep-alive>
</div>
```

子组件还可以接收 Vue 组件等属性。

可以看这个实际案例：[嵌套路由示例](https://www.nuxtjs.cn/examples/nested-routes)

3. 接受`name` prop 来呈现渲染命名视图：

```vue
<template>
  <div>
    <nuxt-child name="top" />
    <nuxt-child />
  </div>
</template>
```

查看更多例子，请点击 [named-views 例子](https://www.nuxtjs.cn/examples/named-views).

## `<nuxt-link>` 组件

> **别名:** `<n-link>`, `<NuxtLink>`, 和 `<NLink>`
> `nuxt-link` 组件用于在页面中添加链接至别的页面。作用和`router-link`一致
>
> 为了提高 Nuxt.js 应用程序的响应能力，当链接将显示在视口中时，Nuxt.js 将自动预获取代码分割页面。此功能的灵感来自 Google Chrome Labs 的[quicklink.js](https://github.com/GoogleChromeLabs/quicklink)。

要禁用链接页面的预获取，可以使用`no-prefetch`：

```vue
<n-link to="/about" no-prefetch>About page not pre-fetched</n-link>
```

您可以使用[router.prefetchLinks](https://www.nuxtjs.cn/api/configuration-router#prefetchlinks)全局配置此行为。

关于`prefetched-class`还可用于自定义在预获取代码分割页面时添加的类。确保使用[router.linkPrefetchedClass](https://www.nuxtjs.cn/api/configuration-router#linkprefetchedclass)全局设置此功能。

## `<client-only>` 组件

> 此组件用于仅在客户端渲染其他组件。Nuxt 版本小于 `v2.9.0` 的用户, 请使用 **`<no-ssr>`**

**属性：**

- placeholder：`string`

  - 在 `<client-only />` 被挂载之前, 使用此属性作为文本占位符.

    ```vue
    <template>
      <div>
        <sidebar />
        <client-only placeholder="Loading...">
          <!-- comments 组件只会在客户端被渲染 -->
          <comments />
        </client-only>
      </div>
    </template>
    ```

**Slots**:

- placeholder:

  - 在 `<client-only />` 被挂载之前, 使用此属性作为插槽.

    ```vue
    <template>
      <div>
        <sidebar />
        <client-only>
          <!-- comments 组件只会在客户端被渲染 -->
          <comments />
    
          <!-- comments-placeholder 会在服务端被加载-->
          <comments-placeholder slot="placeholder" />
        </client-only>
      </div>
    </template>
    ```

    
