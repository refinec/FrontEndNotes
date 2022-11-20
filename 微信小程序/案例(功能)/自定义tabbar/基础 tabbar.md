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

   一、自定义tab 组件

   - `app.json`
   
     ```json
     {
         "tabBar": {
         "custom": true,
         "color": "#7A7E83",
         "selectedColor": "#3cc51f",
         "borderStyle": "black",
         "backgroundColor": "#ffffff",
         "list": [
           {
             "pagePath": "index/index",
             "iconPath": "image/icon_component.png",
             "selectedIconPath": "image/icon_component_HL.png",
             "text": "组件"
           },
           {
             "pagePath": "index/index2",
             "iconPath": "image/icon_API.png",
             "selectedIconPath": "image/icon_API_HL.png",
             "text": "接口"
           }
         ]
       }
     }
     ```
   
   - `index.json`
   
     ```json
     {
       "component": true
     }
     ```
   
   - `index.js`
   
     ```js
     Component({
       data: {
         selected: 0,
         color: "#7A7E83",
         selectedColor: "#3cc51f",
         list: [{
           pagePath: "/index/index",
           iconPath: "/image/icon_component.png",
           selectedIconPath: "/image/icon_component_HL.png",
           text: "组件"
         }, {
           pagePath: "/index/index2",
           iconPath: "/image/icon_API.png",
           selectedIconPath: "/image/icon_API_HL.png",
           text: "接口"
         }]
       },
       attached() {
       },
       methods: {
         switchTab(e) {
           const data = e.currentTarget.dataset
           const url = data.path
           wx.switchTab({url})
           this.setData({
             selected: data.index
           })
         }
       }
     })
     ```
   
   - `index.wxml`
   
     ```html
     <!--miniprogram/custom-tab-bar/index.wxml-->
     <cover-view class="tab-bar">
       <cover-view class="tab-bar-border"></cover-view>
       <cover-view wx:for="{{list}}" wx:key="index" class="tab-bar-item" data-path="{{item.pagePath}}" data-index="{{index}}" bindtap="switchTab">
         <cover-image src="{{selected === index ? item.selectedIconPath : item.iconPath}}"></cover-image>
         <cover-view style="color: {{selected === index ? selectedColor : color}}">{{item.text}}</cover-view>
       </cover-view>
     </cover-view>
     ```
   
   - `index.wxss`
   
     ```css
     .tab-bar {
       position: fixed;
       bottom: 0;
       left: 0;
       right: 0;
       height: 48px;
       background: white;
       display: flex;
       padding-bottom: env(safe-area-inset-bottom);
     }
     
     .tab-bar-border {
       background-color: rgba(0, 0, 0, 0.33);
       position: absolute;
       left: 0;
       top: 0;
       width: 100%;
       height: 1px;
       transform: scaleY(0.5);
     }
     
     .tab-bar-item {
       flex: 1;
       text-align: center;
       display: flex;
       justify-content: center;
       align-items: center;
       flex-direction: column;
     }
     
     .tab-bar-item cover-image {
       width: 27px;
       height: 27px;
     }
     
     .tab-bar-item cover-view {
       font-size: 10px;
     }
     ```
   
   二、使用vant组件库里面的 [tab组件](https://vant-contrib.gitee.io/vant-weapp/#/tabbar)
   
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
   
        
