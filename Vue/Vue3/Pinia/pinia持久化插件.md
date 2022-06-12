> pinia 和 vuex 都有一个通病 页面刷新状态会丢失

写一个pinia 插件缓存他的值

```ts
const __piniaKey = '__PINIAKEY__'

type Options = {
  key?: string
}
type OptPinia = Partial<Options>

const setStorage = (key: string, value: any): void => {
  localStorage.setItem(key, JSON.stringify(value))
}


const getStorage = (key: string) => {
  return (localStorage.getItem(key) ? JSON.parse(localStorage.getItem(key) as string) : {})
}


const piniaPlugin = (options: OptPinia) => {

  return (context: PiniaPluginContext) => {
    const { store } = context;
    const data = getStorage(`${options?.key ?? __piniaKey}-${store.$id}`)
    store.$subscribe(() => {
      setStorage(`${options?.key ?? __piniaKey}-${store.$id}`, toRaw(store.$state));
    })
    return {
      ...data
    }
  }
}

const pinia = createPinia()

pinia.use(piniaPlugin({
  key: "pinia"
}))
```

