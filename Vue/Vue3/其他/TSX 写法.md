## TSX

> 之前使用Template去写我们模板，现在可以扩展另一种风格TSX风格

使用`vite`打包工具的话，需要安装: `npm install @vitejs/plugin-vue-jsx -D`,并配置`vite.config.ts `

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from "@vitejs/plugin-vue-jsx"
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), vueJsx()],
})
```

`tsconfig.json` 配置文件添加：

```json
{
  compilerOptions: {
    ...
    "jsx": "preserve",
    "jsxFactory": "h",
    "jsxFragmentFactory": "Fragment",
  }
}
```

配置完成就可以使用啦，在目录新建一个xxxxxx.tsx文件

## 使用TSX

### ref

> tsx不会自动解包，使用ref要加`.vlaue` 

```tsx
 
import { ref } from 'vue'
 
let flag = ref("hello")
 
const renderDom = () => {
    return (
        <div>
           {flag.value}
      	</div>
    )
}

export default renderDom
```

### v-model

```tsx
 
import { ref } from 'vue'
 
let v = ref<string>('')
 
const renderDom = () => {
    return (
        <div>
           <input v-model={v.value} type="text" />
           <div>
               {v.value}
           </div>
        </div>
    )
}
 
export default renderDom
```

### v-show

```tsx
 
import { ref } from 'vue'
 
let flag = ref(false)
 
const renderDom = () => {
    return (
        <div>
           <div v-show={flag.value}>景天</div>
           <div v-show={!flag.value}>雪见</div>
        </div>
    )
}
 
export default renderDom
```

### v-bind

> 直接赋值就可以

```tsx
import { ref } from 'vue'
 
let arr = [1, 2, 3, 4, 5]
 
const renderDom = () => {
    return (
        <div>
            <div data-arr={arr}>1</div>
        </div>
    )
}
 
export default renderDom
```

### v-on

> 所有绑定事件都按照react风格来

- 所有事件有on开头
- 所有事件名称首字母大写

```tsx
const renderDom = () => {
    return (
        <>
            <button onClick={clickTap}>点击</button>
        </>
    )
}
 
const clickTap = () => {
    console.log('click');
}
 
export default renderDom
```

### 不支持`v-if`

> 需要改变风格

```tsx
import { ref } from 'vue'
 
let flag = ref(false)
 
const renderDom = () => {
    return (
        <div>
            {
                flag.value ? <div>真</div> : <div>假</div>
            }
      	</div>
    )
}
 
export default renderDom
```

### 不支持`v-for`

> 需要使用Map

```tsx
import { ref } from 'vue'
 
let arr = [1,2,3,4,5]
 
const renderDom = () => {
    return (
        <div>
            {
              arr.map(v=>{
                  return <div>${v}</div>
              })
            }
        </div>
    )
}
 
export default renderDom
```

### Props 接受值

```tsx
 
import { ref } from 'vue'
 
type Props = {
    title:string
}
 
const renderDom = (props:Props) => {
    return (
        <div>
            <div>{props.title}</div>
            <button onClick={clickTap}>点击</button>
        </div>
    )
}
 
const clickTap = () => {
    console.log('click');
}
 
export default renderDom
```

### Emit派发

```tsx
type Props = {
  title: string
}

const renderDom = (props: Props,content:any) => {
  return (
    <div>
      <div>{props.title}</div>
      <!-- 传参必要要绑定this, 也可以箭头函数写法，内部调用该方法 -->
      <button onClick={clickTap.bind(this,content)}>点击</button>
    </div>
  )
}

const clickTap = (ctx:any) => {
  ctx.emit('on-click',1)
}
```

