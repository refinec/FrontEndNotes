# 双向绑定

```js
// custom-component.js
Component({
  properties: { // 类似 Vue组件中的 prop 属性
    myValue: String
  }
})
```

```html
<!-- custom-component.wxml -->
<input model:value="{{myValue}}" />
```

自定义组件将自身的 `myValue` 属性双向绑定到了组件内输入框的 `value` 属性上。当输入框的值变更时，自定义组件的 `myValue` 属性会同时变更，这样，页面的 `this.data.pageValue` 也会同时变更，页面 WXML 中所有绑定了 `pageValue` 的位置也会被一同更新。

```html
<custom-component model:my-value="{{pageValue}}" />
```

**自定义组件还可以自己触发双向绑定更新**，做法就是：使用 `setData` 设置自身的属性。例如：

当组件使用 `setData` 更新 `myValue` 时，页面的 `this.data.pageValue` 也会同时变更，页面 WXML 中所有绑定了 `pageValue` 的位置也会被一同更新。

```js
// custom-component.js
Component({
  properties: {
    myValue: String
  },
  methods: {
    update: function() {
      // 更新 myValue
      this.setData({
        myValue: 'leaf'
      })
    }
  }
})
```

```html
<custom-component model:my-value="{{pageValue}}" />
```

