## 定义全局函数和变量

### globalProperties

由于Vue3 没有`Prototype` 属性，使用 `app.config.globalProperties` 代替，然后去定义变量和函数

### 案例

> Vue3 移除了过滤器，正好可以使用全局函数代替Filters

```ts
// main.ts
app.config.globalProperties.$filters = {
  format<T extends any>(str: T): string {
    return `$${str}`
  }
}


// 声明文件，不然TS无法正确类型 推导
type Filter = {
  format: <T extends any>(str: T) => T
}
// 声明要扩充@vue/runtime-core包的声明.
// 这里扩充"ComponentCustomProperties"接口, 因为他是vue3中实例的属性的类型.
declare module '@vue/runtime-core' {
  export interface ComponentCustomProperties {
    $filters: Filter
  }
```

```vue
<script setup lang="ts">
import { getCurrentInstance, ComponentInternalInstance } from 'vue';
 
const { appContext } = <ComponentInternalInstance>getCurrentInstance()
console.log(appContext.config.globalProperties.$env);
  
</script>
```

或者 `hooks`目录下新建一个`useCurrentInstance.ts`文件：

```ts
import { getCurrentInstance, ComponentInternalInstance } from 'vue'

export default function useCurrentInstance() {
  const { appContext } = getCurrentInstance() as ComponentInternalInstance
  const globalProperties = appContext.config.globalProperties
  return globalProperties
}
```



