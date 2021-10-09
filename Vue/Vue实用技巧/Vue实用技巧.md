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
// 将key直接设置为路由的完整路径
<router-view :key="$route.fullpath" />
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

## 6. 一劳永逸的组件注册

场景还原：

```js
import BaseButton from  ./baseButton 
import BaseIcon from  ./baseIcon 
import BaseInput from  ./baseInput 

export default {
  components: {
    BaseButton,
    BaseIcon,
    BaseInput
  }
}
```

```vue
<BaseInput
  v-model="searchText"
  @keydown.enter="search"
/>
<BaseButton @click="search">
  <BaseIcon name="search"/>
</BaseButton>
```

我们写了一堆基础UI组件，然后每次我们需要使用这些组件的时候，都得先import，然后声明components，很繁琐！秉持能偷懒就偷懒的原则，我们要想办法优化！

***招式解析：***
我们需要借助一下神器webpack，使用 **`require.context()`** 方法来创建自己的（模块）上下文，从而实现自动动态require组件。这个方法需要3个参数：要搜索的文件夹目录，是否还应该搜索它的子目录，以及一个匹配文件的正则表达式。

我们在components文件夹添加一个叫global.js的文件，在这个文件里借助webpack动态将需要的基础组件统统打包进来。

```js
import Vue from  vue 

function capitalizeFirstLetter(string) {
  return string.charAt(0).toUpperCase() + string.slice(1)
}

const requireComponent = require.context(
   . , false, /.vue$/
   //找到components文件夹下以.vue命名的文件
)

requireComponent.keys().forEach(fileName => {
  const componentConfig = requireComponent(fileName)

  const componentName = capitalizeFirstLetter(
    fileName.replace(/^.//, ).replace(/.w+$/, )
    //因为得到的filename格式是:  ./baseButton.vue , 所以这里我们去掉头和尾，只保留真正的文件名
  )

  Vue.component(componentName, componentConfig.default || componentConfig)
})
```

最后我们在main.js中import  components/global.js ，然后我们就可以随时随地使用这些基础组件，无需手动引入了。

## 7. 使用render函数解决一个组件只能有一个根元素

**场景还原:**
vue要求每一个组件都只能有一个根元素，当你有多个根元素时，vue就会给你报错。

```vue
<template>
  <li
    v-for="route in routes"
    :key="route.name"
  >
    <router-link :to="route">
      {{ route.title }}
    </router-link>
  </li>
</template>

 ERROR - Component template should contain exactly one root element. 
    If you are using v-if on multiple elements, use v-else-if 
    to chain them instead.
```

***招式解析：***
那有没有办法化解呢，答案是有的，只不过这时候我们需要使用render()函数来创建HTML，而不是template。

其实用js来生成html的好处就是极度的灵活功能强大，而且你不需要去学习使用vue的那些功能有限的指令API，比如v-for, v-if。（reactjs就完全丢弃了template）

```vue
functional: true,
render(h, { props }) {
  return props.routes.map(route =>
    <li key={route.name}>
      <router-link to={route}>
        {route.title}
      </router-link>
    </li>
  )
}
```

## 8. 无招胜有招的高阶组件

划重点：这一招威力无穷，请务必掌握
当我们写组件的时候，通常我们都需要从父组件传递一系列的props到子组件，同时父组件监听子组件emit过来的一系列事件。举例子：

```vue
//父组件
<BaseInput 
    :value="value"
    label="密码" 
    placeholder="请填写密码"
    @input="handleInput"
    @focus="handleFocus>
</BaseInput>

//子组件
<template>
  <label>
    {{ label }}
    <input
      :value="value"
      :placeholder="placeholder"
      @focus=$emit( focus , $event)"
      @input="$emit( input , $event.target.value)"
    >
  </label>
</template>
```

有下面几个优化点：

1、每一个从父组件传到子组件的props,我们都得在子组件的Props中显式的声明才能使用。这样一来，我们的子组件每次都需要申明一大堆props, 而类似placeholer这种dom原生的property我们其实完全可以直接从父传到子，无需声明。方法如下：

```vue
<input
      :value="value"
      v-bind="$attrs"
      @input="$emit( input , $event.target.value)"
    >
```

`$attrs`包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定，并且可以通过 v-bind="$attrs" 传入内部组件——在创建更高层次的组件时非常有用。

2、注意到子组件的`@focus=$emit( focus , $event)"`其实什么都没做，只是把event传回给父组件而已，那其实和上面类似，我完全没必要显式地申明：

```vue
<input
    :value="value"
    v-bind="$attrs"
    v-on="listeners"
>

computed: {
  listeners() {
    return {
      ...this.$listeners,
      input: event => 
        this.$emit( input , event.target.value)
    }
  }
}

```

$listeners包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。

3、需要注意的是，由于我们input并不是BaseInput这个组件的根节点，而默认情况下父作用域的不被认作 props 的特性绑定将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。所以我们需要设置inheritAttrs:false，这些默认行为将会被去掉, 以上两点的优化才能成功。