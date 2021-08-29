# vuex

## 1. Vuex有哪些重要概念

* **state**：**存储数据状态**；可以通过this.$store.state访问；对应vue里的data；存放数据的方式是响应式的，vue组件可以从store中读取数据，如果数据变化，组件也会发生相应的更新。mapstate可以在页面里简化调用

- **Getter**：**就相当于store的计算属性**，他的返回值会根据他的依赖被缓存起来，且只有当它的依赖发生改变的时候才会被重新计算。这里我们可以通过定义vuex的Getter来获取，Getters 可以**用于监听、state中的值的变化，返回计算后的结果**，
- **muation**：**更改vuex的store的唯一方法就是提交mutation**
- **action**：**包含任意异步操作，通过提交mutation间接改变状态**。调用mutation最大的好处是可以异步，分发请求
- **module**：**将store分割成模块**，每个模块都具备上边四种方法，甚至子模块

1. 当组件进行数据修改时，调用dispatch来触发action里边的方法；
2. 每个action里边都有一个commit方法，可以通过commit来修改mutations里边的方法进行数据修改
3. mutation中都会有一个state参数，这样可以通过mutation修改了state数据
4. 数据修改完毕之后，页面发生改变

## 2. Vuex的底层原理

基本的Vue状态自管理应用包含以下几个部分：

- state，驱动应用的数据源；
- view，以声明方式将 state 映射到视图；
- actions，响应在 view 上的用户输入导致的状态变化。

实现原理：vuex是利用vue的**mixin**混入机制，在beforeCreate钩子前混入**vuexInit**方法，**vuexInit方法实现了store注入vue组件实例**，**并注册了vuex store的引用属性$store**。store注入过程如下图所示：

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7aedaddfab44e63b906a77c60932187~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom: 50%;" />

## 3. 为什么用vuex

1. 当**组件多层嵌套**的时候，**父子组件传递数据很麻烦**。
2. **增加代码的可读性和可维护性**
3. **数据是响应式的，减少数据的派发操作**；

## 4. Vuex 如何区分 state 是外部直接修改，还是通过 mutation 方法修改的？

Vuex 中修改 state 的唯一渠道就是执行 **commit('xx', payload)** 方法，其**底层通过执行 this._withCommit(fn) 设置_committing 标志变量为 true，然后才能修改 state，修改完毕还需要还原_committing 变量。**外部修改虽然能够直接修改state，但是并没有修改_committing 标志位，所以只要 watch 一下 state，state change 时判断是否_committing 值为 true，即可判断修改的合法性 。

## 5. vue 中 ajax 请求代码应该写在组件的 methods 中还是 vuex 的 action 中

如果请求来的数据不是要被其他组件公用，仅仅在请求的组件内使用，就不需要放入 vuex 的 state 里