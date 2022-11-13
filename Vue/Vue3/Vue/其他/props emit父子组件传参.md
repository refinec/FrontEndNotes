## 父子组件传参

> 父 ——> 子 ：`defineProps`
>
> 子 ——> 父：`defineEmits`

### 子组件接收值

1. **父组件通过`v-bind`绑定一个数据**

   ```vue
   <template>
       <div class="layout">
           <Menu v-bind:data="data"  title="我是标题"></Menu>
       </div>
   </template>
    
   <script setup lang="ts">
   import Menu from './Menu/index.vue'
   import { reactive } from 'vue';
    
   const data = reactive<number[]>([1, 2, 3])
   </script>
   ```

2. **然后子组件通过`defineProps`接受传过来的值**

   > 注意📢：**`defineProps`是无须 import 引入的，直接使用即可**

   * **如果使用的是TypeScript**

     > 可以使用传递字面量类型的纯类型语法做为参数，这是TS特有的

     ```vue
     <template>
         <div class="menu">
             菜单区域 {{ title }}
             <div>{{ data }}</div>
         </div>
     </template>
      
     <script setup lang="ts">
     defineProps<{
         title:string,
         data:number[]
     }>()
     </script>
     ```

     默认值使用`withDefaults`:

     > `withDefaults`是个函数也是无须引入开箱即用。接受一个props函数，第二个参数是一个对象设置默认值

     ```js
     type Props = {
         title?: string,
         data?: number[]
     }
     withDefaults(defineProps<Props>(), {
         title: "张三",
         data: () => [1, 2, 3]
     })
     ```

   * **如果不使用TypeScript**

     ```js
     defineProps({
         title:{
             default:"",
             type:string
         },
         data:Array
     })
     ```

### 父组件接收值

> 方式是通过`defineEmits`派发一个事件

```vue
<template>
    <div class="menu">
        <button @click="clickTap">派发给父组件</button>
    </div>
</template>
 
<script setup lang="ts">
import { reactive } from 'vue'
const list = reactive<number[]>([4, 5, 6])
 
const emit = defineEmits(['on-click']) // 派发一个事件, 事件名为 on-click
const clickTap = () => {
    emit('on-click', list) // 触发 emit 去调用我们注册的事件 然后传递参数
}
</script>
```

**父组件接受子组件的事件**

```vue
<template>
    <div class="layout">
        <Menu @on-click="getList"></Menu>
    </div>
</template>
 
<script setup lang="ts">
import Menu from './Menu/index.vue'
import { reactive } from 'vue';
 
const data = reactive<number[]>([1, 2, 3])
 
const getList = (list: number[]) => {
    console.log(list,'父组件接受子组件');
}
</script>
```

### 子组件暴露给父组件内部属性

> 子组件通过`defineExpose`方法暴露特定的属性给父组件（在vue2中子组件无需暴露，父组件就可以拿到子组件内部属性）

我们从父组件获取子组件实例通过ref

```js
 <Menu ref="menus"></Menu>
  const menus = ref(null)
```

然后打印`menus.value `发现没有任何属性

这时候父组件想要读到子组件的属性可以通过 `defineExpose`暴露

```js
const list = reactive<number[]>([4, 5, 6])

defineExpose({
    list
})
```

这样父组件就可以读到了
