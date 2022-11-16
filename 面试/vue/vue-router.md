# VueRouter

**模式：**hash、history

**跳转方式：**

1. this.$router.push()
2. this.$router.replace()
3. <router-link to=''></router-link>

**占位：**<router-view></router-view>

默认情况下是hash模式，history需要后台配置，利用Html5的History API实现的，监听change变化。

## router和route的区别：

**$router：**是VueRouter的一个**实例**，是一个**全局的对象**，主要**实现路由的跳转使用**。常用的`router.push()`和`router.replace()`方法。

push方法会像浏览器的history栈添加一个新纪录。

replace替换路由，没有历史记录。

**route：**是一个**路由信息对象**，**每一个路由都有一个route对象**，是一个**局部对象**。可以获取**name、query、params、path**等参数。

## vue-router 有哪几种导航钩子?

三种

- **全局导航钩子**

```
router.beforeEach(to, from, next),

router.beforeResolve(to, from, next),

router.afterEach(to, from )
```

- **组件内钩子**

```
beforeRouteEnter,

beforeRouteUpdate,

beforeRouteLeave
```

- **单独路由独享组件**

```
beforeEnter
```