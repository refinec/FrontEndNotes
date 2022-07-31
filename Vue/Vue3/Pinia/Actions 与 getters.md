## Actions（支持同步异步）

### 同步，直接调用即可

```ts
import { defineStore } from 'pinia'
import { Names } from './store-naspace'
export const useTestStore = defineStore(Names.TEST, {
    state: () => ({
        counter: 0,
    }),
    actions: {
        increment() {
            this.counter++
        },
        randomizeCounter() {
            this.counter = Math.round(100 * Math.random())
        },
    },
})
```

```vue
<template>
     <div>
         <button @click="Add">+</button>
          <div>
             {{Test.counter}}
          </div>    
     </div>
</template>
 
<script setup lang='ts'>
import {useTestStore} from './store'
const Test = useTestStore()
const Add = () => {
     Test.randomizeCounter()
}
 
</script>
```

### 异步,可以结合async await 修饰

```ts
import { defineStore } from 'pinia'
import { Names } from './store-naspace'
 
type Result = {
    name: string
    isChu: boolean
}
 
const Login = (): Promise<Result> => {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve({
                name: '小满',
                isChu: true
            })
        }, 3000)
    })
}
 
export const useTestStore = defineStore(Names.TEST, {
    state: () => ({
        user: <Result>{},
        name: "123"
    }),
    actions: {
        async getLoginInfo() {
            const result = await Login()
            this.user = result;
        }
    },
})
```

```vue
<template>
     <div>
         <button @click="Add">test</button>
          <div>
             {{Test.user}}
          </div>    
     </div>
</template>
 
<script setup lang='ts'>
import {useTestStore} from './store'
const Test = useTestStore()
const Add = () => {
     Test.getLoginInfo()
}
 
</script>
```

多个action互相调用

```ts
state: () => ({
  user: <Result>{},
  name: "default"
}),
  actions: {
    async getLoginInfo() {
      const result = await Login()
      this.user = result;
      this.setName(result.name)
    },
      setName (name:string) {
        this.name = name;
      }
  },
```

## getters

> 主要作用类似于computed 数据修饰并且有缓存

1. 使用箭头函数不能使用this，this指向已经改变指向`undefined`，修改值请用state

       getters:{
          newPrice:(state)=>  `$${state.user.price}`
       },

2. 普通函数形式可以使用this

   ```
   getters:{
     newCurrent ():number {
     	return ++this.current
     }
   },
   ```

3. getters 互相调用

   ```
   getters:{
     newCurrent ():number | string {
     	return ++this.current + this.newName
     },
     newName ():string {
     	return `$-${this.name}`
     }
   },
   ```

   