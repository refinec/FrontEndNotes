## 小程序环境变量对象`wx.env`

* `wx.env.USER_DATA_PATH`

  文件系统中的用户目录路径

## WXML 页面相关

### 一、获取页面节点 `createSelectorQuery()`

> **`wx.createSelectorQuery()`** 用于获取节点属性、样式、在界面上的位置等信息。

在**自定义组件**或**包含自定义组件**的**页面中**，推荐使用 **`this.createSelectorQuery`** 来代替 [wx.createSelectorQuery](https://developers.weixin.qq.com/miniprogram/dev/api/wxml/wx.createSelectorQuery.html) ，这样可以确保在正确的范围内选择节点。

```js
const query = wx.createSelectorQuery()
query.select('#the-id').boundingClientRect(function(res){
  res.top // #the-id 节点的上边界坐标（相对于显示区域）
})
query.selectViewport().scrollOffset(function(res){
  res.scrollTop // 显示区域的竖直滚动位置
})
query.exec()
```

 **CSS 的选择器，但仅支持下列语法：**

- ID选择器：`#the-id`
- class选择器（可以连续指定多个）：`.a-class.another-class`
- 子元素选择器：`.the-parent > .the-child`
- 后代选择器：`.the-ancestor .the-descendant`
- 跨自定义组件的后代选择器：`.the-ancestor >>> .the-descendant`
- 多选择器的并集：`#a-node, .some-other-nodes`

### 二、WXML多节点布局相交状态`createIntersectionObserver`

`wx.createIntersectionObserver`用于监听**两个**或**多个组件节点**在布局位置上的相交状态。这一组API常常可以用于推断某些节点是否可以被用户看见、有多大比例可以被用户看见。

- **参照节点**：监听的参照节点，取它的布局区域作为参照区域。如果有多个参照节点，则会取它们布局区域的 **交集** 作为参照区域。页面显示区域也可作为参照区域之一。
- **目标节点**：监听的目标，默认只能是一个节点（使用 `selectAll` 选项时，可以同时监听多个节点）。
- **相交区域**：目标节点的布局区域与参照区域的相交区域。
- **相交比例**：相交区域占参照区域的比例。
- **阈值**：相交比例如果达到阈值，则会触发监听器的回调函数。阈值可以有多个。

**`wx.createIntersectionObserver(Object component, Object options)`** 在**自定义组件**或**包含自定义组件**的页面中，推荐使用 `this.createIntersectionObserver` 来代替 [wx.createIntersectionObserver](https://developers.weixin.qq.com/miniprogram/dev/api/wxml/wx.createIntersectionObserver.html) ，这样可以确保在正确的范围内选择节点。

参数：

* `component`

  自定义组件实例

* `options`

  | 属性           | 类型     | 默认值 | 必填 | 说明                                                         |
  | :------------- | :------- | :----- | :--- | :----------------------------------------------------------- |
  | `thresholds`   | number[] | [0]    | 否   | 一个数值数组，包含所有阈值。                                 |
  | `initialRatio` | number   | 0      | 否   | 初始的相交比例，如果调用时检测到的相交比例与这个值不相等且达到阈值，则会触发一次监听器的回调函数。 |
  | `observeAll`   | boolean  | false  | 否   | 是否同时观测多个目标节点（而非一个），如果设为 true ，observe 的 targetSelector 将选中多个节点（注意：同时选中过多节点将影响渲染性能） |

1. 在目标节点（用选择器 `.target-class` 指定）每次进入或离开页面显示区域时，触发回调函数。

   ```js
   Page({
     onLoad: function(){
       this._observer = wx.createIntersectionObserver().relativeToViewport().observe('.target-class', (res) => {
         res.id // 目标节点 id
         res.dataset // 目标节点 dataset
         res.intersectionRatio // 相交区域占目标节点的布局区域的比例
         res.intersectionRect // 相交区域
         res.intersectionRect.left // 相交区域的左边界坐标
         res.intersectionRect.top // 相交区域的上边界坐标
         res.intersectionRect.width // 相交区域的宽度
         res.intersectionRect.height // 相交区域的高度
       })
     },
     onUnload() {
       if (this._observer) this._observer.disconnect()
     }
   })
   ```

2. 在目标节点（用选择器 `.target-class` 指定）与参照节点（用选择器 `.relative-class` 指定）在页面显示区域内相交或相离，且相交或相离程度达到目标节点布局区域的20%和50%时，触发回调函数

   ```js
   Page({
     onLoad: function(){
       this._observer = wx.createIntersectionObserver(this, {
         thresholds: [0.2, 0.5]
       }).relativeTo('.relative-class').relativeToViewport().observe('.target-class', (res) => {
         res.intersectionRatio // 相交区域占目标节点的布局区域的比例
         res.intersectionRect // 相交区域
         res.intersectionRect.left // 相交区域的左边界坐标
         res.intersectionRect.top // 相交区域的上边界坐标
         res.intersectionRect.width // 相交区域的宽度
         res.intersectionRect.height // 相交区域的高度
       })
     },
     onUnload() {
       if (this._observer) this._observer.disconnect()
     }
   })
   ```

**注意📢：**与页面显示区域的相交区域并不准确代表用户可见的区域，因为参与计算的区域是“布局区域”，布局区域可能会在绘制时被其他节点裁剪隐藏（如遇祖先节点中 overflow 样式为 hidden 的节点）或遮盖（如遇 fixed 定位的节点）。

