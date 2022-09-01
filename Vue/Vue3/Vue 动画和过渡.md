## `<transition>`元素到底是什么？

transition元素是一个包装器，可协助向元素添加过渡动画功能。其本质是设置不同的钩子并添加类到不断变化的元素中，因此我们可以在过渡的不同阶段设置样式。

有6种不同的过渡动画类。

- `v-enter-from` / `v-leave-from`：过渡的开始状态；过渡开始后将被删除（Vue2中的是`v-enter`和`v-leave`）
- `v-enter-active` / `v-leave-active`：过渡的活动状态
- `v-enter-to` / `v-leave-to`：过渡的结束状态

> 注意：对于这些在过渡中切换的类名来说，如果你使用一个没有名字的`<transition>`，则`v-`是这些类名的默认前缀。如果你给`<transition>`指定了`name`属性，那么`v-enter`会替换为`{name}-enter-from`、`{name}-enter-active`等等，以此类推。

## Vue Transition示例

创建Vue Transition的模板代码并不难。只需选择你想要过渡的元素并将其包装在`<transition>`组件中即可。

在下面这个例子中，我们将创建一个按钮来转换`<p>`元素以进行渲染。

```vue
<template>
  <div>
    <h1>Vue Transition Animation</h1>
    <button @click="open = !open">Toggle Animation</button>
    <transition name="fade">
      <p v-if="open" class="example-div">Hello World</p>
    </transition>
  </div>
</template>
```

相应的`<script>`部分。

```vue
import { ref } from "vue";

export default {
  setup() {
    const open = ref(true);

    return {
      open,
    };
  },
};
```

现在，我们只需要添加一些CSS样式来使过渡动画工作。

请看Vue文档中的样式示例。

```css
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

看到所有的类都包含前缀`fade`没？这是因为我们给`<transition>`指定了`name`属性！

这段代码的目的非常直观，因为具有相似状态的类。

这些样式表明当过渡处于活动状态时，将过渡添加到`opacity`属性可以使其平滑运动。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ibugX3Hm3p4oML0aecPEUicn4vicwkCSQWaPLibGFJNBoV97raHxDxm3Y8RV1ntQznwvUnbfD7zlBPN4IEzGe07xMQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

很好！

除了使用CSS过渡，还可以使用CSS动画。

只要能够使用正确的类名，我们就可以随心所欲地设计这些组件的样式。

## 自定义类名和JS钩子

此外，将下面6个属性中添加到`<transition>`元素，可以覆盖默认的类名：

- `enter-from-class`
- `enter-active-class`
- `enter-to-class`
- `leave-from-class`
- `leave-active-class`
- `leave-to-class`

这在将自定义库添加到代码时尤其有用。也是我们稍后要对`Animate.css`所做的事情。

```vue
<transition 
  enter-active-class="animated fadeIn zoomIn" 
  leave-active-class="animated fadeOut zoomOut"
>
...
</transition>
```

还可以使用过渡元素发出的JS钩子，此时我们需要捕获它们并用JavaScript而不是CSS执行动画。可用的钩子是：

- `before-enter` / `before-leave`
- `enter` / `leave`
- `after-enter` / `after-leave`
- `enter-cancelled` / `leave-cancelled`

声明JS事件处理程序

```vue
<transition @before-enter='beforeEnter'>
    <!-- ... -->
</transition>
```

然后用JavaScript进行处理。

```vue
export default {
  methods: {
    // done is an optional callback method
    beforeEnter(el, done) {
      done();
    },
  },
};
```

接下来继续讨论使用Vue过渡的一些更高级的技术。

## Vue过渡的高级技术

虽然我们刚刚构建的过渡元素很好地概述了组件的工作原理，但我们在现实中遇到的通常是更复杂的用例。

幸运的是，与大多数Vue一样，模板非常灵活，可以适应大多数项目。

### 让组件在加载时过渡

简单极了。只需像这样将属性`appear`添加到过渡元素即可。

```vue
<transition name="fade" appear>
      ...
</transition>
```

### 在多个元素之间过渡

假设你有两个`div`，它们像这样在彼此之间交替。

```vue
<transition name="fade" appear>
      <div v-if="visible">
        Option A
      </div>
      <div v-else>
        Option B
      </div>
</transition>
```

你所要做的就是将这两个`div`包装在一个过渡元素和BAM中——过渡样式两者皆适用。

要使代码按想要的方式运行，需要注意以下几点：

**1.可能需要绝对定位元素**

当Vue在两个元素之间进行过渡时，有时两个元素都是可见的，有时两个元素都在`transition in/out`。如果你想要平滑的效果，那么你可能会采取将它们彼此绝对定位的措施。

否则，当从DOM中添加/删除元素时，它们可能就会到处乱跳。

**2.如果元素有相同的标签，那必须给组件添加`key`属性**

如果元素有相同的标签，Vue会尝试优化，替换元素的内容。根据文档，如果在多个元素之间过渡，添加`key`始终是最佳实践。

### 改变过渡时间

Vue通常可以检测到过渡/动画何时结束，但如果你想设置确切的持续时间，Vue过渡有一个`duration`属性。

你可以为进入和离开转换传递一个值，也可以传递一个具有两个值的对象。

自定义过渡时间：

```vue
<transition :duration="500">...</transition>

...

<transition :duration="{ enter: 1000, leave: 200 }">...</transition>
```

### 在动态组件之间过渡

你所要做的就是将Vue动态组件包装在一个过渡元素中。跟过渡的基本用例相同！

模板代码如下：

```vue
 <transition name="fade" appear>
       <component :is='componentType' />
 </transition>
```

### 创建可重用的Vue Transition组件

在Vue中工作时要养成的一个好习惯是设计可重用的组件。

使用过渡很容易做到这一点——我们真正需要做的就是在根目录中放置过渡元素并插入组件槽，这样就可以添加更多内容。

代码如下所示：

```vue
 <template>
   <transition name="fade" appear>
     <slot></slot>
   </transition>
 </template>
```

现在，你不必再烦恼添加过渡样式、名称和所有内容到每个组件中，只需使用这个组件即可。

超棒的！在了解了`<transition>`元素之后，现在我们来制作动画。

## 构建动画

首先，我们需要一个由过渡元素包围的条件元素。起始单文件组件如下。

旋转图像：

```vue
<template>
  <div class="main-content">
    <transition name="rotate">
      <img v-if="show" src="../img/logo.png" />
    </transition>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        show: true
      }
    }
  }
</script>
```

接下来添加一个按钮，通过切换变量的值来转换元素的显示。

```vue
<button @click='show = !show'> Toggle </button>
```

设置元素的条件渲染后，我们通过两个类来实际设置动画样式：`rotate-enter-active`和`rotate-leave-active`，将过渡命名为`rotate`。

有一个很酷的技巧可以让离开动画与进入动画相同，那就是`reverse`！

```css
@keyframes rotate {
    0% { opacity: 0; transform: scale(0) rotate(-180deg); }
    100% { opacity: 1; transform: scale(1) rotate(0deg); }
}

.rotate-enter-active {
    animation: rotate 0.2s;
}

.rotate-leave-active {
    animation: rotate 0.2s reverse;
}
```

现在，如果我们查看组件并进行切换的话，应该能看到类似这样的动画。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ibugX3Hm3p4oML0aecPEUicn4vicwkCSQWaVPqOxABiaUXWmPdI4ZeliagwicsSTQzKoiak042Y3uViaR5wDMY1EnVQIdA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

这就是VueJS动画！你实现了！

## 使用第三方库

假设我们不想编写CSS动画怎么办？没关系，我们也有很多很棒的CSS动画库，可以很容易地合并到VueJS动画中。

在上面的第一个示例中，我们只用了`<transition>`元素生成的默认类名，但是我们可以覆盖这些值到任何类，在本例中，就是来自CSS库的类名。

对于示例，我们将使用Animate.css——要添加Animate.css，我们只需将CDN链接添加到`index.html`文件。

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.2/animate.min.css">
```

现在，在`<transition>`元素中，我们可以使用属性`enter-active-class`和`leave-active-class`将过渡连接到Animate.js。

请注意，对于Animate.js，我们需要添加类`animated`。

```vue
<transition 
  enter-active-class="animated fadeIn zoomIn" 
  leave-active-class="animated fadeOut zoomOut"
>
...
</transition>
```

超级简单。请看最后的成果：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ibugX3Hm3p4oML0aecPEUicn4vicwkCSQWarUhRSgDT8Xn8Sk42AViabYBRMWQ0ldqYZL22ibMqpicUiaSRLAoZwl0OPg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)