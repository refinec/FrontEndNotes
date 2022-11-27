# `vue-class-componet`

> 以`class`类样式的语法创建`Vue`组件，👉🏻[详细阅读](https://github.com/vuejs/vue-class-component)

## 一、装饰器

### `@Component`

> 使用 `@Component` 装饰器对类进行注解，使你的类成为一个Vue组件:

```js
import Vue from 'vue'
import Component from 'vue-class-component'

// HelloWorld class will be a Vue component
@Component
export default class HelloWorld extends Vue {}
```

### `data`

> 组件数据

- `data`、`render`和所有`Vue生命周期钩子`也可以直接声明为类原型方法，但是不能在实例本身上调用它们。
- 在声明自定义方法时，应避免使用这些保留名称。

```vue
<template>
  <div>{{ message }}</div>
</template>

<script>
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class HelloWorld extends Vue {
  message = 'Hello World!' // 声明为组件数据
  
  /* 注意，如果初始值是`undefined`，class属性将不会是响应性的，这意味着属性的变化将不会被检测到 
   * 使用null 或 data hook来代替
  */
  // message = undefined
  data() {
    return {
      // `hello` will be reactive as it is declared via `data` hook.
      hello: undefined
    }
  }
  
}
</script>
```

### `Methods`

> 组件方法可以直接声明为类原型方法

```vue
<template>
  <button v-on:click="hello">Click</button>
</template>

<script>
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class HelloWorld extends Vue {
  // Declared as component method
  hello() {
    console.log('Hello World!')
  }
}
</script>
```

### ` Computed 属性`

> 计算属性可以声明为类属性 `getter / setter`

```vue
<template>
  <input v-model="name">
</template>

<script>
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class HelloWorld extends Vue {
  firstName = 'John'
  lastName = 'Doe'

  // Declared as computed property getter
  get name() {
    return this.firstName + ' ' + this.lastName
  }

  // Declared as computed property setter
  set name(value) {
    const splitted = value.split(' ')
    this.firstName = splitted[0]
    this.lastName = splitted[1] || ''
  }
  
  
}
</script>
```



### 其他选项

> 对于所有其他选项，将它们传递给装饰器函数:

```vue
<template>
  <OtherComponent />
</template>

<script>
import Vue from 'vue'
import Component from 'vue-class-component'
import OtherComponent from './OtherComponent.vue'

@Component({
  // Specify `components` option.
  // See Vue.js docs for all available options:
  // https://vuejs.org/v2/api/#Options-Data
  components: {
    OtherComponent
  }
})
export default class HelloWorld extends Vue {}
</script>
```

### 额外的Hooks

如果使用一些Vue插件(如 [Vue Router](https://link.juejin.cn/?target=https%3A%2F%2Frouter.vuejs.org%2F))，则可能需要类组件来解析它们提供的挂钩。在这种情况下，组件。`registerHooks`允许你注册这样的钩子:

`class-component-hooks.js`

```js
import Component from 'vue-class-component'

// Register the router hooks with their names
Component.registerHooks([
  'beforeRouteEnter',
  'beforeRouteLeave',
  'beforeRouteUpdate'
])
```

`main.js`

```js
// Make sure to register before importing any components
import './class-component-hooks'
```

`HelloWorld.vue`

```js
import Vue from 'vue'

@Component
export default class HelloWorld extends Vue {
  // The class component now treats beforeRouteEnter,
  // beforeRouteUpdate and beforeRouteLeave as Vue Router hooks
  beforeRouteEnter(to, from, next) {
    console.log('beforeRouteEnter')
    next()
  }

  beforeRouteUpdate(to, from, next) {
    console.log('beforeRouteUpdate')
    next()
  }

  beforeRouteLeave(to, from, next) {
    console.log('beforeRouteLeave')
    next()
  }
}
```

## 二、自定义装饰器

> 您可以通过创建自己的装饰器来扩展这个库的功能。Vue类组件提供了`createDecorator`方法来创建定制的`decorator`。
>
> `createDecorator`期望一个回调函数作为第一个参数，回调函数将接收以下参数:
>
> - `options`：Vue组件选项对象。此对象的更改将影响所提供的组件。
> - `key`：应用装饰器的属性或方法键。
> - `parameterIndex`：如果自定义装饰器用于参数，则为已装饰参数的索引。

创建**日志装饰器**的示例，当被装饰的方法被调用时，它打印带有方法名和传递参数的日志消息:

```js
// decorators.js
import { createDecorator } from 'vue-class-component'

// Declare Log decorator.
export const Log = createDecorator((options, key) => {
  // Keep the original method for later.
  const originalMethod = options.methods[key]

  // Wrap the method with the logging logic.
  options.methods[key] = function wrapperMethod(...args) {
    // Print a log.
    console.log(`Invoked: ${key}(`, ...args, ')')

    // Invoke the original method.
    originalMethod.apply(this, args)
  }
})
```

使用`Log`装饰器

```js
import Vue from 'vue'
import Component from 'vue-class-component'
import { Log } from './decorators'

@Component
class MyComp extends Vue {
  // It prints a log when `hello` method is invoked.
  @Log
  hello(value) {
    // ...
  }
}
```

在上面的代码中，当用`42`作为参数调用`hello`方法时，将打印以下日志:

```js
Invoked: hello( 42 )
```

## 三、继承（`extends`）和混入（`mixins`）

> Vue类组件提供了`mixins` 函数，以类样式的方式使用`mixins`。通过使用`mixins`函数，TypeScript可以推断出`mixin`类型，并在组件类型上继承它们

声明mixins `Hello`和`World`的例子:

```js
// mixins.js
import Vue from 'vue'
import Component from 'vue-class-component'

// You can declare mixins as the same style as components.
@Component
export class Hello extends Vue {
  hello = 'Hello'
}

@Component
export class World extends Vue {
  world = 'World'
}
```

在`class-style`组件中使用它们:

```js
import Component, { mixins } from 'vue-class-component'
import { Hello, World } from './mixins'

// Use `mixins` helper function instead of `Vue`.
// `mixins` can receive any number of arguments.
@Component
export class HelloWorld extends mixins(Hello, World) {
  created () {
    console.log(this.hello + ' ' + this.world + '!') // -> Hello World!
  }
}
```

和父类一样，所有的mixin都必须声明为类组件