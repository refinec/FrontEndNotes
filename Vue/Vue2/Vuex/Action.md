# Action

>Action 类似于 mutation，不同在于：
>
>- Action 提交的是 mutation，而不是直接变更状态。
>- Action 可以包含任意异步操作。

#### Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

```javascript
const store = new Vuex.store({
	state:{
        count:0
    },
    mutations:{
        increment(state){
            state.count++
        }
    },
    actions:{
        increment(context){
            context.commit('increment')
        }
        deduce({ commit }){
    		commit('increment')
		}
    }
})
```

#### Actions 支持与Mutations同样的载荷方式和对象方式进行分发

```javascript
// 以载荷形式分发
store.dispatch('incrementAsync',{
    amount:10
})
// 以对象的形式分发
store.dispatch({
    type:'incrementAsync',
    amount:10
})

actions: {
    checkout({ commit, state }, products){
        // 把当前购物车的物品备份起来
        const savedCartItems = [...state.cart.added]
        // 发出结账请求，然后乐观地清空购物车
        commit(types.CHECKOUT_REQUEST) 
    }
}
```

#### 在组件中分发 action

* 使用 this.$store.dispatch('xxx') 分发 action
* 使用 mapActions 辅助函数将组件的 **methods** 映射为 store.dispatch 调用（需要先在根节点注入 store）

```vue
<script>
	methods:{
        ...mapActions([
            // 将'this.increment()' 映射为 'this.$store.dispatch('increment')'
            'increment', 
            // 'mapMutations' 也支持载荷
// 将'this.incrementBy(amount)' 映射为'this.$store.dispatch('incrementBy',amount)'
            'incrementBy'
        ]),
        ...mapActions({
            // 将'this.add()' 映射为 'this.$store.commit('increment')'
            add:'increment'
        })
    }
</script>
```

#### 组合 Action

* **store.dispatch 可以处理被触发的 action 的处理函数返回的 Promise，并且 store.dispatch 仍旧返回 Promise：**

  ```vue
  <script>
      actions:{
          actionA({ commit }){
              return new Promise((resolve, reject)=>{
                  setTimeout(()=>{
                      commit('someMutation')
                      resolve()
                  },1000)
              })
          }
      }
      
      store.dispatch('actionA').then(()=>{
          // ...
      })
  </script>
  ```

* **在action中分发action：**

  ```vue
  <script>
  	actions:{
          actionB({ dispatch, commit}){
              return dispatch('actionA').then(()=>{
                  commit('someOtherMutation')
              })
          }
      }
  </script>
  ```

#### 利用 [async / await](https://tc39.github.io/ecmascript-asyncawait/)，我们可以如下组合 action，一个 store.dispatch 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行：

```vue
<script>
    //假设getData() 和 getOtherData() 返回的是 promise
	actions:{
        async actionA({ commit }){
            commit('gotData', await getData())
        },
        async actionB({ dispatch, commit}){
            await dispatch('actionA'); //等待actionA完成
            commit('gotOtherData', await getOtherData())
        }
    }
</script>
```

