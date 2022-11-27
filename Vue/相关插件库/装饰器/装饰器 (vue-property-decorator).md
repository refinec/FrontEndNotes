# `vue-property-decorator`

> 👉🏻[详细阅读](https://github.com/kaorun343/vue-property-decorator)

- [`@Prop`](https://github.com/kaorun343/vue-property-decorator#Prop)
- [`@PropSync`](https://github.com/kaorun343/vue-property-decorator#PropSync)
- [`@Model`](https://github.com/kaorun343/vue-property-decorator#Model)
- [`@ModelSync`](https://github.com/kaorun343/vue-property-decorator#ModelSync)
- [`@Watch`](https://github.com/kaorun343/vue-property-decorator#Watch)
- [`@Provide`](https://github.com/kaorun343/vue-property-decorator#Provide)
- [`@Inject`](https://github.com/kaorun343/vue-property-decorator#Provide)
- [`@ProvideReactive`](https://github.com/kaorun343/vue-property-decorator#ProvideReactive)
- [`@InjectReactive`](https://github.com/kaorun343/vue-property-decorator#ProvideReactive)
- [`@Emit`](https://github.com/kaorun343/vue-property-decorator#Emit)
- [`@Ref`](https://github.com/kaorun343/vue-property-decorator#Ref)
- [`@VModel`](https://github.com/kaorun343/vue-property-decorator#VModel)
- `@Component` (**provided by** [vue-class-component](https://github.com/vuejs/vue-class-component))
- `Mixins` (the helper function named `mixins` **provided by** [vue-class-component](https://github.com/vuejs/vue-class-component))

## @Component

> 格式：`@Component(options:ComponentOptions = {})`
>
> `@Component` 装饰器可以接收一个对象作为参数，可以在对象中声明 `components` ，`filters`，`directives` 等未提供装饰器的选项，虽然也可以在 `@Component` 装饰器中声明 `computed`，`watch` 等，但并不推荐这么做，因为在访问 this 时，编译器会给出错误提示，即使没有组件也不能省略`@Component`，否则会报错。

```js
import {Component,Vue} from 'vue-property-decorator';
import {componentA,componentB} from '@/components';
 
 @Component({
  name:"name",
    components:{
        componentA,
        componentB,
    },
    directives: {
        focus: {
            // 指令的定义
            inserted: function (el) {
                el.focus()
            }
        }
    }
})
export default class YourCompoent extends Vue{ }
```

## @Prop

> 父子组件之间值的传递
>
> 格式：`@Prop(options: (PropOptions | Constructor[] | Constructor) = {})`
>
> - `Constructor` ：例如 `String`，`Number`，`Boolean` 等，指定 `prop` 的类型；
> -  `Constructor[]` ：指定 `prop` 的可选类型；
> -  `PropOptions` ：可以使用以下选项： `type`，`default`，`required`，`validator` 。

```js
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default_value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}
```

> 注意📢：属性的ts类型后面需要加上 `undefined` 类型；或者在属性名后面加上`!`，表示 `非null` 和 `非undefined`的断言，否则编译器会给出错误提示；指定默认值必须使用上面例子中的写法，如果直接在属性名后面赋值，会重写这个属性，并且会报错。

等同于

```js
export default {
  props: {
    propA: {
      type: Number,
    },
    propB: {
      default: 'default value',
    },
    propC: {
      type: [String, Boolean],
    },
  },
}
```

## @PropSync

> `@PropSync` 会生成一个新的计算属性。`@PropSync` 需要配合父组件的 `.sync` 修饰符使用
>
> 格式：`@PropSync(propName: string, options: (PropOptions | Constructor[] | Constructor) = {})`
>
> - `propName`: string 表示父组件传递过来的属性名；
> -  `options`: 与 @Prop 的第一个参数一致；

```js
import { Vue, Component, PropSync } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @PropSync('name', { type: String }) syncedName!: string
}
```

等同于

```js
export default {
  props: {
    name: {
      type: String,
    },
  },
  computed: {
    syncedName: {
      get() {
        return this.name
      },
      set(value) {
        this.$emit('update:name', value)
      },
    },
  },
}
```

## @Model

> `@Model` 装饰器允许我们在一个组件上自定义 `v-model`。
>
> 格式：`@Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {})`
>
> -  `event`: string 事件名。
> -  `options`: 与 @Prop 的第一个参数一致。

```js
import { Vue, Component, Model } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Model('change', { type: Boolean }) readonly checked!: boolean
}
```

等同于

```js
export default {
  model: {
    prop: 'checked',
    event: 'change',
  },
  props: {
    checked: {
      type: Boolean,
    },
  },
}
```

## @ModelSync

> 格式：`@ModelSync(propName: string, event?: string, options: (PropOptions | Constructor[] | Constructor) = {})`

```js
import { Vue, Component, ModelSync } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @ModelSync('checked', 'change', { type: Boolean })
  readonly checkedValue!: boolean
}
```

等同于

```js
export default {
  model: {
    prop: 'checked',
    event: 'change',
  },
  props: {
    checked: {
      type: Boolean,
    },
  },
  computed: {
    checkedValue: {
      get() {
        return this.checked
      },
      set(value) {
        this.$emit('change', value)
      },
    },
  },
}
```

## @Watch

> 侦听开始，发生在 `beforeCreate` 勾子之后， `created` 勾子之前
>
> 格式：`@Watch(path: string, options: WatchOptions = {})`
>
> - `path`: string 被侦听的属性名；
> -  `options`?: WatchOptions={} options 可以包含两个属性 ：
>   -  `immediate`?:boolean 侦听开始之后是否立即调用该回调函数；
>   -  `deep`?:boolean 被侦听的对象的属性被改变时，是否调用该回调函数；

```js
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Watch('child')
  onChildChanged(val: string, oldVal: string) {}

  @Watch('person', { immediate: true, deep: true })
  onPersonChanged1(val: Person, oldVal: Person) {}

  @Watch('person')
  onPersonChanged2(val: Person, oldVal: Person) {}

  @Watch('person')
  @Watch('child')
  onPersonAndChildChanged() {}
}
```

等同于

```js
export default {
  watch: {
    child: [
      {
        handler: 'onChildChanged',
        immediate: false,
        deep: false,
      },
      {
        handler: 'onPersonAndChildChanged',
        immediate: false,
        deep: false,
      },
    ],
    person: [
      {
        handler: 'onPersonChanged1',
        immediate: true,
        deep: true,
      },
      {
        handler: 'onPersonChanged2',
        immediate: false,
        deep: false,
      },
      {
        handler: 'onPersonAndChildChanged',
        immediate: false,
        deep: false,
      },
    ],
  },
  methods: {
    onChildChanged(val, oldVal) {},
    onPersonChanged1(val, oldVal) {},
    onPersonChanged2(val, oldVal) {},
    onPersonAndChildChanged() {},
  },
}
```

## @Provide / @Inject

> 格式：
>
> - `@Provide(key?: string | symbol)` 
> -  `@Inject(options?: { from?: InjectKey, default?: any } | InjectKey)`

```js
import { Component, Inject, Provide, Vue } from 'vue-property-decorator'

const symbol = Symbol('baz')

@Component
export class MyComponent extends Vue {
  @Inject() readonly foo!: string
  @Inject('bar') readonly bar!: string
  @Inject({ from: 'optional', default: 'default' }) readonly optional!: string
  @Inject(symbol) readonly baz!: string

  @Provide() foo = 'foo'
  @Provide('bar') baz = 'bar'
}
```

等同于

```js
const symbol = Symbol('baz')

export const MyComponent = Vue.extend({
  inject: {
    foo: 'foo',
    bar: 'bar',
    optional: { from: 'optional', default: 'default' },
    baz: symbol,
  },
  data() {
    return {
      foo: 'foo',
      baz: 'bar',
    }
  },
  provide() {
    return {
      foo: this.foo,
      bar: this.baz,
    }
  },
})
```

## @ProvideReactive / @InjectReactive

> 这些装饰器是`@Provide`和`@Inject`的响应式版本。如果提供的值被父组件修改，则子组件可以捕捉此修改。

> 格式：
>
> - ` @ProvideReactive(key?: string | symbol)` 
> - `@InjectReactive(options?: { from?: InjectKey, default?: any } | InjectKey)`

```js
const key = Symbol()
@Component
class ParentComponent extends Vue {
  @ProvideReactive() one = 'value'
  @ProvideReactive(key) two = 'value'
}

@Component
class ChildComponent extends Vue {
  @InjectReactive() one!: string
  @InjectReactive(key) two!: string
}
```

## @Emit

> 如果事件的名称不是通过`event`参数提供的，则使用函数名。在这种情况下，`camelCase` 名称将被转换为 `kebab-case`。
>
> 如果返回值是一个`promise`，则在触发之前解析它。@Emit 会将回调函数的返回值作为第二个参数，如果返回值是一个 Promise 对象， emit 会在 Promise 对象被标记为 resolved 之后触发。 @Emit 的回调函数的参数，会放在其返回值之后，一起被emit 当做参数使用。

> 格式：`@Emit(event?: string)`

```js
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  count = 0

  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  onInputChange(e) {
    return e.target.value
  }

  @Emit()
  promise() {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```

等同于

```js
export default {
  data() {
    return {
      count: 0,
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    onInputChange(e) {
      this.$emit('on-input-change', e.target.value, e)
    },
    promise() {
      const promise = new Promise((resolve) => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })

      promise.then((value) => {
        this.$emit('promise', value)
      })
    },
  },
}
```

## @Ref

> 获取DOM元素引用

> 格式：`@Ref(refKey?: string)`
>
> - `refKey`：可选参数，用来指向元素或子组件的引用信息。如果没有提供这个参数，会使用装饰器后面的属性名充当参数

```js
import { Vue, Component, Ref } from 'vue-property-decorator'

import AnotherComponent from '@/path/to/another-component.vue'

@Component
export default class YourComponent extends Vue {
  @Ref() readonly anotherComponent!: AnotherComponent
  @Ref('aButton') readonly button!: HTMLButtonElement
}
```

等同于

```js
export default {
  computed() {
    anotherComponent: {
      cache: false,
      get() {
        return this.$refs.anotherComponent as AnotherComponent
      }
    },
    button: {
      cache: false,
      get() {
        return this.$refs.aButton as HTMLButtonElement
      }
    }
  }
}
```

## @VModel

> 格式：`@VModel(propsArgs?: PropOptions)`

```js
import { Vue, Component, VModel } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @VModel({ type: String }) name!: string
}
```

等同于

```js
export default {
  props: {
    value: {
      type: String,
    },
  },
  computed: {
    name: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      },
    },
  },
}
```

