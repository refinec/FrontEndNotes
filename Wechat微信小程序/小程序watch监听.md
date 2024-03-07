1. 新建 `watch.js`

   ```js
   const observe = (obj, key, watchFun, deep, page) => {
       let oldVal = obj[key]
       // 如果监听对象是object类型并且指定deep（深度监听）
       if (oldVal !== null && typeof oldVal === 'object' && deep) {
         // 递归子集，依次执行observe()
         Object.keys(oldVal).forEach(item => {
           observe(oldVal, item, watchFun, deep, page)
         })
       }
       // 使用Object.defineProperty()劫持数据的写操作，在监听对象改变后执行传入的watchFun
       Object.defineProperty(obj, key, {
         configurable: true,
         enumerable: true,
         set(value) {
           if (value === oldVal) return
           watchFun.call(page, value, oldVal)
           oldVal = value
         },
         get() {
           return oldVal
         }
       })
     }
     
     export const setWatcher = (page) => {
       // 页面里的data字段
       const data = page.data
       // 页面里的watch字段
       const watch = page.watch
       // 对watch里列举的每一个字段（需要监听的字段）执行observe()
       Object.keys(watch).forEach(key => {
         let targetData = data
         const targetKey = key
         // 支持deep深度监听，使用deep时需要配合handler使用，否则直接编写函数
         const watchFun = watch[key].handler || watch[key]
         const deep = watch[key].deep
         observe(targetData, targetKey, watchFun, deep, page)
       })
     }
   ```

2. 引入使用

   在需要使用监听机制页面的js文件的 `onLoad`钩子里，执行`setWatcher`，并传入当前页面实例`this`，完成初始化。

   添加`watch`对象，内部写入需要被监听的字段以及执行函数：

   ```js
   // test.js
   
   import { setWatcher } from '../../watch.js'
   
   Page({
     data: { foo:false },
     watch: {
       // 需要监听的字段
       foo(val) {
         console.log('foo变化了，变化后的值是', val)
         // 具体操作=>doSomething
       }
     },
     // watch初始化，传入当前页面实例this
     onLoad() {
       setWatcher(this)
     }
   })
   ```

   