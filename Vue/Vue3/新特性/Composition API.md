## 一、Composition API: 类似React Hooks

> 为vue应用提供更好的逻辑复用和代码组织
>
> 1. **ref** 定义基础数据类型的响应式数据
>
> 2. **reactive** 定义引用数据类型的响应式数据
>
> 3. **setup(props, { attrs, slots, emit, expose }) { … }** setup函数是 **Composition API** 的入口函数， `setup` 返回的所有内容都暴露给组件的其余部分 (计算属性、方法、生命周期钩子等等) 以及组件的模板。
>
>    - **attrs**：Attribute (非响应式对象，等同于 **$attrs**)
>
>    - **slots**：插槽 (非响应式对象，等同于 **$slots**)
>
>      > **`attrs`** 和 **`slots`** 是有状态的对象，它们总是会随组件本身的更新而更新。这意味着你应该避免对它们进行解构，并始终以 `attrs.x` 或 `slots.x` 的方式引用 property。
>      >
>      > **注意**：与 `props` 不同，`attrs` 和 `slots` 的 property 是**非**响应式的。如果你打算根据 `attrs` 或 `slots` 的更改应用副作用，那么应该在 `onBeforeUpdate` 生命周期钩子中执行此操作。
>
>    - **emit**：触发事件 (方法，等同于 $emit)
>
>    - **expose**：暴露公共 property (函数)

### setup选项

> **注意📢：**在 `setup` 中你应该避免使用 `this`，因为它不会找到组件实例，该选项在组件创建**之前**执行。`setup` 的调用发生在 **`data`** property、**`computed`** property 、 **`methods`** 、**`refs` (模板 ref)** 被解析之前，所以它们无法在 `setup` 中被获取，一旦 `props` 被解析，就将作为组合式 API 的入口。

```vue
<script>
  // ref函数只能监听简单类型的变化，不能监听复杂类型的变化(对象/数组)
  // isRef/isReactive分别监听是否为ref、reactive声明的数据对象
import { ref, reactive, isRef, isReactive, toRefs, toRef } from "vue";
export default {
  props: {
    title: String
  },
  // setup函数是组合API的入口函数,setup在beforeCreate生命钩子之前调用，所以无法使用data和methods，所以setup函数中this修改成了undefined
  setup(props, { attrs, slots, emit }) {
    // 因为 props 是响应式的，所以不能使用 ES6 解构，它会消除 prop 的响应性
    // 如果需要解构 prop，可以在 setup 函数中使用 toRefs 函数来完成此操作
    // 使用 `toRefs` 创建对 `props` 中的 `user` property 的响应式引用. 
    // 是为了确保我们的侦听器能够根据 user prop 的变化做出反应
    const { title } = toRefs(props);
    
    // 如果 title 是可选的 prop，则传入的 props 中可能没有 title 。
    // 在这种情况下，toRefs 将不会为 title 创建一个 ref 。你需要使用 toRef 替代它：
    const title = toRef(props, 'title')
  	console.log(title.value)
    
    // let count = 0; // 这样定义则数据不是响应式的
    // 定义了一个名叫count的变量，该变量初始值为0，该变量改变之后，vue自动更新UI
    // ref的本质还是reactive，当我们给ref传递一个值后，ref函数底层会自动将ref转换为reactive
    // ref(0) 变为 reactive({value:0}),所以赋值时应为 count.value = xx;
    let count = ref(0); 
      
    // 在组合API中，如果想定义方法，不用定义到methods中，直接定义即可
    function mf() {
      count.value += 1;
    }
     let state = reactive({
         stus:[]
     })
    // 注意：在组合API中定义的变量/方法，要想在外界中使用，必须通过return {xxx} 暴露出去
    return { count, mf, stus };
  }
};
</script>
```

<img src="../../../assets/vue3/对比.jpg" style="zoom: 80%;" />

#### 1. `ref`

> **`ref`** 接收参数并将其包裹在一个带有 **`value`** property 的对象中返回，然后可以使用该 property 访问或更改响应式变量的值

```js
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```

#### 2.`reactive`

#### 3. `watch` 响应式更改

> 从 Vue 导入的 `watch` 函数执行相同的操作。它接受 3 个参数：
>
> - 一个想要侦听的**响应式引用**或 getter 函数
> - 一个回调
> - 可选的配置选项

```js
import { ref, watch } from 'vue'

const counter = ref(0)
counter.value = 5;
watch(counter, (newValue, oldValue) => {
  console.log('The new counter value is: ' + counter.value)
})
// 监听多个数据
const data: IDataProps = reactive({
  count: 0,
  double: computed(() => data.count * 2),
  increase() {
    data.count++;
  },
});
watch([counter, data], (newV, oldV) => {
  console.log(newV, oldV); // oldV返回的是Proxy，不好处理
})
watch([counter, () => data.count], (newV, oldV) => { // 用函数改写来返回值
  console.log(newV, oldV);
})
```

#### 4. 独立的 `computed` 属性

> 与 `ref` 和 `watch` 类似，也可以使用从 Vue 导入的 `computed` 函数在 Vue 组件外部创建计算属性。让我们回到 counter 的例子：

```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

#### 5. 返回渲染函数

`setup` 还可以返回一个渲染函数，该函数可以直接使用在同一作用域中声明的响应式状态：

```js
// MyBook.vue

import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    // 请注意这里我们需要显式使用 ref 的 value
    return () => h('div', [readersNumber.value, book.title])
  }
}
```

返回一个渲染函数将阻止我们返回任何其它的东西。从内部来说这不应该成为一个问题，但当我们想要将这个组件的方法通过模板 ref 暴露给父组件时就不一样了。

我们可以通过调用 **`expose`** 来解决这个问题，给它传递一个对象，其中定义的 property 将可以被外部组件实例访问：

```js
import { h, ref } from 'vue'
export default {
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value

    expose({
      increment
    })

    return () => h('div', count.value)
  }
}

```

这个 `increment` 方法现在将可以通过父组件的模板 ref 访问。

#### 6. setup 内的生命周期钩子( 代替组件的生命周期钩子 )

> 组合式 API 上的生命周期钩子与选项式 API 的名称相同，但前缀为 **`on`**：即 **`mounted`** 看起来会像 **`onMounted`**
>
> 这些函数接受一个回调，当钩子被组件调用时，该回调将被执行	

```js
import { onMounted } from 'vue'
```

| 选项式 API        | `setup` 内的钩子                                             |
| :---------------- | ------------------------------------------------------------ |
| `beforeCreate`    | **Not needed**                                               |
| `created`         | **Not needed**                                               |
| `beforeMount`     | `onBeforeMount`                                              |
| `mounted`         | `onMounted`                                                  |
| `beforeUpdate`    | `onBeforeUpdate ` ：data选项中数据更新                       |
| `updated`         | `onUpdated` ：data选项中数据更新                             |
| `beforeUnmount`   | `onBeforeUnmount`                                            |
| `unmounted`       | `onUnmounted`                                                |
| `errorCaptured`   | `onErrorCaptured`                                            |
| `renderTracked`   | `onRenderTracked `：调试用的debug钩子函数                    |
| `renderTriggered` | `onRenderTriggered((event) => {})`：调试用的debug钩子函数，data选项中的数据更新时，会显示数据变化的状态信息 |
| `activated`       | `onActivated`                                                |
| `deactivated`     | `onDeactivated`                                              |

**注意📢：** <u>因为 `setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 `setup` 函数中编写</u>

这些函数**接受一个回调函数**，当钩子被组件调用时将会被执行:

```js
export default {
  setup() {
    onMounted(() => {
      console.log('Component is mounted!')
    })
  }
}
```

#### 7. 使用 Provide、inject

> **`provide`** 函数允许你通过两个参数定义 property：
>
> 1. name (`<String>` 类型)
> 2. value
>
> **`inject`** 函数有两个参数：
>
> 1. 要 inject 的 property 的 name
> 2. 默认值 (**可选**)

```vue
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref, readonly } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    /* provide('location', 'North Pole')
    provide('geolocation', {
      longitude: 90,
      latitude: 135
    }) */
    
    // 1. 为了增加 provide 值和 inject 值之间的响应性，我们可以在 provide 值时使用 ref 或 reactive。
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })
    provide('location', location)
    provide('geolocation', geolocation)
    
    // 2. 尽可能将对响应式 property 的所有修改限制在定义 provide 的组件内部。
    // 有时我们需要在注入数据的组件内部更新 inject 的数据。
    // 在这种情况下，我们建议 provide 一个方法来负责改变响应式 property。
    const updateLocation = () => {
      location.value = 'South Pole'
    }
    provide('updateLocation', updateLocation)

    // 3. 如果要确保通过 provide 传递的数据不会被 inject 的组件更改，建议对提供者的 property 使用 readonly。
    provide('location', readonly(location))
    provide('geolocation', readonly(geolocation))
    provide('updateLocation', updateLocation)
    
  },
  methods: {
    // 尽可能将对响应式 property 的所有修改限制在定义 provide 的组件内部。
    updateLocation() { 
      this.location = 'South Pole'
    }
  }
}
</script>
```

```vue
<!-- src/components/MyMarker.vue -->
<script>
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'The Universe')
    const userGeolocation = inject('geolocation')
    const updateUserLocation = inject('updateLocation')

    return {
      userLocation,
      userGeolocation,
      updateUserLocation
    }
  }
}
</script>
```

#### 8. 模板引用

在使用组合式 API 时，[响应式引用](https://v3.cn.vuejs.org/guide/reactivity-fundamentals.html#创建独立的响应式值作为-refs)和[模板引用](https://v3.cn.vuejs.org/guide/component-template-refs.html)的概念是统一的。为了获得对模板内元素或组件实例的引用，我们可以像往常一样声明 ref 并从 [setup()](https://v3.cn.vuejs.org/guide/composition-api-setup.html) 返回：

```vue
<template> 
  <div ref="root">This is a root element</div>
</template>
<script>
  import { ref, onMounted } from 'vue'
  export default {
    setup() {
      const root = ref(null)
      onMounted(() => {
        // DOM 元素将在初始渲染后分配给 ref
        console.log(root.value) // <div>This is a root element</div>
      })
      return {
        root
      }
    }
  }
</script>
```

这里我们在渲染上下文中暴露 `root`，并通过 `ref="root"`，将其绑定到 div 作为其 ref。在虚拟 DOM 补丁算法中，如果 VNode 的 `ref` 键对应于渲染上下文中的 ref，则 VNode 的相应元素或组件实例将被分配给该 ref 的值。这是在虚拟 DOM 挂载/打补丁过程中执行的，因此模板引用只会在初始渲染之后获得赋值。

作为模板使用的 ref 的行为与任何其他 ref 一样：它们是响应式的，可以传递到 (或从中返回) 复合函数中。

##### JSX 中的用法

```js
export default {
  setup() {
    const root = ref(null)
    return () =>
      h('div', {
        ref: root
      })
    // with JSX
    return () => <div ref={root} />
  }
}
```

##### `v-for` 中的用法

组合式 API 模板引用在 `v-for` 内部使用时没有特殊处理。相反，请使用函数引用执行自定义处理：

```vue
<template>
  <div v-for="(item, i) in list" :ref="el => { if (el) divs[i] = el }">
    {{ item }}
  </div>
</template>
<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'
  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])
      // 确保在每次更新之前重置ref
      onBeforeUpdate(() => {
        divs.value = []
      })
      return {
        list,
        divs
      }
    }
  }
</script>
```

##### 侦听模板引用

侦听模板引用的变更可以替代前面例子中演示使用的生命周期钩子。

但与生命周期钩子的一个关键区别是，**`watch()`** 和 **`watchEffect()`** 在 DOM 挂载或更新*之前*运行副作用，所以当侦听器运行时，模板引用还未被更新。

```vue
<template>
  <div ref="root">This is a root element</div>
</template>
<script>
  import { ref, watchEffect } from 'vue'
  export default {
    setup() {
      const root = ref(null)
      
      watchEffect(() => {
        // 这个副作用在 DOM 更新之前运行，因此，模板引用还没有持有对元素的引用。
        console.log(root.value) // => null
      })
      
      // 使用模板引用的侦听器应该用 flush: 'post' 选项来定义，
      // 这将在 DOM 更新后运行副作用，确保模板引用与 DOM 保持同步，并引用正确的元素。
      watchEffect(() => {
        console.log(root.value) // => <div>This is a root element</div>
      }, 
      {
        flush: 'post'
      })
      
      return {
        root
      }
    }
  }
</script>
```





## 二、Composition API和Options API(vue2.x中的使用方式)的混合使用

```vue
<script>
    export default{
        data(){
            return{}
        },
        methods:{},
        setup(){
            // 本质上setup中定义的数据会被注入到data、methods中
            return {...};
        }
    }
</script>
```

## 三、Composition API 的优点

1. **更灵活的逻辑组织、抽象、复用能力**
2. **没有this**
3. **更好的类型推导能力（TypeScript）**
4. **更友好的 `Tree-shaking`支持，渐进式体验**
5. **更大的代码压缩空间**
