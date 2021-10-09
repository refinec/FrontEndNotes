## 1. 在多个路由中使用一个组件

> 这是开发人员经常会碰到的一种常见情形，即多个路由解析为同一个Vue组件。
>
> 然而，问题在于Vue优化了app并重用现有组件而不是创建新的组件。因此，如果你尝试在使用相同组件的路由之间切换，不会有任何变化。

```js
# router.js
const routes = [
  {
    path: "/a",
    component: MyComponent
  },
  {
    path: "/b",
    component: MyComponent
  },
];
```

要解决此问题，需要在`<router-view>`元素上添加`:key`属性——可能在`App.vue`文件中。这将帮助路由识别页面何时不同。

```vue
<router-view :key='$route.path' />
```

现在，app将不会重用现有组件，并且会在你切换路由时更新内容。

## 2.$on('hook:') 可以帮助简化代码

> 删除事件侦听器是一种常见的JavaScript最佳实践，因为有助于避免内存泄漏并防止事件冲突。

在Vue中添加/删除组件事件监听器时，我们分别使用了生命周期钩子`mounted`和`beforeDestroy`。这是一个典型的设置。

```js
mounted () {
    window.addEventListener('resize', this.resizeHandler);
},
beforeDestroy () {
    window.removeEventListener('resize', this.resizeHandler);
}
```

在典型模式中，我们必须在组件选项对象中单独声明这些钩子。这样做会出现的一个问题是，如果是较大的组件，这些选项可能会相隔数百行代码。

通过查看Vue文档，我们发现了一个用于侦听实例事件的实例方法`$on`。此外，VueJS生命周期钩子会在触发时发出自定义事件。事件名称是“hook:”+钩子本身的名称（例如`hook:created`）

结合这两个技巧，我们可以编写用于在`mounted`方法内部添加和删除的代码。代码如下所示。

```js
mounted () {
    window.addEventListener('resize', this.resizeHandler);
    this.$on("hook:beforeDestroy", () => {
      window.removeEventListener('resize', this.resizeHandler);
    })
}
```

## 3. $on监听子组件的生命周期钩子

生命周期钩子发出自定义事件这一事实，意味着父组件可以监听其子组件的生命周期钩子。

它将使用正常模式来侦听事件(`@event`)，并且可以像其他自定义事件一样进行处理。

```vue
<child-comp @hook:mounted="someFunction" />
```

## 4.总是验证props

> 它为什么如此重要？因为它可以让现在的你拯救未来的你。在设计大型项目时，我们总是很容易忘记用于prop的确切格式、类型和其他约定。如果你在一个较大的开发团队中，你的同事可不会读心术，那么他们怎么知道如何使用你的组件呢？！因此，只需编写prop验证，即可免除其他人费心费力地跟踪组件以确定prop格式的痛苦。

下面是VueJS样式指南中的prop验证示例。

```js
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

## 5. 将所有props传递给子组件

> 如何将所有props从父组件传递到子组件会很有用。
>
> 用例有很多，但是当项目具有非常明显的层次结构时，这个方法会特别方便。

```vue
<child-comp v-bind="$props"></child-comp>
```