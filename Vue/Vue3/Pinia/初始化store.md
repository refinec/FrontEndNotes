**流程：**

1. **新建一个文件夹Store**

2. **新建名称文件`store-namespace/index.ts`**

   ```ts
   export const enum Names {
       Test = 'TEST'
   }
   ```

3. **新建文件`index.ts`**

   ```ts
   import { defineStore } from 'pinia'
   import { Names } from './store-namespace'
   
   // store定义的defineStore()，需要一个唯一的名称，作为第一个参数传递
   // 这个名称，也称为id，是必要的，Pania 使用它来将商店连接到 devtools。将返回的函数命名为use...是可组合项之间的约定，以使其使用习惯。
   export const useTestStore = defineStore(Names.Test, {
       // State 必须箭头函数返回一个对象，在对象里面定义值
       state: () => {
           return {
               count: 0
           }
       },
       //类似于computed 可以帮我们去修饰我们的值
       getters: {
   
       },
       //可以操作异步 和 同步提交state
       actions: {
   
       }
   })
   
   ```

3. 在组件中导入使用

   ```vue
   <template>
     <div>
    		<!-- State 是允许直接修改值的 -->   
       <button @click="Test.count++">加加加</button>
       {{ Test.count}}
     </div>
   </template>
   
   <script setup lang="ts">
   import { useTestStore } from "./store"
   const Test = useTestStore();
   
   </script>
   ```

   