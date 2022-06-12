## reactive 全家桶

# reactive

> 用来绑定复杂的数据类型 例如 对象 数组

```vue
<script setup lang="ts">
let person = reactive<number[]>([])
setTimeout(() => {
   // person = [1, 2, 3]; // 不能直接赋值，会脱离响应式，因为person的引用地址被改变了
	person.push(...arr);
},1000)
</script>
```

或者

```vue
<script setup lang="ts">
type Person = {
  list?: Array<number>
};
let person = reactive<Person>({
  list: []
})

setTimeout(() => {
  person.list = [1, 2, 3];
})
</script>
```

### readonly

> 拷贝一份proxy对象将其设置为只读

```vue
<script setup lang="ts">
import { reactive, readonly } from "vue"

let person = reactive({
  name: "John",
  age: 30
})

let cp = readonly(person)
</script>
```

### shallowReactive

> 只能对浅层(第一级)的数据，如果是深层(第二级及之后)的数据只会改变值，不会改变视图

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

let person = shallowReactive({
  name: "first",
  group: {
    name: "John",
    age: 30
  },
})
const testShalldowRe = () => {
  // 注意📢：浅层和深层的数据不要在一个逻辑块中同时修改，否则 shallowReactive 将失去应有的作用，深层的数据修改会同步到视图上
  // person.name = "second"; 
  
  person.group.name = "xxxxxx"; // 深沉的数据更新不会同步到视图上
}
</script>
```

