# 数据监听器 observers

> 用于监听和响应任何属性和数据字段的变化，类似**`Vue`**中的**`watch`**

```js
Component({
  attached: function() {
    this.setData({
      numberA: 1,
      numberB: 2,
    })
  },
  observers: {
    'numberA, numberB': function(numberA, numberB) {
      // 在 numberA 或者 numberB 被设置时，执行这个函数
      this.setData({
        sum: numberA + numberB
      })
    }
  }
})
```

1. 支持监听**属性**或**内部数据的变化**，可以同时**监听多个**，**监听子数据字段**。一次 `setData` 最多触发每个监听器一次。

   ```js
   Component({
     observers: {
       'some.subfield': function(subfield) {
         // 使用 setData 设置 this.data.some.subfield 时触发
         // （除此以外，使用 setData 设置 this.data.some 也会触发）
         subfield === this.data.some.subfield
       },
       'arr[12]': function(arr12) {
         // 使用 setData 设置 this.data.arr[12] 时触发
         // （除此以外，使用 setData 设置 this.data.arr 也会触发）
         arr12 === this.data.arr[12]
       },
     }
   })
   ```

2. **监听所有子数据字段的变化**，可以使用**通配符 `**`** 

   ```js
   Component({
     observers: {
       'some.field.**': function(field) {
         // 使用 setData 设置 this.data.some.field 本身或其下任何子数据字段时触发
         // （除此以外，使用 setData 设置 this.data.some 也会触发）
         field === this.data.some.field
       },
     },
     attached: function() {
       // 这样会触发上面的 observer
       this.setData({
         'some.field': { /* ... */ }
       })
       // 这样也会触发上面的 observer
       this.setData({
         'some.field.xxx': { /* ... */ }
       })
       // 这样还是会触发上面的 observer
       this.setData({
         'some': { /* ... */ }
       })
     }
   })
   ```

   ```js
   Component({
     observers: {
       '**': function() {
         // 每次 setData 都触发
       },
     },
   })
   ```

   **注意：**

   1. 数据监听器监听的是 `setData` 涉及到的数据字段，即使这些数据字段的值没有发生变化，数据监听器依然会被触发。

   2. 如果在数据监听器函数中使用 `setData` 设置本身监听的数据字段，可能会导致死循环，需要特别留意。

   2. 数据监听器和属性的 observer 相比，数据监听器更强大且通常具有更好的性能。

