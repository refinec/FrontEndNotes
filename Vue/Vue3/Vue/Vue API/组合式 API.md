## 一、setup()





## 二、依赖注入

### provide()

> 提供一个值，可以被后代组件注入。
>
> ```ts
> function provide<T>(key: InjectionKey<T> | string, value: T): void
> ```
>
> - 参数一：是要注入的 key，可以是一个**字符串**或者一个 **symbol**
> - 参数二：要注入的值

与注册生命周期钩子的 API 类似，`provide()` 必须在组件的 `setup()` 阶段同步调用

```vue
<script setup>
import { ref, provide } from 'vue'
//import { fooSymbol } from './injectionSymbols'
import type { InjectionKey } from 'vue'
const fooSymbol = Symbol() as InjectionKey<string>

// 1.提供静态值
provide('foo', 'bar')

// 2.提供响应式的值
const count = ref(0)
provide('count', count)

// 3.提供时将 Symbol 作为 key
provide(fooSymbol, count)
  
// 4.如果你想确保提供的数据不能被注入方的组件更改，你可以使用 readonly() 来包装提供的值。
provide('read-only-count', readonly(count))
</script>
```

### inject()

> 注入一个由祖先组件或整个应用 ( `app.provide()`) 提供的值
>
> ```ts
> // 没有默认值
> function inject<T>(key: InjectionKey<T> | string): T | undefined
> 
> // 带有默认值
> function inject<T>(key: InjectionKey<T> | string, defaultValue: T): T
> 
> // 使用工厂函数
> function inject<T>(
>   key: InjectionKey<T> | string,
>   defaultValue: () => T,
>   treatDefaultAsFactory: true
> ): T
> ```
>
> 参数一：**注入的 key**。Vue 会遍历父组件链，通过匹配 key 来确定所提供的值。如果父组件链上多个组件对同一个 key 提供了值，那么离得更近的组件将会“覆盖”链上更远的组件所提供的值。如果没有能通过 key 匹配到值，`inject()` 将返回 `undefined`，除非提供了一个默认值。
>
> 参数二：在没有匹配到 key 时使用的**默认值**。它也可以是一个**工厂函数**，用来返回某些创建起来比较复杂的值。**如果默认值本身就是一个函数**，那么你必须将 `false` 作为<u>第三个参数</u>传入，表明这个函数就是默认值，而不是一个工厂函数。

与注册生命周期钩子的 API 类似，`inject()` 必须在组件的 `setup()` 阶段同步调用。

```vue
<script setup>
import { inject } from 'vue'
import { fooSymbol } from './injectionSymbols'

// 注入值的默认方式
const foo = inject('foo')

// 注入响应式的值
const count = inject('count')

// 通过 Symbol 类型的 key 注入
const foo2 = inject(fooSymbol)

// 注入一个值，若为空则使用提供的默认值
const bar = inject('foo', 'default value')

// 注入一个值，若为空则使用提供的工厂函数
const baz = inject('foo', () => new Map())

// 注入时为了表明提供的默认值是个函数，需要传入第三个参数
const fn = inject('function', () => {}, false)
</script>
```

## 三、生命周期钩子





## 四、响应式：核心





## 五、响应式：工具





## 六、响应式: 进阶

### EffectScope 副作用域

> ```ts
> interface EffectScope {
>   run<T>(fn: () => T): T | undefined // 如果作用域不活跃就为 undefined
>   stop(): void
> }
> function effectScope(detached?: boolean): EffectScope
> ```

```ts
const scope = effectScope()

scope.run(() => {
  const doubled = computed(() => counter.value * 2)

  watch(doubled, () => console.log(doubled.value))

  watchEffect(() => console.log('Count: ', doubled.value))
})

// 停用掉当前作用域内的所有 effect
// 当 scope.stop ()被调用时，它将递归地停止所有 捕获的effects 和 嵌套的scopes。
scope.stop()
```

```ts
console.log(scope.run(() => 1)) // 1 run方法还转发已执行函数的返回值
```

#### EffectScope还能再嵌套

> 嵌套作用域由其父作用域收集。当父作用域被释放时，它的所有子作用域也将被停止，**除非使用`detached`分离参数**。

```ts
const scope = effectScope()

scope.run(() => {
  const doubled = computed(() => counter.value * 2)

  effectScope().run(() => {
    watch(doubled, () => console.log(doubled.value))
  })

  watchEffect(() => console.log('Count: ', doubled.value))
})

// 停用所有effects, 包括嵌套作用域中的effects
scope.stop()
```

**使用`detached`分离参数**

```ts
let nestedScope

const parentScope = effectScope()

parentScope.run(() => {
  const doubled = computed(() => counter.value * 2)

  // 不被父作用域收集和处理
  nestedScope = effectScope(true /* detached */)
  nestedScope.run(() => {
    watch(doubled, () => console.log(doubled.value))
  })

  watchEffect(() => console.log('Count: ', doubled.value))
})

// 停用所有effects, 但不包括nestedScope
parentScope.stop()

// 停用nestedScope
nestedScope.stop()
```

### getCurrentScope()

> 返回当前活跃的 **effect 作用域**
>
> ```ts
> function getCurrentScope(): EffectScope | undefined
> ```

```ts
import { getCurrentScope } from 'vue'

getCurrentScope() // EffectScope | undefined
```

### onScopeDispose()

> 当相关的 effect 作用域停止时会调用这个回调函数，可以作为**可复用的**组合式函数中 `onUnmounted` 的替代品。
>
> 它并不与组件耦合，因为每一个 Vue 组件的 `setup()` 函数也是在一个 effect 作用域中调用的。
>
> ```ts
> function onScopeDispose(fn: () => void): void
> ```

全局钩子 `onScopeDispose ()`具有与 `onUnmount ()`类似的功能，但是它适用于当前作用域而不是组件实例。这有利于可组合函数清除它们的副作用及其范围。由于 setup ()也为组件创建了一个作用域，所以当没有创建显式effects作用域时，它将等效于 `onUnmount ()`。

```ts
import { onScopeDispose } from 'vue'

const scope = effectScope()

scope.run(() => {
  onScopeDispose(() => {
    console.log('cleaned!')
  })
})

scope.stop() // logs 'cleaned!'
```

**示例**

```ts
function useMouse() {
  const x = ref(0)
  const y = ref(0)

  function handler(e) {
    x.value = e.x
    y.value = e.y
  }

  window.addEventListener('mousemove', handler)

  //onUnmounted(() => {
  onScopeDispose(() => {
    window.removeEventListener('mousemove', handler)
  })

  return { x, y }
}
```

```ts
export default {
  setup() {
    const enabled = ref(false)
    let mouseState, mouseScope

    const dispose = () => {
      mouseScope && mouseScope.stop()
      mouseState = null
    }

    watch(
      enabled,
      () => {
        if (enabled.value) {
          mouseScope = effectScope()
          mouseState = mouseScope.run(() => useMouse())
        } else {
          dispose()
        }
      },
      { immediate: true }
    )

    onScopeDispose(dispose) // 这仍然有效，因为 Vue 组件现在还在作用域内运行它的 setup()，当组件卸载时将释放这个作用域。
  },
}
```

