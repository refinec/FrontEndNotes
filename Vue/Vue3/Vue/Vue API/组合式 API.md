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



### onErrorCaptured()

> 捕获了后代组件传递的错误时调用
>
> ```ts
> function onErrorCaptured(callback: ErrorCapturedHook): void
> 
> type ErrorCapturedHook = (
>   err: unknown,
>   instance: ComponentPublicInstance | null,
>   info: string
> ) => boolean | void
> ```

注意不要让错误状态再次渲染导致本次错误的内容，否则组件会陷入无限循环。这个钩子可以通过返回 `false` 来阻止错误继续向上传递。

**错误传递规则**

- 默认情况下，所有的错误都会被发送到应用级的 [`app.config.errorHandler`](https://cn.vuejs.org/api/application.html#app-config-errorhandler) (前提是这个函数已经定义)，这样这些错误都能在一个统一的地方报告给分析服务。
- 如果组件的继承链或组件链上存在多个 `errorCaptured` 钩子，对于同一个错误，这些钩子会被按从底至上的顺序一一调用。这个过程被称为“向上传递”，类似于原生 DOM 事件的冒泡机制。
- 如果 `errorCaptured` 钩子本身抛出了一个错误，那么这个错误和原来捕获到的错误都将被发送到 `app.config.errorHandler`。
- `errorCaptured` 钩子可以通过返回 `false` 来阻止错误继续向上传递。即表示“这个错误已经被处理了，应当被忽略”，它将阻止其他的 `errorCaptured` 钩子或 `app.config.errorHandler` 因这个错误而被调用。

### onRenderTracked()

> 一个调试钩子，当组件渲染过程中追踪到响应式依赖时调用。
>
> **这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用。**
>
> ```ts
> type DebuggerEvent = {
>   effect: ReactiveEffect
>   target: object
>   type: TrackOpTypes /* 'get' | 'has' | 'iterate' */
>   key: any
> }
> 
> type DebuggerHook = (e: DebuggerEvent) => void
> 
> function onRenderTracked(callback: DebuggerHook): void
> ```

### onRenderTriggered()

> 一个调试钩子，当响应式依赖的变更触发了组件渲染时调用
>
> **这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用。**
>
> ```ts
> type DebuggerEvent = {
>   effect: ReactiveEffect
>   target: object
>   type: TriggerOpTypes /* 'set' | 'add' | 'delete' | 'clear' */
>   key: any
>   newValue?: any
>   oldValue?: any
>   oldTarget?: Map<any, any> | Set<any>
> }
> 
> type DebuggerHook = (e: DebuggerEvent) => void
> 
> function onRenderTriggered(callback: DebuggerHook): void
> ```

### onServerPrefetch()

> 一个异步函数，在组件实例在服务器上被渲染之前调用
>
> ```ts
> function onServerPrefetch(callback: () => Promise<any>): void
> ```

- 如果这个钩子返回了一个 Promise，服务端渲染会在渲染该组件前等待该 Promise 完成。

  这个钩子仅会在服务端渲染中执行，可以用于执行一些仅存在于服务端的数据抓取过程。

  ```vue
  <script setup>
  import { ref, onServerPrefetch, onMounted } from 'vue'
  
  const data = ref(null)
  
  onServerPrefetch(async () => {
    // 组件作为初始请求的一部分被渲染
    // 在服务器上预抓取数据，因为它比在客户端上更快。
    data.value = await fetchOnServer(/* ... */)
  })
  
  onMounted(async () => {
    if (!data.value) {
      // 如果数据在挂载时为空值，这意味着该组件
      // 是在客户端动态渲染的。将转而执行
      // 另一个客户端侧的抓取请求
      data.value = await fetchOnClient(/* ... */)
    }
  })
  </script>
  ```

## 四、响应式：核心



### watch()

> 侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。
>
> ```ts
> type WatchCallback<T> = (
>   value: T,
>   oldValue: T,
>   onCleanup: (cleanupFn: () => void) => void
> ) => void
> 
> type WatchSource<T> =
>   | Ref<T> // ref
>   | (() => T) // getter
>   | T extends object
>   ? T
>   : never // 响应式对象
> 
> interface WatchOptions extends WatchEffectOptions {
>   immediate?: boolean // 默认：false
>   deep?: boolean // 默认：false
>   flush?: 'pre' | 'post' | 'sync' // 默认：'pre'
>   onTrack?: (event: DebuggerEvent) => void
>   onTrigger?: (event: DebuggerEvent) => void
> }
> 
> // 侦听单个来源
> function watch<T>(
>   source: WatchSource<T>,
>   callback: WatchCallback<T>,
>   options?: WatchOptions
> ): StopHandle
> 
> // 侦听多个来源
> function watch<T>(
>   sources: WatchSource<T>[],
>   callback: WatchCallback<T[]>,
>   options?: WatchOptions
> ): StopHandle
> ```

1. 参数一：`watch()`的侦听源可以是以下几种：

   - 一个函数，返回一个值

   - 一个 ref

   - 一个响应式对象（reactive）

   - ...或是由以上类型的值组成的数组

2. 参数二：这个回调函数接受三个参数：新值、旧值，以及一个用于注册副作用清理的回调函数。

   该回调函数会在副作用下一次重新执行前调用，可以用来清除无效的副作用，例如等待中的异步请求。

3. 参数三(可选)：配置选项
   - **`immediate`**：在侦听器创建时立即触发回调。第一次调用时旧值是 `undefined`。
   - **`deep`**：如果源是对象，强制深度遍历，以便在深层级变更时触发回调。参考[深层侦听器](https://cn.vuejs.org/guide/essentials/watchers.html#deep-watchers)。
   - **`flush`**：调整回调函数的刷新时机(默认情况下，用户创建的侦听器回调，都会在 Vue 组件更新**之前**被调用。这意味着你在侦听器回调中访问的 DOM 将是被 Vue 更新之前的状态)。参考[回调的刷新时机](https://cn.vuejs.org/guide/essentials/watchers.html#callback-flush-timing)及 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect)。
   - **`onTrack / onTrigger`**：调试侦听器的依赖。参考[调试侦听器](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#watcher-debugging)。

与 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect) 相比，`watch()` 使我们可以：

- 懒执行副作用；
- 更加明确是应该由哪个状态触发侦听器重新执行；
- 可以访问所侦听状态的前一个值和当前值。

```ts
// 1. 侦听一个 getter 函数
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {}
)

// 2. 侦听一个 ref
const count = ref(0)
watch(count, (count, prevCount) => {})

// 3. 侦听多个来源
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {})
```

当使用 getter 函数作为源时，回调只在此函数的返回值变化时才会触发。如果你想让回调在深层级变更时也能触发，你需要使用 `{ deep: true }` 强制侦听器进入深层级模式。在深层级模式时，如果回调函数由于深层级的变更而被触发，那么新值和旧值将是同一个对象。

```ts
const state = reactive({ count: 0 })
watch(
  () => state,
  (newValue, oldValue) => {
    // newValue === oldValue
    // 在嵌套的属性变更时触发
  	// 注意：newValue 此处和 oldValue 是相等的
  	// 因为它们是同一个对象！
  },
  { deep: true }
)
```

```ts
// 当直接侦听一个响应式对象时，侦听器会自动启用深层模式
const state = reactive({ count: 0 })
watch(state, () => {
  /* 深层级变更状态所触发的回调 */
})

// 一个返回响应式对象的 getter 函数，只有在返回不同的对象时，才会触发回调：
watch(
  () => state.someObject,
  () => {
    // 仅当 state.someObject 被替换时触发
  }
)

// 显式地加上 deep 选项，强制转成深层侦听器
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // *除非* state.someObject 被整个替换了
  },
  { deep: true }
)

// watch() 和 watchEffect() 享有相同的刷新时机和调试选项：
watch(source, callback, {
  flush: 'post',
  onTrack(e) {
    debugger
  }
})
```



### watchEffect()

> 立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行。
>
> ```ts
> type OnCleanup = (cleanupFn: () => void) => void
> 
> interface WatchEffectOptions {
>   flush?: 'pre' | 'post' | 'sync' // 默认：'pre'
>   onTrack?: (event: DebuggerEvent) => void
>   onTrigger?: (event: DebuggerEvent) => void
> }
> 
> type StopHandle = () => void
> 
> function watchEffect(
>   effect: (onCleanup: OnCleanup) => void,
>   options?: WatchEffectOptions
> ): StopHandle
> ```

* **第一个参数**：就是要运行的副作用函数。这个副作用函数的参数也是一个函数，用来注册清理回调。

  清理回调会在该副作用下一次执行前被调用，可以用来清理无效的副作用，例如等待中的异步请求。

* **第二个参数**：用来调整**副作用的刷新时机**或调试副作用的依赖。默认情况下，侦听器将在组件渲染之前执行。

  设置 `flush: 'post'` 将会使侦听器延迟到组件渲染之后再执行。详见[回调的触发时机](https://cn.vuejs.org/guide/essentials/watchers.html#callback-flush-timing)。

  在某些特殊情况下 (例如要使缓存失效)，可能有必要在响应式依赖发生改变时立即触发侦听器。这可以通过设置 `flush: 'sync'` 来实现。然而，该设置应谨慎使用，因为如果有多个属性同时更新，这将导致一些性能和数据一致性的问题。

* **返回值**：是一个用来停止该副作用的函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> 输出 0

count.value++
// -> 输出 1
```

```ts
// 回调会立即执行。在执行期间，它会自动追踪 url.value 作为依赖（和计算属性的行为类似）。
// 每当 url.value 变化时，回调会再次执行。
watchEffect(async () => {
  const response = await fetch(url.value)
  data.value = await response.json()
})

```

**注意📢**：`watchEffect` 仅会在其**同步**执行期间，才追踪依赖。在使用异步回调时，只有在第一个 `await` 正常工作前访问到的属性才会被追踪。



副作用清除：

```js
watchEffect(async (onCleanup) => {
  const { response, cancel } = doAsyncWork(id.value)
  // `cancel` 会在 `id` 更改时调用
  // 以便取消之前
  // 未完成的请求
  onCleanup(cancel)
  data.value = await response
})
```

选项：

```js
watchEffect(() => {}, {
  flush: 'post',
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})
```

停止侦听器：

> 在 `setup()` 或 `<script setup>` 中用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件卸载时自动停止。
>
> 但是，关键点是，侦听器必须用**同步**语句创建：如果用异步回调创建一个侦听器，那么它不会绑定到当前组件上，你必须手动停止它，以防内存泄漏。如下方这个例子：

```vue
<script setup>
import { watchEffect } from 'vue'

// 它会自动停止
watchEffect(() => {})

// ...这个则不会！
setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```

必须手动停止一个侦听器，请调用 `watch` 或 `watchEffect` 返回的函数：

```js
const unwatch = watchEffect(() => {})

// ...当该侦听器不再需要时
unwatch()
```

如果需要等待一些异步数据，你可以使用条件式的侦听逻辑：

```js
// 需要异步请求得到的数据
const data = ref(null)

watchEffect(() => {
  if (data.value) {
    // 数据加载后执行某些操作...
  }
})
```

#### watchPostEffect()

> [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect) 使用 `flush: 'post'` 选项时的别名。

#### watchSyncEffect()

> [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect) 使用 `flush: 'sync'` 选项时的别名。

### `watch` vs. `watchEffect`

它们之间的主要区别是**追踪响应式依赖的方式**：

- `watch` 只追踪明确侦听的数据源。

  它不会追踪任何在回调中访问到的东西。另外，仅在数据源确实改变时才会触发回调。`watch` 会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。

- `watchEffect`，则会在副作用发生期间追踪依赖。

  它会在**同步执行过程**中，自动追踪所有能访问到的响应式属性。这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。

## 五、响应式：工具

### isProxy()

> 检查一个对象是否是由 [`reactive()`](https://cn.vuejs.org/api/reactivity-core.html#reactive)、[`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly)、[`shallowReactive()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreactive) 或 [`shallowReadonly()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreadonly) 创建的代理。
>
> ```ts
> function isProxy(value: unknown): boolean
> ```

### isReactive()

> 检查一个对象是否是由 [`reactive()`](https://cn.vuejs.org/api/reactivity-core.html#reactive) 或 [`shallowReactive()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreactive) 创建的代理。
>
> ```ts
> function isReactive(value: unknown): boolean
> ```

### isReadonly()

> 检查一个对象是否是由 [`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly) 或 [`shallowReadonly()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreadonly) 创建的代理。
>
> ```ts
> function isReadonly(value: unknown): boolean
> ```

## 六、响应式: 进阶

### shallowReadonly()

> [`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly) 的浅层作用形式，和 `readonly()` 不同，这里没有深层级的转换：只有根层级的属性变为了只读。属性的值都会被原样存储和暴露，这也意味着值为 ref 的属性**不会**被自动解包了。
>
> ```ts
> function shallowReadonly<T extends object>(target: T): Readonly<T>
> ```

```ts
const state = shallowReadonly({
  foo: 1,
  nested: {
    bar: 2
  }
})

// 更改状态自身的属性会失败
state.foo++

// ...但可以更改下层嵌套对象
isReadonly(state.nested) // false

// 这是可以通过的
state.nested.bar++
```

### markRaw()

> 将一个对象标记为不可被转为响应式。返回该对象本身。
>
> ```ts
> function markRaw<T extends object>(value: T): T
> ```

```ts
const foo = markRaw({})
console.log(isReactive(reactive(foo))) // false

// 也适用于嵌套在其他响应性对象
const bar = reactive({ foo })
console.log(isReactive(bar.foo)) // false
```

**使用原因**：

- 有些值不应该是响应式的，例如复杂的第三方类实例或 Vue 组件对象。
- 当呈现带有不可变数据源的大型列表时，跳过代理转换可以提高性能。

只在根层访问能到原始值，所以如果把一个嵌套的、没有标记的原始对象设置成一个响应式对象，然后再次访问它，你获取到的是代理的版本。这可能会导致**对象身份风险**，即执行一个依赖于对象身份的操作，但却同时使用了同一对象的原始版本和代理版本：

```ts
const foo = markRaw({
  nested: {}
})

const bar = reactive({
  // 尽管 `foo` 被标记为了原始对象，但 foo.nested 却没有
  nested: foo.nested
})

console.log(foo.nested === bar.nested) // false
```

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

