# mutation

mutation 都有一个字符串的 **事件类型 (type) **和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方。

**注意：** mutation 都是同步事务，Mutation 必须是同步函数

### mutation的参数

* **参数一：** state

* **参数二：** payload

  **payload可以是单个参数，也可以是包含多个参数的一个对象**

  ```vue
  <script>
  	mutations:{
          increment(state, payload) {
              state.count += payload.amount
          }
      }
      store.commit('increment', {
          amount:10
      })
  </script>
  ```

### Mutation 需遵守 Vue 的响应规则：

1. **最好提前在你的 store 中初始化好所有所需属性**

2. **当需要在对象上添加新属性时，你应该**

   * 使用**Vue.set(obj, 'newProp', 123) **

   * 或者以新对象替换老对象。例如，利用[对象展开运算符](https://github.com/tc39/proposal-object-rest-spread)我们可以这样写

     ```javascript
     state.obj = { ...state.obj, newProp:123 }
     ```

### 使用常量替代 Mutation 事件类型

这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然

```vue
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'

// store.js
<script>
	import Vuex from 'vuex'
    import { SOME_MUTATION } from './mutation-types'
    
    const store = new Vuex.Store({
        state: {...},
        mutations:{
            // 可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
            [SOME_MUTATION](state){
                
            }
        }
    })
</script> 
```

### 在组件中提交 Mutation

* **组件中使用 this.$store.commit('xxx') 提交 mutation**

* **或使用 mapMutations 辅助函数将组件中的 <u>methods</u> 映射为 store.commit 调用（需要在根节点注入 store）**

  ```VUE
  <script>
  	import { mapMutations } from 'vuex'
      export default {
          methods:{
              ...mapMutations([
                  // 将'this.increment()' 映射为 'this.$store.commit('increment')'
                  'increment', 
                  // 'mapMutations' 也支持载荷
  // 将'this.incrementBy(amount)' 映射为'this.$store.commit('incrementBy',amount)'
                  'incrementBy'
              ]),
              ...mapMutations({
                  // 将'this.add()' 映射为 'this.$store.commit('increment')'
                  add:'increment'
              })
          }
      }
  </script>
  ```

  