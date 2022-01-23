# getter

> 在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，**getter 的返回值会根据它的依赖被缓存起来**，且只有当它的依赖值发生了改变才会被重新计算。

#### getter的参数

 1. **参数一：** state

    ```vue
    <script>
    	const store = new Vuex.store({
            state:{
                todos:[
                    { id:1, text:'...', done: true }
                ]
            },
            getters:{
                doneTodos: state =>{
                    return state.todos.filter(todo => todo.done)
                }
            }
        })
    </script>
    ```

 2. **参数二：** getters

    ```vue
    <script>
    	getters:{
            doneTodosCount: (state, getters) =>{
                return getters.doneTodos.length
            }
        }
    </script>
    ```

#### 除了通过属性访问，还可通过方法访问

​	通过让 getter 返回一个函数，来实现给 getter 传参

```vue
<script>
	getters:{
        
        getTodoById: (state) => (id) =>{
            return state.todos.find(todo => todo.id === id)
        }
    }
</script>

store.getters.getTodoById(2) // -> { id:2, text:'...', done:false}
```

#### mapGetters 辅助函数

mapGetters 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```vue
<script>
	import { mapGetters } from 'vuex'
    
    export default {
        
        computed:{
            // 使用对象展开运算符将 getter 混入 computed 对象中
            ...mapGetters([
                'done',
                'another'
            ])
            
            //	将一个 getter 属性另取一个名字，使用对象形式：
			...mapGetters({
            doneCount:'doneTodosCount'
        })
        }
    }
</script>
```

