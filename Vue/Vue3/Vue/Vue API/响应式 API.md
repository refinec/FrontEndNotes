## 一、computed 计算属性

### 函数形式

> computed() 函数的返回值是一个 **`ref`** 的实例。

```js
import { computed, reactive, ref } from 'vue'
let price = ref(0)//$0
 
let m = computed<string>(()=>{
   return `$` + price.value
})

price.value = 500
console.log('m ==>', m.value)
```

### 对象形式

```vue
<template>
	<div>{{ mul }}</div>
	<div @click="mul = 100">click</div>
</template>

<script setup lang="ts">
  import { computed, ref } from 'vue'
  let price = ref<number | string>(1)//$0
  let mul = computed({
    get: () => {
      return price.value
    },
    set: (value) => {
      price.value = 'set' + value
    }
  })
</script>
```

### 购物车案例

```vue
<template>
   <div>
      <table style="width:800px" border>
         <thead>
            <tr>
               <th>名称</th>
               <th>数量</th>
               <th>价格</th>
               <th>操作</th>
            </tr>
         </thead>
         <tbody>
            <tr :key="index" v-for="(item, index) in data">
               <td align="center">{{ item.name }}</td>
               <td align="center">
                  <button @click="AddAnbSub(item, false)">-</button>
                  {{ item.num }}
                  <button @click="AddAnbSub(item, true)">+</button>
               </td>
               <td align="center">{{ item.num * item.price }}</td>
               <td align="center">
                  <button @click="del(index)">删除</button>
               </td>
            </tr>
         </tbody>
         <tfoot>
            <tr>
               <td></td>
               <td></td>
               <td></td>
               <td align="center">总价:{{ $total }}</td>
            </tr>
         </tfoot>
      </table>
   </div>
</template>
 
<script setup lang="ts">
import { computed, reactive, ref } from 'vue'
type Shop = {
   name: string,
   num: number,
   price: number
}
let $total = ref<number>(0)
const data = reactive<Shop[]>([
   {
      name: "小满的袜子",
      num: 1,
      price: 100
   },
   {
      name: "小满的裤子",
      num: 1,
      price: 200
   },
   {
      name: "小满的衣服",
      num: 1,
      price: 300
   },
   {
      name: "小满的毛巾",
      num: 1,
      price: 400
   }
])
 
const AddAnbSub = (item: Shop, type: boolean = false): void => {
   if (item.num > 1 && !type) {
      item.num--
   }
   if (item.num <= 99 && type) {
      item.num++
   }
}
const del = (index: number) => {
   data.splice(index, 1)
}
 
$total = computed<number>(() => {
   return data.reduce((prev, next) => {
      return prev + (next.num * next.price)
   }, 0)
})

</script>
```

## 二、watch 与 watchEffect 侦听器

### 1.watch 侦听器

> 参数一：监听源
>
> 参数二：回调函数(newV, oldV)
>
> 参数三：配置选项 { immediate: true /* 是否立即调用一次 */, deep: true / * 是否开启深度监听 */}

```js
import { ref, watch } from 'vue'
 
let message = ref({
    nav:{
        bar:{
            name:""
        }
    }
})
 
watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
},{
    immediate:true,
    deep:true
})
```

#### 监听多个ref

```js
import { ref, watch ,reactive} from 'vue'
 
let message = ref('')
let message2 = ref('')
 
watch([message,message2], (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

#### 使用reactive监听深层对象开启和不开启deep 效果一样

```js
import { ref, watch ,reactive} from 'vue'
 
let message = reactive({
    nav:{
        bar:{
            name:""
        }
    }
})
 
 
watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

#### 监听reactive 单一值

```js
import { ref, watch ,reactive} from 'vue'
 
let message = reactive({
    name:"",
    name2:""
})
 
 
watch(()=> message.name, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

### 2.watchEffect 高级侦听器

> `watchEffect` 它与 `watch` 的区别主要有以下几点：
>
> 1. 不需要指定监听的属性，会自动的收集依赖（初始化时会执行一次回调函数来自动获取依赖，与`computed`同理）
> 2. 无法获取旧值，只能得到变化后的新值

`watchEffect`的类型定义：

```ts
function watchEffect(
  effect: (onInvalidate: InvalidateCbRegistrator) => void, // 回调函数(包含副作用的函数)
  options?: WatchEffectOptions 														 // 配置项
): StopHandle;

interface WatchEffectOptions {
  flush?: "pre" | "post" | "sync";
  onTrack?: (event: DebuggerEvent) => void;
  onTrigger?: (event: DebuggerEvent) => void;
}

interface DebuggerEvent {
  effect: ReactiveEffect;
  target: any;
  type: OperationTypes;
  key: string | symbol | undefined;
}

type InvalidateCbRegistrator = (invalidate: () => void) => void; // 用于清除 effect 产生的副作用

type StopHandle = () => void;
```

监听属性，用到几个监听几个，而且是非惰性，会默认调用一次

```js
let message = ref<string>('')
let message2 = ref<string>('')
 watchEffect(() => {
    //console.log('message', message.value);
    console.log('message2', message2.value);
})
```

#### onInvalidate 清除副作用

`onInvalidate` 只作用于异步函数，并且只有在如下两种情况下才会被调用：

1. 当 `effect` 函数被重新调用时
2. 当监听器被注销时（如组件被卸载了）

如下代码中，`onInvalidate` 会在 **`id` 改变**时 或 **停止侦听**时，取消之前的异步操作（`asyncOperation`）：

```javascript
/*
	场景：
	假设我们现在用一个用户ID去查询用户的详情信息，然后我们监听了这个用户ID， 当用户ID 改变的时候我们就会去发起一次请求，这很简	单，用watch 就可以做到。 但是如果在请求数据的过程中，我们的用户ID发生了多次变化，那么我们就会发起多次请求，而最后一次返回的数据将会覆盖掉我们之前返回的所有用户详情。这不仅会导致资源浪费，还无法保证 watch 回调执行的顺序。而使用 watchEffect 我们就可以做到。
*/
import { asyncOperation } from "./asyncOperation";
const id = ref(0);
watchEffect((onInvalidate) => {
  const token = asyncOperation(id.value);
  // 在回调函数中，onInvalidate会先执行
  onInvalidate(() => {
    // run if id has changed or watcher is stopped
    token.cancel();
  });
});
```

#### 停止监听

> watchEffect 返回一个函数，调用之后将停止监听并清理副作用

```js
const stop =  watchEffect((oninvalidate) => {
    console.log('message2', message2.value);
})
stop() //  stopHandle 可以在 setup 函数里显式调用，也可以在组件被卸载时隐式调用。
```

#### 参数二 配置项

* `flush`

  决定**副作用(回调函数)调用时机，`flush` 一般使用post**(默认)

  |          | pre                | sync                 | post               |
  | :------- | :----------------- | :------------------- | :----------------- |
  | 更新时机 | 组件**更新前**执行 | 强制效果始终同步触发 | 组件**更新后**执行 |

* `onTrigger、onTrack`

  这两个选项可用于调试一个侦听器的行为（当然只开发阶段有效）

  **onTrigger 可以帮助我们调试 watchEffect**
  
  ```js
  import { watchEffect, ref } from 'vue'
  let message = ref<string>('')
  let message2 = ref<string>('')
   watchEffect((oninvalidate) => {
      //console.log('message', message.value);
      oninvalidate(()=>{
   
      })
      console.log('message2', message2.value);
  },{
      flush:"post",
      onTrigger (e) {
          debugger
      }
  })
  ```
  

#### 注意点

`watchEffect` 会在 Vue3 开发中大量使用，这里说几个注意点：

1. 如果有多个副作用，不要粘合在一起，建议写多个 `watchEffect`。

   

   ```javascript
   watchEffect(() => {
     setTimeout(() => console.log(a.val + 1), 1000);
     setTimeout(() => console.log(b.val + 1), 1000);
   });
   ```

   这两个 setTimeout 是两个不相关的副作用，不需要同时监听 a 和 b，分开写吧：

   ```javascript
   watchEffect(() => {
     setTimeout(() => console.log(a.val + 1), 1000);
   });
   
   watchEffect(() => {
     setTimeout(() => console.log(b.val + 1), 1000);
   });
   ```

2. `watchEffect` 也可以放在其他生命周期函数内

   比如你的副作用函数在首次执行时就要调用 DOM，你可以把他放在 `onMounted` 钩子里：

   ```javascript
   onMounted(() => {
     watchEffect(() => {
       // access the DOM or template refs
     });
   }
   ```

## 三、reactive 全家桶

### reactive

> 等价于 vue 2.x 中的 `Vue.observable()` 函数
>
> 用来绑定复杂的数据类型，例如：对象、数组

```vue
<script setup lang="ts">
import { reactive } from 'vue'
const person = reactive<number[]>([])
// person = [1, 2, 3]; // 不能直接赋值，会脱离响应式，因为person的引用地址被改变了
person.push(...arr);
  
/* 或者下面这样写 */
type Person = {
  list?: Array<number>
};
const person = reactive<Person>({
  list: []
})
person.list = [1, 2, 3];
</script>
```

### readonly

> 拷贝一份 **proxy**对象将其设置为只读

```vue
<script setup lang="ts">
import { reactive, readonly } from "vue"

const person = reactive({
  name: "John",
  age: 30
})
const cp = readonly(person)
</script>
```

### shallowReactive

> 只能对浅层(第一级)的数据作为响应式对象，如果是深层(第二级及之后)的数据只会改变值，不会改变视图

什么时候用 **`shallowReactive`**? 如果有一个对象数据，结构比较深, 但只有最外层属性值发生改变，那就用 **`shallowReactive`**

```vue
<template>
  <div>
    name: {{ person.name }}
    group: {{ person.group }}
    <button @click="testShalldowRe">测试 shallowReactive</button>
  </div>
</template>
<script setup lang="ts">
import { shallowReactive } from "vue"

const person = shallowReactive({
  name: "first",
  group: {
    name: "John",
    age: 30
  },
})
const testShalldowRe = () => {
  // 注意📢：浅层和深层的数据不要在一个'逻辑块'中同时修改，否则 shallowReactive 将失去应有的作用，深层的数据修改会同步到视图上
  // person.name = "second"; 
  
  person.group.name = "xxxxxx"; // 深沉的数据更新不会同步到视图上
}
</script>
```

## 四、ref 全家桶

> 共有3组
>
> * `ref,` `Ref`, `isRef`，`unref`;
> * `shallowRef`, `triggerRef`; 
> * `customRef`

### ref、Ref

> 包装对象 **`ref()`** 的意义在于提供一个让我们能够在函数之间以引用的方式传递任意类型值的容器
>
> **`ref()`** 返回的是一个 `value reference` （**包装对象**），一个包装对象只有一个属性：`value` 

**注意📢：**当包装对象被暴露给**模版渲染上下文**，或是被**嵌套在另一个响应式对象中**的时候，包装对象 `value` 属性自动展开(即我们不再需要用`.value`来获取值)

```vue
<template>
  <div>reactive ref count: {{state.num}}</div>
</template>

<script setup lang='ts'>
import { reactive, ref } from '@vue/composition-api'
const count: Ref<number> = ref(0)
const state = reactive({
  num: count // 包装对象 value 属性自动展开
})
</script>
```

用 `ref()` 包装对象引用**页面元素**、**组件**:

```vue
<template>
  <div><p ref="text">Hello</p></div>
</template>

<script setup lang="ts">
import { ref, Ref, onMounted } from '@vue/composition-api'
const text: Ref<HTMLElement | undefined> = ref()
onMounted(() => {
  console.log('text: ', text.value?.innerHTML)
})
</script>
```

### isRef

> 检查一个值是否为一个 ref 对象

```vue
<script>
  let message: Ref<string> = ref("Hello World")
  let notRef: string = "Hello World"
  console.log('是否为Ref响应式类型：', isRef(message), isRef(notRef)) // true false
</script>
```

### unref

> 如果参数是一个 `ref` 则返回它的 `value`，否则返回参数本身
>
> **unref()**：是 `val = isRef(val) ? val.value : val` 的语法糖

```vue
<script setup>
const valueRef = ref('');
const value = unref(valueRef);
if (!value) {
   console.warning('请输入要拷贝的内容！');
   return;
}  
<script>
```

### shallowRef、triggerRef

> **`shallowRef`**：浅响应式，只处理**基本数据类型**的响应式, 不进行**引用类型**的响应式处理
>
> 1. 如果传入的是**基本数据类型**，那么跟 `ref` 没区别
> 2. 如果传入的是**引用类型**，`.value`值将不会是响应式的数据
>
> **`triggerRef`**: 手动执行与 `shallowRef` 关联的任何副作用，当 `shallowRef` 传入的是引用类型的数据的时候使用

什么时候用 **`shallowRef`**? 如果有一个对象数据，后续功能不会修改该对象中的属性，而是生新的对象来替换，那么就用**`shallowRef`**

```vue
<template>
	<div>
  	<button @click="change">改值</button>
  	<h1>{{shallowMessage.name}}</h1>
  </div>
</template>
<script setup>
import { Ref, shallowRef, triggerRef } from "vue"

/*
 // 传入基本类型的值
 let shallowMessage: Ref<string> = shallowRef("Hello World")
 let shallowMessage = shallowRef("Hello World")
 const change = () => {
   shallowMessage.value = "Hello Vue" // 改变了 shallowMessage 值
 } 
 */

let shallowMessage = shallowRef({
  name: "Hello World"
})
const change = () => {
  /*
  shallowMessage.value = { // 可以直接改变 shallowMessage 对象的 name 属性值，因为对象也改变了
    name: "Hello Vue"
  }
	*/
  
  shallowMessage.value.name = "Hello Vue" // 无法改变 shallowMessage 对象的 name 属性值
  triggerRef(shallowMessage) // 触发 shallowMessage 对象的 getter, 强制更新
  
  /**
   * 注意📢： 除了使用 triggerRef 会触发 shallowRef更新外。
   * 还有一种特殊情况也会触发 shallowRef更新： 
   *  1. 不使用 triggerRef
   *  2. 给 shallowRef 赋值的同时，也给 ref 赋值，并且页面模板中使用了 ref，这时候会触发 shallowRef 更新
   */
  // message.value = "Hello Vue" 
  // shallowMessage.value.name = "Hello Vue2"
}
</script>
```

### customRef

> 自定义 `ref`
> `customRef`需要有一个工厂函数作参数，**`track`**（追踪），**`trigger`**（触发），分别对应`getter `和`setter`

```vue
<template>
  <div>
    <button @click="changeMyRef">改值(自定义ref)</button>
    <h1>{{myRefMessage}}</h1>
  </div>
</template>

<script setup lang="ts">
import { customRef } from "vue"

function myRef<T>(value: T) {
  // customRef接收一个工厂函数，返回一个ref，该 ref对象 必须包含 get 和 set 方法
  return customRef((track, trigger) => { 
    return {
      get() {
        // 当 ref 对象的值发生变化时，触发 track 函数, 可以在 track 函数中执行一些异步操作, 
        // 比如请求数据, 更新数据, 更新状态等, 在 track 函数中可以触发 trigger 函数, 触发 trigger 函数会触发 getter 函数, 这样就可以实现响应式编程了
        track();
        return value;
      },
      set(newValue: T) {
        value = newValue;
        trigger(); // 触发更新，即触发 getter 函数, 触发 getter 函数会触发 track 函数, 这样就可以实现响应式编程了
      }
    }
  })
}
let myRefMessage = myRef<string>("Hello World");
const changeMyRef = () => {
  myRefMessage.value = "Hello Vue my ref";
}
</script>
```

## 五、toXX 全家桶

### toRef

> 用来为源对象上的某个 **property** 新创建一个 `ref`。`ref` 可以被传递，它会保持对其源 **property** 的响应式连接

```vue
<script setup lang="ts">
import { reactive, toRef } from 'vue'
  
/* 1.源对象为普通对象 */
const obj = {
   foo: 1,
   bar: 1
}
const state = toRef(obj, 'bar') // bar 转化为响应式对象
state.value++
console.log(obj, state); // {foo: 1, bar: 2} 2

/* 2.源对象为响应式对象 */
const state = reactive({
    foo: 1,
    bar: 1
})
const fooRef = toRef(state, 'foo')
fooRef.value++
console.log(fooRef.value) // 2
console.log(state.foo)    // 2
</script>
```

### toRefs

> 不同于`toRef`的单个属性转换，`toRefs`会把一个响应式对象转换成普通对象，该普通对象的每个 **property** 都是一个 `ref` ，和响应式对象 **property** 一一对应。

`toRefs`的作用是方便我们解构、扩展使用。并且其返回的对象，不会丢失响应性

```vue
<script setup>
import { reactive, toRefs } from 'vue'
  
const obj = reactive({
   foo: 1,
   bar: 1
})
let { foo, bar } = toRefs(obj) // 解构出来不丢失响应性
foo.value++
</script>
```

```vue
<template>
	<p>count: {{count}}</p>
</template>

<script>
import { reactive, toRefs } from 'vue'

export default {
  setup() {
    const state = reactive({
      count: 0
    })

    return {
      ...toRefs(state), // 解构出来不丢失响应性
    }
  }
}
</script>
```

### toRaw

> 将响应式对象转化为普通对象

```vue
<script setup lang="ts">
import { reactive, toRaw } from 'vue'
 
const obj = reactive({
   foo: 1,
   bar: 1
})
 
const state = toRaw(obj) // 响应式对象转化为普通对象

</script>
```

