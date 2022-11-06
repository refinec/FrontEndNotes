## 渲染函数

### resolveComponent()

> 按名称手动解析已注册的组件(**如果你可以直接引入组件就不需使用此方法**)
>
> ```ts
> function resolveComponent(name: string): Component | string
> ```

为了能从正确的组件上下文进行解析，`resolveComponent()` 必须在`setup()` 或渲染函数内调用。

如果组件未找到，会抛出一个运行时警告，并返回组件名字符串。

```js
const { h, resolveComponent } = Vue

export default {
  setup() {
    const ButtonCounter = resolveComponent('ButtonCounter')

    return () => {
      return h(ButtonCounter)
    }
  }
}
```



## 服务端渲染





## TypeScript 工具类型





## 自定义渲染