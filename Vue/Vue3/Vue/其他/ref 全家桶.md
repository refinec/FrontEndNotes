## ref 全家桶

> 共有3组
>
> * `ref,` `Ref`, `isRef`;
> * `shallowRef`, `triggerRef`; 
> * `customRef`

```vue
<template>
  <div>
    <button @click="change">改值</button>
    <h1>{{shallowMessage.name}}</h1>
    
    <button @click="changeMyRef">改值(自定义ref)</button>
    <h1>{{myRefMessage}}</h1>
  </div>
</template>

<script setup lang="ts">
import { ref, Ref, isRef, shallowRef, triggerRef, customRef } from "vue"

/**
 * 1. ref, Ref, isRef的使用
 */
let message: Ref<string> = ref("Hello World")
let notRef: string = "Hello World"
console.log('是否为Ref响应式类型：', isRef(message), isRef(notRef)) // >>> true false

  
/**
 * 2. shallowRef，triggerRef的使用
 */

// let shallowMessage: Ref<string> = shallowRef("Hello World")
// let shallowMessage = shallowRef("Hello World")
// const change = () => {
//   shallowMessage.value = "Hello Vue" // 改变 shallowMessage 值
// }

let shallowMessage = shallowRef({
  name: "Hello World"
})
const change = () => {
  // shallowMessage.value = { // 可以直接改变 shallowMessage 对象的 name 属性值，因为对象也改变了
  //   name: "Hello Vue"
  // }

  shallowMessage.value.name = "Hello Vue" // 无法改变 shallowMessage 对象的 name 属性值
  triggerRef(shallowMessage) // 触发 shallowMessage 对象的 getter, 强制更新
  
  /**
   * 注意📢： 除了使用 triggerRef 会触发 shallowRef更新外。
   * 还有一种特殊情况也会触发 shallowRef更新： 
   *  1. 不使用 triggerRef
   *  2. 给 shallowRef 赋值的同时，在也给 ref 赋值，并且页面模板中使用了 ref，这时候会触发 shallowRef 更新
   */
  // message.value = "Hello Vue" 
  // shallowMessage.value.name = "Hello Vue2"
}


/**
 * 3. 自定义 ref
 */
function myRef<T>(value: T) {
  // customRef接收一个工厂函数，返回一个ref，该 ref对象 必须包含 get 和 set 方法
  return customRef((track, trigger) => { // 
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

