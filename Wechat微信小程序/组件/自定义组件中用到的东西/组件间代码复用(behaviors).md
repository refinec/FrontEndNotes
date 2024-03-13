> 每个 `behavior` 可以包含一组属性、数据、生命周期函数和方法。**组件引用它时，它的属性、数据和方法会被合并到组件中，生命周期函数也会在对应时机被调用。** 每个组件可以引用多个 `behavior` ，`behavior` 也可以引用其它 `behavior` 

```js
// my-behavior.js
module.exports = Behavior({
    behaviors: [], // 引入其它的 behavior
    properties: { // 同组件的属性
        myBehaviorProperty: {
            type: String
        }
    },
    data: { // 同组件的数据
        myBehaviorData: {}
    },
    methods: { // 同自定义组件的方法
        myBehaviorMethod: function(){}
    },
    
    // 生命周期函数
    created: function(){},
    attached: function(){},
		ready: function(){},
    moved: function(){},
    detached: function(){},
})
```

## 同名字段的覆盖和组合规则

* 如果有**同名的属性** (properties) 或**方法** (methods)：
  1. 若组件本身有这个属性或方法，则组件的属性或方法会覆盖 `behavior` 中的同名属性或方法；
  2. 若组件本身无这个属性或方法，则在组件的 `behaviors` 字段中定义靠后的 `behavior` 的属性或方法会覆盖靠前的同名属性或方法；
  3. 在 2 的基础上，若存在嵌套引用 `behavior` 的情况，则规则为：`父 behavior` 覆盖 `子 behavior` 中的同名属性或方法。

* 有同名的数据字段 (data)：
  1. 若同名的数据字段都是对象类型，会进行对象合并；
  2. 其余情况会进行数据覆盖，覆盖规则为：组件 > `父 behavior` > `子 behavior` 、 `靠后的 behavior` > `靠前的 behavior`。（优先级高的覆盖优先级低的，最大的为优先级最高）

* 生命周期函数不会相互覆盖，而是在对应触发时机被逐个调用：
  - 对于不同的生命周期函数之间，遵循组件生命周期函数的执行顺序；
  - 对于同种生命周期函数，遵循如下规则：
    - `behavior` 优先于组件执行；
    - `子 behavior` 优先于 `父 behavior` 执行；
    - `靠前的 behavior` 优先于 `靠后的 behavior` 执行；
  - 如果同一个 `behavior` 被一个组件多次引用，它定义的生命周期函数只会被执行一次。

## 内置 behaviors

> 自定义组件可以通过引用内置的 `behavior` 来获得内置组件的一些行为

### wx://form-field

```html
<form bindsubmit="formSubmit">
  <text>表单</text>
  <custom-form-field name="custom-name"></custom-form-field>
  <button form-type="submit">提交</button>
</form>
```

```js
// custom-form-field
Component({
  behaviors: ['wx://form-field'],
  data: {
    value: ''
  },
  methods: {
    onChange: function (e) {
      this.setData({
        value: e.detail.value,
      })
    }
  }
})
```

使自定义组件有类似于表单控件的行为。 form 组件可以识别这些自定义组件，并在 submit 事件中返回组件的字段名及其对应字段值。这将为它添加以下两个属性。

| 属性名 | 类型   | 描述             |
| :----- | :----- | :--------------- |
| name   | String | 在表单中的字段名 |
| value  | 任意   | 在表单中的字段值 |

### wx://form-field-group

> 使 form 组件可以识别到这个自定义组件内部的所有表单控件

```html
<form bindsubmit="submit">
  <custom-comp></custom-comp>
  <button form-type="submit">submit</button>
</form>
```

```html
<!-- custom-comp.wxml -->
<input name="name" />
<switch name="student" />
```

```js
// custom-comp.js
Component({
  behaviors: ['wx://form-field-group']
})
```

### wx://form-field-button

> 使 form 组件可以识别到这个自定义组件内部的 button ， 如果自定义组件内部有设置了 form-type 的 button ，它将被组件外的 form 接受。

```html
<form bindsubmit="submit">
  <input name="name" placeholder="请输入名字"></input>
  <custom-comp></custom-comp>
</form>
```

```html
<!-- custom-comp.wxml -->
<button form-type="submit">submit</button>
```

```js
// custom-comp.js
Component({
 behaviors: ['wx://form-field-button']
})
```

此时点击组件内的 button ，将触发 form 的 submit 事件
