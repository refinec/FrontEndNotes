## 如何实现？

数据双向绑定需要在属性前添加`model: `前缀：

```html
<input model:value="{{xxx}}" />
```

> **注意：**
>
> 目前，尚不能指定 data 路径，如`<input model:value="{{ a.b }}" />`，这样的表达式目前暂不支持。

## 在自定义组件(`Component`)中实现双向绑定

`custom-component.js`

```js
Component({
  properties: {
    myValue: String
  }
})
```

`custom-component.wxml`

```html
<input model:value="{{myValue}}" />
```

定义好之后使用：

```html
<custom-component model:my-value="{{pageValue}}" />
```

当`input`的值变更时，自定义组件的 `myValue` 属性会同时变更。这样页面的 `this.data.pageValue` 也会同时变更，页面 WXML 中所有绑定了 `pageValue` 的位置也会被一同更新。

> 另外，自定义组件还可以自己触发双向绑定更新。
>
> 做法就是：使用 `setData` 设置自身的属性。

例如：

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

