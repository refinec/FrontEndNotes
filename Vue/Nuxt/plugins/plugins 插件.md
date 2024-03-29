> Nuxt.js 在运行 Vue.js 应用程序之前执行 js 插件，这在需要使用第三方模块或库的时候特别有用

需要注意的是，在任何 Vue 组件的[生命周期](https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram)内， 只有 **`beforeCreate`** 和 **`created`** 这两个方法会在 **客户端和服务端**被调用。其他生命周期函数仅在**客户端**被调用。

## 使用第三方模块

一个典型的例子是在客户端和服务端使用 [axios](https://github.com/mzabriskie/axios) 做 HTTP 请求。

```vue
<template>
  <h1>{{ title }}</h1>
</template>

<script>
  import axios from 'axios'

  export default {
    async asyncData({ params }) {
      let { data } = await axios.get(`https://my-api/posts/${params.id}`)
      return { title: data.title }
    }
  }
</script>
```

## 使用 Vue 插件

假如我们想使用 [vue-notifications](https://github.com/se-panfilov/vue-notifications) 显示应用的通知信息，我们需要在程序运行前配置好这个插件。

首先增加文件 `plugins/vue-notifications.js`：

```js
import Vue from 'vue'
import VueNotifications from 'vue-notifications'

Vue.use(VueNotifications)
```

然后, 在 `nuxt.config.js` 内配置 `plugins` 如下：

```js
module.exports = {
  plugins: ['~/plugins/vue-notifications']
}
```

请参考 [插件 API 文档](https://www.nuxtjs.cn/api/configuration-plugins)。

## ES6 插件

如果插件位于`node_modules`并导出模块，需要将其添加到`transpile`构建选项：

```js
module.exports = {
  build: {
    transpile: ['vue-notifications']
  }
}
```

参考 [构建配置](https://www.nuxtjs.cn/api/configuration-build/#transpile) 文档来获取更多构建选项

## 注入 $root 和 context

> 有时您希望在整个应用程序中**使用某个函数**或**属性值**，此时，你需要将它们注入到 **Vue 实例（客户端）**，**context（服务器端）**甚至 **store(Vuex)**。按照惯例，新增的属性或方法名使用**`$`作为前缀**。

```js
export default function ({ app, $axios, store, error, redirect, req }, inject) {
  
}
```

### 注入到 Vue 实例

将内容注入 Vue 实例，避免重复引入，在 Vue 原型上挂载注入一个函数，所有组件内都可以访问(**仅客户端**)。

```js
// plugins/vue-inject.js
import Vue from 'vue'

Vue.prototype.$myInjectedFunction = string =>
  console.log('This is an example', string)
```

```js
// nuxt.config.js
export default {
  plugins: ['~/plugins/vue-inject.js']
}
```

这样，您就可以在所有 Vue 组件中使用该函数。

```js
// example-component.vue
export default {
  mounted() {
    this.$myInjectedFunction('test')
  }
}
```

### 注入到 context

context 注入方式和在其它 vue 应用程序中注入类似。

```js
// plugins/ctx-inject.js
export default ({ app }, inject) => {
  // Set the function directly on the context.app object
  app.myInjectedFunction = string =>
    console.log('Okay, another function', string)
}
```

```js
// nuxt.config.js
export default {
  plugins: ['~/plugins/ctx-inject.js']
}
```

现在，只要您获得 context，你就可以使用该函数（例如在`asyncData`和`fetch`中）。

```js
// ctx-example-component.vue
export default {
  asyncData(context) {
    context.app.myInjectedFunction('ctx!')
  }
}
```

### 同时注入到 Vue示例、context、Vuex

如果您需要同时在`context`，`Vue`实例，甚至`Vuex`中同时注入，您可以使用`inject`方法,它是 plugin 导出函数的第二个参数。将内容注入 Vue 实例的方式与在 Vue 应用程序中进行注入类似。系统会自动将`$`添加到方法名的前面。

```js
// plugins/combined-inject.js
export default ({ app }, inject) => {
  inject('myInjectedFunction', string => console.log('That was easy!', string))
}
```

```js
// nuxt.config.js
export default {
  plugins: ['~/plugins/combined-inject.js']
}
```

现在您就可以在`context`，或者`Vue`实例中的`this`，或者`Vuex`的`actions/mutations`方法中的`this`来调用`myInjectedFunction`方法。 

```js
// ctx-example-component.vue
export default {
  mounted() {
    this.$myInjectedFunction('works in mounted')
  },
  asyncData(context) {
    context.app.$myInjectedFunction('works with context')
  }
}
```

```js
// store/index.js
export const state = () => ({
  someValue: ''
})

export const mutations = {
  changeSomeValue(state, newValue) {
    this.$myInjectedFunction('accessible in mutations')
    state.someValue = newValue
  }
}

export const actions = {
  setSomeValueToWhatever({ commit }) {
    this.$myInjectedFunction('accessible in actions')
    const newValue = 'whatever'
    commit('changeSomeValue', newValue)
  }
}
```

## 只在客户端使用的插件

> 不支持 ssr 的系统，插件只在浏览器里使用，这种情况下下，你可以用 **`ssr: false`** ，使得插件只会在客户端运行。

```js
// nuxt.config.js
module.exports = {
  // 由于Nuxt.js 2.4，模式已被引入作为插件的选项来指定插件类型，可能的值是：client 或 server, 
  // ssr:false 在下一个主要版本中弃用,将过渡为 mode: 'client'
  plugins: [{ src: '~/plugins/vue-notifications', ssr: false /* mode: 'client' */}]
}
```

```js
// plugins/vue-notifications.js
import Vue from 'vue'
import VueNotifications from 'vue-notifications'

Vue.use(VueNotifications)
```

- 您可以通过检测**`process.server`**这个变量来控制插件中的某些脚本库只在服务端使用。当值为 `true` 表示是当前执行环境为服务器中。
- 此外，可以通过检查**`process.static`**是否为`true`来判断应用是否通过`nuxt generator`生成。
- 您也可以组合**`process.server`**和**`process.static`**这两个选项，确定当前状态为服务器端渲染且使用`nuxt generate`命令运行。

### 传统命名插件

如果假设插件仅在 *客户端* 或 *服务器端* 运行，则 `.client.js` 或 `.server.js`可以作为插件文件的扩展名应用，该文件将自动包含在相应客户端或者服务端上。

```js
// nuxt.config.js
export default {
  plugins: [
    '~/plugins/foo.client.js', // only in client side
    '~/plugins/bar.server.js', // only in server side
    '~/plugins/baz.js' // both client & server
  ]
}
```

