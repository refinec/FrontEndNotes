## 自定义 tabbar 的好处

1. **小程序自带的tabbar** 当设置 `position `为 `top `时，将不会显示 `icon`。自定义 tabbar 不受限制
2. **小程序自带的tabBar** 中的 `list `是一个数组，只能配置最少2个、最多5个 tab，tab 按数组的顺序排序。**自定义 tabbar** 可自定义扩展
3. 在实际开发过程中，小程序**自带的tabbar**样式并不能满足设计提供的开发需求，所以需要我们自定义一套属于自己的tabbar。

## 如何自定义 tabbar

> 参考官方网站：[自定义 tabBar | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/custom-tabbar.html)

### 1. 设置 `custom` 字段、`usingComponents`选项

- 在 `app.json` 中的 `tabBar` 项指定 `custom` 字段，同时其余 `tabBar` 相关配置也补充完整。
- 所有 tab 页的 json 里需声明 `usingComponents` 项，也可以在 `app.json` 全局开启。

```js
{
  "tabBar": {
    "custom": true, // 必需
    "color": "#000000",
    "selectedColor": "#000000",
    "backgroundColor": "#000000",
    "list": [{ // 必需。该list数组保留，需要保持完整配置项以及低版本里面不适用 自定义tabBar 时向下兼容
      "pagePath": "page/component/index",
      "text": "组件"
    }, {
      "pagePath": "page/API/index",
      "text": "接口"
    }]
  },
  "usingComponents": {} // 可在tab 页的 json 里声明
}
```

### 2. 添加 tabBar 代码文件

1. 在**根目录**下新建 `custom-tab-bar` 目录(固定)，并创建入口文件(目录结构是固定)

   ```
   custom-tab-bar/index.js
   custom-tab-bar/index.json
   custom-tab-bar/index.wxml
   custom-tab-bar/index.wxss
   ```

2. 编写 tabbar 代码

   > 用自定义组件的方式编写即可，该自定义组件完全接管 tabBar 的渲染。另外，自定义组件新增 `getTabBar` 接口，可获取当前页面下的自定义 tabBar 组件实例。

   - `index.json`

     引入vant组件库里面的 [tab组件](https://vant-contrib.gitee.io/vant-weapp/#/tabbar)：


     ```json
     {
         "component": true,
         "usingComponents": {
             "van-tabbar": "/miniprogram_npm/@vant/weapp/tabbar/index",
             "van-tabbar-item": "/miniprogram_npm/@vant/weapp/tabbar-item/index"
         }
     }
     ```

   - `index.wxml`

     在`wxml`中写入 vant官网提供的tabbar标签栏

     ```html
     <van-tabbar active="{{ active }}" bind:change="onChange">
       <van-tabbar-item icon="wap-home">首页</van-tabbar-item>
       <van-tabbar-item icon="shopping-cart" info="5">购物车</van-tabbar-item>
       <van-tabbar-item icon="manager" dot>我的</van-tabbar-item>
     </van-tabbar>
     ```

   - `index.js`

     在js文件下写入官网提供的变量以及点击切换按钮`change`事件。

     ```js
     Page({
       data: {
         active: 0,
       },
       onChange(event) {
         // event.detail 的值为当前选中项的索引
         this.setData({ active: event.detail });
       },
     });
     ```

     
