# vue

## 1. 虚拟Dom是什么，为什么会有这项技术

Web界面由DOM树(树的意思是数据结构)来构建，当其中一部分发生变化时，其实就是对应某个DOM节点发生了变化， **虚拟DOM就是为了解决浏览器性能问题而被设计出来的**

<u>若一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地一个JS对象中，最终将这个JS对象一次性挂载到DOM树上，再进行后续操作，避免大量无谓的计算量</u>。

所以，用JS对象模拟DOM节点的好处是，**页面的更新可以先全部反映在JS对象(虚拟DOM)上**，操作内存中的JS对象的速度显然要更快，等更新完成后，再将最终的JS对象映射成真实的DOM，交由浏览器去绘制。

所谓**虚拟DOM，就是一个JS对象，用来存储DOM树结构**。

过程：

1. 创建dom树
2. **利用diff算法进行对比，存储差异patches对象**，按下边四种格式
   * 节点类型改变
   * 属性改变
   * 文本改变
   * 增加/移动/删除子节点

3. 渲染差异
   * 遍历patches，把需要修改的节点取出来
   * 局部更新dom，利用**document.fragments**来操作dom，进行浏览器一次性更新。

**document.fragment**

是DOM节点，不是主DOM树的一部分。

通常用来**创建文档片段**，将元素附加到文档片段中，然后将文档片段附加到DOM树中。由于**文档片段存在内存中，不在DOM树中，所以插入子元素的时候不会引发浏览器回流，所以性能更好.**

**浏览器为了<u>重新渲染部分或整个页面</u>，重新<u>计算页面元素</u>位置和<u>几何结构</u>（geometries）的进程叫做 reflow（回流）**

## 2.说一说Diff算法

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa673ebe9da146b0b2cce32ddca036b4~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom:80%;" />

**Diff算法的作用是用来计算出 Virtual DOM 中被改变的部分**，然后针对该部分进行原生DOM操作，而不用重新渲染整个页面。 Diff算法有三大策略：

- Tree Diff
- Component Diff
- Element Diff

三种策略的执行顺序也是顺序依次执行:

* Tree Diff 是对树每一层进行遍历，找出不同。

* Component Diff 是数据层面的差异比较

  * 如果都是同一类型的组件(即：两节点是同一个组件类的两个不同实例，比如：`<div id="before"></div>`与`<div id="after"></div>`)，按照原策略继续比较Virtual DOM树即可

  * 如果出现不是同一类型的组件，则将该组件判断为dirty component，从而替换整个组件下的所有子节点

* Element Diff真实DOM渲染，结构差异的比较

  当节点处于同一层级时，Diff提供三种DOM操作：删除、移动、插入。

### 1. 当数据发生变化时，vue是怎么更新节点的？

要知道渲染真实DOM的开销是很大的，比如有时候我们修改了某个数据，如果直接渲染到真实dom上会引起整个dom树的重绘和重排，有没有可能我们只更新我们修改的那一小块dom而不要更新整个dom呢？diff算法能够帮助我们。

**我们先根据真实DOM生成一颗`virtual DOM`，当`virtual DOM`某个节点的数据改变后会生成一个新的`Vnode`，然后`Vnode`和`oldVnode`作对比，发现有不一样的地方就直接修改在真实的DOM上，然后使`oldVnode`的值为`Vnode`。**

diff的过程就是调用名为`patch`的函数，比较新旧节点，一边比较一边给**真实的DOM**打补丁。

### 2. virtual DOM和真实DOM的区别？

virtual DOM是将真实的DOM的数据抽取出来，以对象的形式模拟树形结构。比如dom是这样的：

```
<div>
    <p>123</p>
</div>
```

对应的virtual DOM（伪代码）：

```
var Vnode = {
    tag: 'div',
    children: [
        { tag: 'p', text: '123' }
    ]
};
```

（温馨提示：`VNode`和`oldVNode`都是对象，一定要记住）

### 3. diff的比较方式？

在采取diff算法比较新旧节点的时候，比较只会在同层级进行, 不会跨层级比较。

![img](https://images2018.cnblogs.com/blog/998023/201805/998023-20180519212338609-1617459354.png)

### 4.具体分析

1.**`patch`**

patch函数接收两个参数`oldVnode`和`Vnode`分别代表新的节点和之前的旧节点

* 判断两节点是否值得比较，值得比较则执行`patchVnode`

* 不值得比较则用`Vnode`替换`oldVnode`

如果两个节点都是一样的，那么就深入检查他们的子节点。如果两个节点不一样那就说明`Vnode`完全被改变了，就可以直接替换`oldVnode`。

虽然这两个节点不一样但是他们的子节点一样怎么办？别忘了，diff可是逐层比较的，如果第一层不一样那么就不会继续深入比较第二层了。

2.**`patchVnode `**

当我们确定两个节点值得比较之后我们会对两个节点指定`patchVnode`方法。

这个函数做了以下事情：

- 找到对应的真实dom，称为`el`
- 判断`Vnode`和`oldVnode`是否指向同一个对象，如果是，那么直接`return`
- 如果他们都有文本节点并且不相等，那么将`el`的文本节点设置为`Vnode`的文本节点。
- 如果`oldVnode`有子节点而`Vnode`没有，则删除`el`的子节点
- 如果`oldVnode`没有子节点而`Vnode`有，则将`Vnode`的子节点真实化之后添加到`el`
- 如果两者都有子节点，则执行`updateChildren`函数比较子节点，这一步很重要

3.**`updateChildren`**

先说一下这个函数做了什么

- 将`Vnode`的子节点`Vch`和`oldVnode`的子节点`oldCh`提取出来
- `oldCh`和`vCh`各有两个头尾的变量`StartIdx`和`EndIdx`，它们的2个变量相互比较，一共有4种比较方式。如果4种比较都没匹配，如果设置了`key`，就会用`key`进行比较，在比较的过程中，变量会往中间靠，一旦`StartIdx>EndIdx`表明`oldCh`和`vCh`至少有一个已经遍历完了，就会结束比较。

<img src="https://images2018.cnblogs.com/blog/998023/201805/998023-20180519211756656-545182812.png" style="zoom:67%;" />

粉红色的部分为oldCh和vCh.将它们取出来并分别用s和e指针指向它们的头child和尾child

<img src="https://images2018.cnblogs.com/blog/998023/201805/998023-20180519211809225-1140464542.png" style="zoom:67%;" />

现在分别对`oldS、oldE、S、E`两两做`sameVnode`比较，有四种比较方式，当其中两个能匹配上那么真实dom中的相应节点会移到Vnode相应的位置，这句话有点绕，打个比方

- 如果是oldS和E匹配上了，那么真实dom中的第一个节点会移到最后
- 如果是oldE和S匹配上了，那么真实dom中的最后一个节点会移到最前，匹配上的两个指针向中间移动
- 如果四种匹配没有一对是成功的，那么遍历`oldChild`，`S`挨个和他们匹配，匹配成功就在真实dom中将成功的节点移到最前面，如果依旧没有成功的，那么将`S对应的节点`插入到dom中对应的`oldS`位置，`oldS`和`S`指针向中间移动。

这个匹配过程的结束有两个条件：

- `oldS > oldE`表示`oldCh`先遍历完，那么就将多余的`vCh`根据index添加到dom中去（如上图）
- `S > E`表示vCh先遍历完，那么就在真实dom中将区间为`[oldS, oldE]`的多余节点删掉



## 5. MVC/MVP/MVVM的区别？

MVC、MVP 和 MVVM 是三种常见的软件架构设计模式，主要通过分离关注点的方式来组织代码结构，优化我们的开发效率。

### MVC

MVC 通过分离 Model、View 和 Controller 的方式来组织代码结构。

* **View 负责<u>页面的显示逻辑</u>**

* **Model 负责<u>存储页面的业务数据，以及对相应数据的操作</u>。**并且 View 和 Model 应用了观察者模式，当 Model 层发生改变的时候它会通知有关 View 层更新页面。

* **Controller 层是 View 层和 Model 层的纽带**，它主要**负责用户与应用的响应操作**，当用户与页面产生交互的时候，Controller 中的事件触发器就开始工作了，通过调用 Model 层，来完成对 Model 的修改，然后 Model 层再去通知 View 层更新。

### MVP

### MVVM

MVVM 是 Model-View-ViewModel 的缩写。mvvm 是一种设计思想。**Model 层代表数据模型，也可以在 Model 中定义数据修改和操作的业务逻辑**；**View 代表 UI 组件，它负责将数据模型转化成 UI 展现出来**，**ViewModel 是一个同步 View 和 Model 的对象**。

在 MVVM 架构下，View 和 Model 之间并没有直接的联系，而是通过 ViewModel 进行交互，Model 和 ViewModel 之间的交互是双向的， 因此 View 数据的变化会同步到 Model 中，而 Model 数据的变化也会立即反应到 View 上。

**ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来**，而 View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作 DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。

 **Vue的MVVM模式实现原理:

Mvvm的入口函数，将三者整合

* observe：一个**数据监听器**，对数据对象的所有属性，包括子属性进行监听，如果发生变化，拿到最新值并通知订阅者

* compile：一个**指令解析器**，对每个元素的节点指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的函数

* watcher: 一个**订阅者**，它将作为连接数据监听器和指令解析器的一个桥梁。能够订阅每个属性的数据变化，收到数据变化的通知，执行回调函数，更新UI页面

  

## 6. 看过Vue源码吗？

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b404428030c478f91b70b1280255592~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom: 50%;" />

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82025d8755344f7f81ffef0978d4d37a~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom:50%;" />

## 7. computed 原理

1.每个 computed 属性都会生成对应的**观察者**（Watcher 实例），观察者存在 values 属性和 get 方法。computed 属性的 getter 函数会在 get 方法中调用，并将返回值赋值给 value。初始设置 dirty 和 lazy 的值为 true，lazy 为 true 不会立即 get 方法（懒执行），而是会在读取 computed 值时执行。`🐷划重点，这里是Vue避免重复渲染的方法`

2.将 computed 属性添加到组件实例上，并通过 get、set 获取或者设置属性值，并且重定义 getter 函数。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bed97ed860a849b19287b7081eb1dead~tplv-k3u1fbpfcp-zoom-1.image)

3.页面初始渲染时，读取 computed 属性值，触发重定义后的 getter 函数。由于观察者的 dirty 值为 true，将会调用 get 方法，执行原始 getter 函数。getter 函数中会读取 data（响应式）数据，读取数据时会触发 data 的 getter 方法，会将 computed 属性对应的观察者添加到 data 的依赖收集器中（用于 data 变更时通知更新）。观察者的 get 方法执行完成后，更新观察者的 value 值，并将 dirty 设置为 false，表示 value 值已更新，之后在执行观察者的 depend 方法，将上层观察者（该观察者包含页面更新的方法，方法中读取了 computed 属性值）也添加到 getter 函数中 data 的依赖收集器中（getter 中的 data 的依赖器收集器包含 computed 对应的观察者，以及包含页面更新方法（调用了 computed 属性）的观察者），最后返回 computed 观察者的 value 值。

4.当更改了 computed 属性 getter 函数依赖的 data 值时，将会根据之前依赖收集的观察者，依次调用观察者的 update 方法，先调用 computed 观察者的 update 方法，由于 lazy 为 true，将会设置观察者的 dirty 为 true，表示 computed 属性 getter 函数依赖的 data 值发生变化，但不调用观察者的 get 方法更新 value 值。再调用包含页面更新方法的观察者的 update 方法，在更新页面时会读取 computed 属性值，触发重定义的 getter 函数，此时由于 computed 属性的观察者 dirty 为 true，调用该观察者的 get 方法，更新 value 值，并返回，完成页面的渲染。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80c34d68f7084b4ea41a14190ce83bb3~tplv-k3u1fbpfcp-zoom-1.image)

5.dirty 值初始为 true，即首次读取 computed 属性值时，根据 setter 计算属性值，并保存在观察者 value 上，然后设置 dirty 值为 false。之后读取 computed 属性值时，dirty 值为 false，不调用 setter 重新计算值，而是直接返回观察者的 value，也就是上一次计算值。只有当 computed 属性 setter 函数依赖的 data 发生变化时，才设置 dirty 为 true，即下一次读取 computed 属性值时调用 setter 重新计算。也就是说，computed 属性依赖的 data 不发生变化时，不会调用 setter 函数重新计算值，而是读取上一次计算值。

## 8. vuex 中muation为什么不能有异步操作，如果要异步要怎么做

**每个mutation执行完成后都会对应到一个新的状态变更，这样devtools就可以打个快照存下来，然后就可以实现 time-travel 了。如果mutation支持异步操作，就没有办法知道状态是何时更新的，无法很好的进行状态的追踪，给调试带来困难**。

Action：可以异步，但不能直接操作State。

## 9. Vue设置响应式数据除了set还有什么方法？

对象添加属性还可以使用`Object.assign({},obj1,obj2)`返回获取的新对象替换原有对象

**forceUpatde**

## 10. vue3 新特性

### 1.检测机制

**vue2中基于`Object.defineProperty`的`observer`实现，vue3中则基于`Proxy`的`observer`实现**

- 对属性的添加、删除动作的监测
- 对数组基于下标的修改，对于`length`修改的监测
- 对Map、Set的支持

默认为惰性监测

- 1、 在2.x版本中，响应式数据都会在启动的时候进行监测，如果数据量较大，会有严重的性能消耗
- 2、 在3.x版本中，只有应用初始可见部分所用到的数据会被监测到

更精准的变动通知：

- 1、在2.x版本中，通过`Vue.set`强制添加一个新的属性，所有依赖这个对象的`watch`函数都被执行一次
- 2、在3.x中，只有依赖这个具体属性的`watch`函数被通知

更好的调试：

- 1、通过使用新增的`renderTracked`和`renderTriggered`钩子，可以精确的追踪到一个组件发生重新渲染的触发时机及原因

暴露出`observable`的`api`来创建响应式对象，可以替代掉`event bus`,做一些跨组件的通信

### 2. 性能优化

组件渲染：

- 1、在2.x版本中，父组件重新渲染时，其子组件也必须重新渲染(前提是修改的数据是子组件的props，才会触发子组件的重新渲染)
- 2、在3.x版本中，可以单独重新渲染子组件或者父组件

静态树提升

- 1、 在3.x版本中，编译器可以检测到静态组件，将其提升，降低渲染成本

  静态属性提升

### 3. 合成API(Composition API)

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/018da4a545244a7586bafe68232cc2c6~tplv-k3u1fbpfcp-zoom-1.image)

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/626dd761d59145388300694d0f80d9e2~tplv-k3u1fbpfcp-zoom-1.image)

## 11. Vue Loader是什么？

Vue Loader 是一个 `webpack` 的 `loader`，它允许你以一种名为单文件组件 `(SFCs)`的格式撰写 `Vue` 组件：

```vue
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>
```

Vue Loader 还提供了很多酷炫的特性：

- 允许为 Vue 组件的每个部分使用其它的 webpack loader，例如在  的部分使用 Sass 和在 `<template>` 的部分使用 Pug；
- 允许在一个 .vue 文件中使用自定义块，并对其运用自定义的 loader 链；
- 使用 webpack loader 将  和 `<template> `中引用的资源当作模块依赖来处理；
- 为每个组件模拟出 scoped CSS；
- 在开发过程中使用热重载来保持状态。

## 12.keep-alive

我们在平时开发中，总有部分组件没必要多次 Init,我们需要将组件**进行持久化，使组件状态维持不变**，在下一次展示时，也不会重新init keepalive 音译过来就是保持活跃，所以在vue中我们可以使用keepalive来进行组件缓存 基本使用

```vue
//只缓存组件name为a或者b的组件
<keep-alive include="a,b">
 <component :is="currentView"/>
</keep-alive>

//组件名为c的组件不缓存
<keep-alive exclude="c">
 <component :is="currentView"/>
</keep-alive>

// 如果同时使用include,exclude,那么exclude优先于include， 下面的例子也就是只缓存a组件
<keep-alive include="a,b" exclude="b">
 <component :is="currentView"/>
</keep-alive>

// 如果缓存的组件超过了max设定的值5，那么将删除第一个缓存的组件
<keep-alive exclude="c" max="5">
 <component :is="currentView"/>
</keep-alive>
```

## 13. 父子组件的生命周期

### 加载渲染过程

**父beforeCreate->父created->父beforeMount**->子beforeCreate->子created->子beforeMount->子mounted->**父mounted**

### 子组件更新过程

**父beforeUpdate**->子beforeUpdate->子updated->**父updated**

### 父组件更新过程

父beforeUpdate->父updated

### 销毁过程

**父beforeDestroy-**>子beforeDestroy->子destroyed->**父destroyed**

## 16.vue中路由按需加载的几种方式

**普通加载的缺点 **：

webpack在打包的时候会**把整个路由打包成一个js文件**，如果页面一多，会导致这个文件非常大，加载缓慢

 1、**webpack 中提供了 require.ensure()来实现按需加载 **

- 语法：

```javascript
require.ensuire(dependencies:String[],callback:function(require),errorCallback:function(error),chunkName:String)
```

- vue中使用：

```javascript
const List = resolve =>{ require.ensure([],()=>{ resolve(require('./list')) },'list') }
```

 2、**vue异步组件技术 **

在router中配置，使用这种方法可以实现按需加载，一个组件生成一个js文件

- vue中使用：

```javascript
{
    path: '/home',
    name: 'home',
    component:resolve => require(['@/components/home'],resolve)
}
```

 3、**使用动态的import()语法（推荐使用这种） **

- vue中使用：

```javascript
//没有指定webpackChunkName,每个组件打包成一个js文件
const test1 = ()=>import('@/components/test1.vue') 
const test2 = ()=>import('@/components/test2.vue')

//指定了相同的webpackChunkName，会合并打包成y一个js文件
const test3 = ()=>import(/* webpackChunkName:'grounpTest' */ '@/components/test3.vue') 
const test4 = ()=>import(/* webpackChunkName:'grounpTest' */ '@/components/test4.vue')

const router = new VueRouter({
    routes: [
        { path: '/test1', component: test1 },
        { path: '/test2', component: test2 },
        { path: '/test3', component: test3 },
        { path: '/test4', component: test4 }
    ]
 })
```

- /* webpackChunkName: 'grounpTest' */使用命名chunk，一个特殊的注释语法来提供 chunk name (需要 Webpack > 2.4)

## 18. v-show和v-if指令的共同点和不同点?

- v-show指令是通过修改元素的displayCSS属性让其显示或者隐藏
- v-if指令是直接销毁和重建DOM达到让元素显示和隐藏的效果