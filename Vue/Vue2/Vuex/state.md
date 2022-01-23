# State 单一状态树

##### 1. 从 store 实例中读取状态的最简单方法是在 computed 计算属性中返回某个状态

```vue
<script>
	const Counter = {
        template:`<div>{{ count }} </div>`
        computed:{
            count(){
                return this.$store.state.count
            }
    	}
    }
</script>
```

##### 2. mapState 辅助函数

使用 mapState 辅助函数帮助我们生成计算属性

```vue
<script>
	import { mapState } from 'vuex'
    export default{
        computed: mapState({
            // 箭头函数使代码更简练
            count:state => state.count,
            //传字符串参数 'count' 等同于 'state => state.count'
            countAlias:'count',
            //为了使用‘this’获取局部状态，必须使用常规函数
            countPlusLocalState(state){
                return state.count + this.localCount;
            }
        })
    }
</script>
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 mapState 传一个字符串数组：

```vue
<script>
	computed:mapState([
        // 映射this.count 为 store.state.count
        'count'
    ])
</script>
```

mapState 函数返回的是一个对象，扩展运算符将它与局部计算属性混合使用：

```vue
<script>
	computed:{
        
        ...mapState({
            
        })
    }
</script>
```

