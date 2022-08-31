## toRef

> 如果原始对象是非响应式的就不会更新视图，但数据值会改变。
>
> 如果原始对象是响应式的是会更新视图并且改变数据

```vue
<script setup lang="ts">
import { reactive, toRef } from 'vue'
 
const obj = {
   foo: 1,
   bar: 1
}
 
const state = toRef(obj, 'bar')
// bar 转化为响应式对象
 
const change = () => {
   state.value++
   console.log(obj, state);
 
}
</script>
```

## toRefs

> 批量创建ref对象，主要是方便我们解构使用

```vue
<script setup lang="ts">
import { reactive, toRefs } from 'vue'
const obj = reactive({
   foo: 1,
   bar: 1
})
 
let { foo, bar } = toRefs(obj)
 
foo.value++
</script>
```

## toRaw

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

