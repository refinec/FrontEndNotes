## 一、通用

### nextTick()

> 当你在 Vue 中更改响应式状态时，最终的 DOM 更新并不是同步生效的，而是由 Vue 将它们缓存在一个队列中，直到下一个“tick”才一起执行。这样是为了确保每个组件无论发生多少状态改变，都仅执行一次更新。

`nextTick()` 可以在状态改变后立即使用，以等待 DOM 更新完成。你可以**传递一个回调函数作为参数，或者 await 返回的 Promise。**

```vue
<template>
  <button id="counter" @click="increment">{{ count }}</button>
</template>
<script setup>
import { ref, nextTick } from 'vue'

const count = ref(0)

async function increment() {
  count.value++

  // DOM 还未更新
  console.log(document.getElementById('counter').textContent) // 0

  await nextTick()
  // DOM 此时已经更新
  console.log(document.getElementById('counter').textContent) // 1
}
</script>
```

## 二、应用实例

### createApp()

> 创建一个应用实例
>
> ```ts
> function createApp(rootComponent: Component, rootProps?: object): App
> ```