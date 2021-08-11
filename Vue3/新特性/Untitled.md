## Composition API: 组合API(类似React Hooks)

> 为vue应用提供更好的逻辑复用和代码组织

### setup

```vue
<script>
import { ref } from "vue";
export default {
  // setup函数是组合API的入口函数
  setup() {
    // let count = 0; // 这样定义则数据不是响应式的
    let count = ref(0); // 定义了一个名叫count的变量，该变量初始值为0，该变量改变之后，vue自动更新UI
    // 在组合API中，如果想定义方法，不用定义到methods中，直接定义即可
    function mf() {
      count.value += 1;
    }
    // 注意：在组合API中定义的变量/方法，要想在外界中使用，必须通过return {xxx} 暴露出去
    return { count, mf };
  }
};
</script>
```

