## watch 与 watchEffect 侦听器

### 一、watch 侦听器

> 参数一：监听源
>
> 参数二：回调函数(newV, oldV)
>
> 参数三：配置选项 { immediate: true /* 是否立即调用一次 */, deep: true / * 是否开启深度监听 */}

```js
import { ref, watch } from 'vue'
 
let message = ref({
    nav:{
        bar:{
            name:""
        }
    }
})
 
watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
},{
    immediate:true,
    deep:true
})
```

#### 监听多个ref

```js
import { ref, watch ,reactive} from 'vue'
 
let message = ref('')
let message2 = ref('')
 
watch([message,message2], (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

#### 使用reactive监听深层对象开启和不开启deep 效果一样

```js
import { ref, watch ,reactive} from 'vue'
 
let message = reactive({
    nav:{
        bar:{
            name:""
        }
    }
})
 
 
watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

#### 监听reactive 单一值

```js
import { ref, watch ,reactive} from 'vue'
 
let message = reactive({
    name:"",
    name2:""
})
 
 
watch(()=>message.name, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

### watchEffect 高级侦听器

> 参数一：回调函数 (oninvalidate) => {}
>
> 参数二：配置选项

监听属性，用到几个监听几个，而且是非惰性，会默认调用一次

```js
let message = ref<string>('')
let message2 = ref<string>('')
 watchEffect(() => {
    //console.log('message', message.value);
    console.log('message2', message2.value);
})
```

#### 清除副作用

> 在触发监听之前会调用一个函数可以处理你的逻辑例如防抖

```js
import { watchEffect, ref } from 'vue'
let message = ref<string>('')
let message2 = ref<string>('')
 watchEffect((oninvalidate) => {
    //console.log('message', message.value);
    oninvalidate(()=>{
        
    })
    console.log('message2', message2.value);
})
```

#### 停止监听

> 停止跟踪 watchEffect 返回一个函数，调用之后将停止更新

```js
const stop =  watchEffect((oninvalidate) => {
    console.log('message2', message2.value);
})
stop()
```

#### 参数二 配置项

* `flush`

  **副作用刷新时机 `flush` 一般使用post**

  |          | pre                | sync                 | post               |
  | :------- | :----------------- | :------------------- | :----------------- |
  | 更新时机 | 组件**更新前**执行 | 强制效果始终同步触发 | 组件**更新后**执行 |

* `onTrigger(){}`

  **onTrigger 可以帮助我们调试 watchEffect**

  ```js
  import { watchEffect, ref } from 'vue'
  let message = ref<string>('')
  let message2 = ref<string>('')
   watchEffect((oninvalidate) => {
      //console.log('message', message.value);
      oninvalidate(()=>{
   
      })
      console.log('message2', message2.value);
  },{
      flush:"post",
      onTrigger (e) {
          debugger
      }
  })
  ```

  

