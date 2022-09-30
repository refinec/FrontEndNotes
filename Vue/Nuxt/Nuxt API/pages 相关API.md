### asyncData 方法

> 在服务器端获取数据并渲染。`asyncData`*方法使得你能够**在渲染组件之前异步获取数据**。*

`asyncData`方法会在组件（**限于页面组件**）每次加载之前被调用。

它可以在**服务端**或**路由**更新之前被调用。在这个方法被调用的时候，第一个参数被设定为当前页面的**上下文对象**，你可以利用 `asyncData`方法来获取数据并返回给当前组件。

```vue
<template>
  <div>
    <div>{{ content }}</div>
  </div>
</template>

<script>
  export default {
    asyncData() {
      return { content: 'Created at: ' + new Date() }
    },
  }
</script>
```

注意📢：由于`asyncData`方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 `this` 来引用组件的实例对象。

### fetch 方法

> fetch 方法用于在渲染页面前**填充应用的状态树（store）数据**， 与 asyncData 方法类似，它会在组件每次加载前被调用（在服务端或切换至目标路由之前），所以也无法在内部使用`this`获取**组件实例**。不同的是它不会设置组件的数据。

`fetch` 方法的第一个参数是页面组件的`context`，为了让获取过程可以异步，你需要**返回一个 Promise**，Nuxt.js 会等这个 promise 完成后再渲染组件。

```vue
<template>
  <h1>Stars: {{ $store.state.stars }}</h1>
</template>

<script>
  export default {
    fetch({ store, params }) {
      return axios.get('http://my-api/stars').then(res => {
        store.commit('setStars', res.data)
      })
    }
  }
</script>
```

```vue
<template>
  <h1>Stars: {{ $store.state.stars }}</h1>
</template>

<script>
  export default {
    async fetch({ store, params }) {
      let { data } = await axios.get('http://my-api/stars')
      store.commit('setStars', data)
    }
  }
</script>
```

如果要在`fetch`中调用并操作`store`，请使用`store.dispatch`，但是要确保在内部使用`async / await`等待操作结束：

```vue
<script>
  export default {
    async fetch({ store, params }) {
      await store.dispatch('GET_STARS')
    }
  }
</script>
```

store/index.js

```js
// ...
export const actions = {
  async GET_STARS({ commit }) {
    const { data } = await axios.get('http://my-api/stars')
    commit('SET_STARS', data)
  }
}
```

#### 路由 query 字符串改变不触发`fetch`方法怎么办?

默认情况下，不会在查询字符串更改时调用`fetch`方法。如果想更改此行为，例如，在编写分页组件时，您可以设置通过页面组件的`watchQuery`属性来监听参数的变化。

### validate 方法

> 在动态路由对应的页面组件中配置一个校验方法用于校验动态路由参数的有效性。校验参数无效返回404页面。

```js
validate({ params, query }) {
  return true // 如果参数有效
  return false // 参数无效，Nuxt.js 停止渲染当前页面并显示错误页面
}

async validate({ params, query, store }) {
  // await operations
  return true // 如果参数有效
  return false // 将停止Nuxt.js呈现页面并显示错误页面
}

validate({ params, query, store }) {
  return new Promise((resolve) => setTimeout(() => resolve()))
}
```

Nuxt.js 可以让你在动态路由对应的页面组件（本例为： `pages/users/_id.vue`）中配置一个校验方法。

* 如果校验方法返回的值不为 `true`， Nuxt.js 将自动加载显示 404 错误页面。

```js
export default {
  validate({ params }) {
    // Must be a number
    return /^\d+$/.test(params.id)
  }
}
```

* 也可以在 validate 方法中校验 [store](https://www.nuxtjs.cn/guide/vuex-store) 的数据 (如果 store 此前在 [nuxtServerInit 方法](https://www.nuxtjs.cn/guide/vuex-store#nuxtServerInit-方法) 中被设置了的话 ):

```js
export default {
  validate({ params, store }) {
    // 校验 `params.id` 是否存在
    return store.state.categories.some(category => category.id === params.id)
  }
}
```

还可以在验证函数执行期间抛出预期或意外错误：

```js
export default {
  async validate({ params, store }) {
    // 使用自定义消息触发内部服务器500错误
    throw new Error('Under Construction!')
  }
}
```

### head 方法

- **类型：** `Object` 或 `Function`

> 使用 `head` 方法设置当前页面的头部标签，Nuxt使用了 [`vue-meta`](https://github.com/nuxt/vue-meta) 更新应用的 `头部标签(Head)`和 `html 属性`。

**在 `head` 方法里可通过 `this` 关键字来获取组件的数据。**

```vue
<template>
  <h1>{{ title }}</h1>
</template>

<script>
  export default {
    data() {
      return {
        title: 'Hello World!'
      }
    },
    head() {
      return {
        title: this.title,
        meta: [
          {
            hid: 'description',
            name: 'description',
            content: 'My custom description'
          }
        ]
      }
    }
  }
</script>
```

注意：为了避免子组件中的 meta 标签不能正确覆盖父组件中相同的标签而产生重复的现象，建议利用 `hid` 键为 meta 标签配一个唯一的标识编号。请阅读[关于 `vue-meta` 的更多信息](https://vue-meta.nuxtjs.org/api/#tagidkeyname)。

### key 属性

- **类型:** `String` 或 `Function`

> 设置内部`<router-view>`组件的`key`属性。`key`属性赋值到`<router-view>`，这对于在动态页面和不同路径中进行转换很有用。不同的`key`会使页面组件重新渲染。

```js
export default {
  key(route) {
    return route.fullPath
  }
}
```

结合[nuxt 组件](https://www.nuxtjs.cn/api/components-nuxt)中的`nuxtChildKey`属性使用

### layout 属性

- **类型：** `String` 或 `Function` (默认值： `'default'`)

> `layouts` 根目录下的所有文件都属于个性化布局文件，可以在页面组件中利用`layout` 属性来引用。

```js
export default {
  layout: 'blog', // 使用 layout 属性来为页面指定使用哪一个布局文件, 值为文件名
  // 或
  layout(context) {
    return 'blog'
  }
}
```

在上面的例子中， Nuxt.js 会使用 `layouts/blog.vue` 作为当前页面组件的布局文件。

### loading 属性

- **类型:** `Boolean` (默认: `true`)

> 指定页面上默认加载进度条的开启或关闭。默认情况下，Nuxt.js 使用自己的组件来显示路由跳转之间的进度条

您可以通过[Configuration 的加载选项](https://www.nuxtjs.cn/api/configuration-loading)全局禁用或自定义它，但也可以通过将 `loading` 属性设置为 `false` 来禁用特定页面：

```vue
<template>
  <h1>My page</h1>
</template>

<script>
  export default {
    loading: false
  }
</script>
```

### transition 属性

- **类型：** `String` 或 `Object` 或 `Function`

> Nuxt.js 使用 Vue.js 的`transition`组件来实现路由切换时的页面过渡动效。

如果想给某个页面自定义过渡特效的话，只要在该页面组件中配置 `transition` 字段即可：

```js
export default {
  // 可以是字符
  transition: ''
  // 或对象
  transition: {}
  // 或函数
  transition (to, from) {}
}
```

#### string

> 如果 `transition` 属性的值类型是字符类型， 相当于设置了动效配置对象的 name 属性：`transition.name`。

```js
export default {
  transition: 'test'
}
```

Nuxt.js 将使用上面的配置来设置 Vue.js transition 组件，如下：

```html
<transition name="test"></transition>
```

#### object

如果 `transition` 属性的值类型是对象类型：

```js
export default {
  transition: {
    name: 'test',
    mode: 'out-in'
  }
}
```

Nuxt.js 将使用上面的配置来设置 Vue.js transition 组件，如下：

```html
<transition name="test" mode="out-in"></transition>
```

`transition` 允许配置的字段介绍：

| 属性字段           | 类型    | 默认值     | 描述                                                         |
| ------------------ | ------- | ---------- | ------------------------------------------------------------ |
| `name`             | String  | `"page"`   | 所有路由过渡都会用到的过渡名称。                             |
| `mode`             | String  | `"out-in"` | 所有路由都用到的过渡模式，见 [Vue.js transition 使用文档](http://vuejs.org/v2/guide/transitions.html#Transition-Modes)。 |
| `css`              | Boolean | `true`     | 是否给页面组件根元素添加 CSS 过渡类名。如果值为 `false`，路由过渡时将只会触发页面组件注册的 Javascript 钩子事件方法（不会设置 css 类名）。 |
| `duration`         | Integer | `n/a`      | 在页面切换的持续时间（以毫秒为单位）详情请参考 [Vue.js documentation](https://vuejs.org/v2/guide/transitions.html#Explicit-Transition-Durations) |
| `type`             | String  | `n/a`      | 指定过滤动效事件的类型，用于判断过渡结束的时间点。值可以是 "transition" 或 "animation"。 默认情况下, Nuxt.js 会自动侦测动效事件的类型。 |
| `enterClass`       | String  | `n/a`      | 目标路由动效开始时的类名。 详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |
| `enterToClass`     | String  | `n/a`      | 目标路由动效结束时的类名。 详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |
| `enterActiveClass` | String  | `n/a`      | 目标路由过渡过程中的类名。详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |
| `leaveClass`       | String  | `n/a`      | 当前路由动效开始时的类名。 详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |
| `leaveToClass`     | String  | `n/a`      | 当前路由动效结束时的类名。 详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |
| `leaveActiveClass` | String  | `n/a`      | 当前路由动效过程中的类名。详情请参考 [Vue.js transition 使用文档](https://vuejs.org/v2/guide/transitions.html#Custom-Transition-Classes) 。 |

你也可以在页面组件事件里面使用 Javascript 来控制过渡动效，可以称之为 [JavaScript 钩子方法](https://vuejs.org/v2/guide/transitions.html#JavaScript-Hooks)：

- beforeEnter(el)
- enter(el, done)
- afterEnter(el)
- enterCancelled(el)
- beforeLeave(el)
- leave(el, done)
- afterLeave(el)
- leaveCancelled(el)

*注意：如果使用纯 Javascript 控制路由过渡动效，建议将 `transition` 组件的 `css` 属性的值设置为 `false`。这样可以避免 Vue 做 CSS 动效相关的侦测逻辑，同时防止 CSS 影响到过渡的动效。*

#### Function

```js
export default {
  transition(to, from) {
    if (!from) {
      return 'slide-left'
    }
    return +to.query.page < +from.query.page ? 'slide-right' : 'slide-left'
  }
}
```

- `/` to `/posts` => `slide-left`
- `/posts` to `/posts?page=3` => `slide-left`
- `/posts?page=3` to `/posts?page=2` => `slide-right`

### middleware 属性

类型： `String` 或 `Array<String>`

> 为页面设置中间件`middleware`

```vue
<!-- pages/secret.vue -->
<template>
  <h1>Secret page</h1>
</template>

<script>
  export default {
    middleware: 'authenticated'
  }
</script>
```

```js
// middleware/authenticated.js
export default function ({ store, redirect }) {
  // 如果用户没有被授权
  if (!store.state.authenticated) {
    return redirect('/login')
  }
}
```

### scrollToTop 属性

- **类型：** `Boolean` (默认值： `false`)

> scrollToTop 属性用于**控制页面渲染前是否滚动至页面顶部**。

* 默认情况下，从当前页面切换至目标页面时，Nuxt.js 会让目标页面滚动至顶部。
* 但是在**嵌套子路由**的场景下，Nuxt.js 会保持当前页面的滚动位置，除非在子路由的页面组件中将 `scrollToTop` 设置为 `true`。

```vue
<template>
  <h1>子页面组件</h1>
</template>

<script>
  export default {
    scrollToTop: true
  }
</script>
```

如果你想改变 Nuxt.js 上述的默认的页面滚动逻辑，请参考 [scrollBehavior 配置项](https://www.nuxtjs.cn/api/configuration-router#scrollBehavior)。

### watchQuery 属性

- **类型:** `Boolean` or `Array` (默认: `[]`)

> **监听参数字符串更改**并在更改时执行组件方法 (`asyncData`, `fetch`, `validate`, `layout`, ...)，为了提高性能，默认情况下禁用。

如果您要**为所有参数字符串设置监听**， 请设置： `watchQuery: true`.

```js
export default {
  watchQuery: ['page']
}
```