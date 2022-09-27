> 对于每个大项目来说，使用状态树 (store) 管理状态 (state) 十分有必要。

## 使用状态树

Nuxt.js 会尝试找到 src 目录（默认是应用根目录）下的 `store` 目录，如果该目录存在，它将做以下的事情：

1. 引用 `vuex` 模块
2. 将 `vuex` 模块 加到 vendors 构建配置中去
3. 设置 `Vue` 根实例的 `store` 配置项

Nuxt.js 支持两种使用 `store` 的方式，你可以择一使用：

- **模块方式：** `store` 目录下的每个 `.js` 文件会被转换成为状态树[指定命名的子模块](http://vuex.vuejs.org/en/modules.html) （当然，`index` 是根模块）
- **Classic(不建议使用)：** `store/index.js`返回创建 Vuex.Store 实例的方法。

> 无论使用那种模式，`state`的值应该**始终是**`function`，为了避免返回引用类型，会导致多个实例相互影响。

### 普通(模块)方式

在`store` 目录下，其中包含与模块对应的每个文件

首先，只需将状态导出为 *函数*，将变量和操作作为 `store/index.js` 中的对象导出：

```js
export const state = () => ({
  counter: 0
})

export const mutations = {
  increment(state) {
    state.counter++
  }
}
```

然后，您可以拥有 `store/todos.js` 文件：

```js
export const state = () => ({
  list: []
})

export const mutations = {
  add(state, text) {
    state.list.push({
      text,
      done: false
    })
  },
  remove(state, { todo }) {
    state.list.splice(state.list.indexOf(todo), 1)
  },
  toggle(state, todo) {
    todo.done = !todo.done
  }
}
```

Vuex 将如下创建：

```js
new Vuex.Store({
  state: () => ({
    counter: 0
  }),
  mutations: {
    increment(state) {
      state.counter++
    }
  },
  modules: {
    todos: {
      namespaced: true,
      state: () => ({
        list: []
      }),
      mutations: {
        add(state, { text }) {
          state.list.push({
            text,
            done: false
          })
        },
        remove(state, { todo }) {
          state.list.splice(state.list.indexOf(todo), 1)
        },
        toggle(state, { todo }) {
          todo.done = !todo.done
        }
      }
    }
  }
})
```

在您的 `pages/todos.vue` 中，使用 `todos` 模块：

```vue
<template>
  <ul>
    <li v-for="todo in todos">
      <input type="checkbox" :checked="todo.done" @change="toggle(todo)" />
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
    </li>
    <li>
      <input placeholder="What needs to be done?" @keyup.enter="addTodo" />
    </li>
  </ul>
</template>

<script>
  import { mapMutations } from 'vuex'

  export default {
    computed: {
      todos() {
        return this.$store.state.todos.list
      }
    },
    methods: {
      addTodo(e) {
        this.$store.commit('todos/add', e.target.value)
        e.target.value = ''
      },
      ...mapMutations({
        toggle: 'todos/toggle'
      })
    }
  }
</script>

<style>
  .done {
    text-decoration: line-through;
  }
</style>
```

> 模块方法也适用于顶级定义，而无需在 `store` 目录中实现子目录

示例：您创建文件 `store/state.js` 并添加以下内容

```js
export default () => ({
  counter: 0
})
```

相应的可以在文件夹中添加 `store/mutations.js`

```js
export default {
  increment(state) {
    state.counter++
  }
}
```

### 插件

可以将其他插件添加到 store（**在模块模式下**），将其放入`store/index.js`文件中：

```js
import myPlugin from 'myPlugin'

export const plugins = [myPlugin]

export const state = () => ({
  counter: 0
})

export const mutations = {
  increment(state) {
    state.counter++
  }
}
```