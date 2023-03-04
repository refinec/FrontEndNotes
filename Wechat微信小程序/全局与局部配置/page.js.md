## `page.js` 页面脚本

> `page.js`不是真的为page页面的文件，page根据具体页面更改

`page.json`

```json
{
  "restartStrategy": "homePageAndLatestPage"
}
```

`page.js`

```js
Page({
    data: {
        text: "This is page data."
    },
  	
  	// 页面的组件选项，同 Component 构造器 中的 options
    options:{ },
  	
  	// 1.页面创建时执行(生命周期)
    onLoad: function(options) {
        // 可获取路由中的参数
      
        var prevExitState = this.exitState // 尝试获得上一次退出前 onSaveExitState 保存的数据
        if (prevExitState !== undefined) { // 如果是根据 restartStrategy 配置进行的冷启动，就可以获取到
            prevExitState.myDataField === 'myData' 
        }
    },
  
  	// 2.页面出现在前台时执行(生命周期)
    onShow: function() { },
  	
  	// 3.页面首次渲染完毕时执行(生命周期)
    onReady: function() {
        // 可对界面内容进行设置的 API 如wx.setNavigationBarTitle
    },
  
  	// 4.页面隐藏/切入后台时触发(生命周期) 
    onHide: function() {
        // 如 wx.navigateTo触发 或 底部tab 切换到其他页面、小程序切入后台等
    },
  
  	// 5.页面销毁时执行(生命周期)
    onUnload: function() {
        // 如wx.redirectTo 或 wx.navigateBack到其他页面时
    },
  
    // 每当小程序可能被销毁之前，页面回调函数 onSaveExitState 会被调用。
    // 如果想保留页面中的状态，可以在这个回调函数中“保存”一些数据，下次启动时可以通过 exitState 获得这些已保存数据.
    //在某些特殊情况下（如微信客户端直接被系统杀死），这个方法将不会被调用，下次冷启动也不遵循 restartStrategy 的配置，而是直接从首页冷启动。
    onSaveExitState: function() {
        var exitState = { myDataField: 'myData' } // 需要保存的数据
        return {
            data: exitState, // 需要保存的数据（只能是 JSON 兼容的数据）
            expireTimeStamp: Date.now() + 24 * 60 * 60 * 1000 // 超时时刻,在这个时刻后，保存的数据保证一定被丢弃，默认为 (当前时刻 + 1 天)
        }
    },
  
  	// 1.下拉刷新时执行
    onPullDownRefresh: function() {
        // 需要在 app.json 的 window选项中 或 页面配置中 开启enablePullDownRefresh
    		// 可以通过 wx.startPullDownRefresh函数 触发下拉刷新，调用后触发下拉刷新动画，效果与用户手动下拉刷新一致
        // 当处理完数据刷新后，wx.stopPullDownRefresh函数 可以停止当前页面的下拉刷新
    },
  	
  	// 2.上拉刷新时执行
    onReachBottom: function() {
        // 需要在 app.json 的 window选项中 或 页面配置中设置触发距离 onReachBottomDistance
    		// 在触发距离内滑动期间，本事件只会被触发一次
    },
  
  	// 1.页面被用户分享时执行,如用户点击右上角转发、button 组件 open-type="share"
    onShareAppMessage: function () {
        // 只有定义了此事件处理函数，右上角菜单才会显示“转发”按钮
        const promise = new Promise(resolve => {
            setTimeout(() => {
                resolve({
                    title: '自定义转发标题'
                })
            }, 2000)
        })
        return { // 返回一个对象，用于自定义转发内容
        	title: '自定义转发标题', // 当前小程序名称
            path: '/page/user?id=123', // 转发路径，当前页面 path ，必须是以 / 开头的完整路径
            imageUrl:"", // 
            promise 
        }
    },
  
  	// 2.点击右上角转发到朋友圈
    onShareTimeline: function () { },
  
  	// 3.用户点击右上角收藏
    onAddToFavorites: function (res) {
      // 页面中包含web-view组件时，返回当前web-view的url
      console.log('webViewUrl: ', res.webViewUrl) 
      
    	return { // 返回一个对象，用于自定义收藏内容
            title: "自定义标题", // 页面标题或账号名称
            imageUrl:"http://demo.png", // 页面截图
            query:"name=xxx&age=xxx" // 当前页面的query
        }
    },
  	
  	// 页面滚动时执行
    onPageScroll: function(obj) {
        // 在需要的时候才在 page 中定义此方法，不要定义空方法。以减少不必要的事件派发对渲染层-逻辑层通信的影响。 注意：请避免在 onPageScroll 中过于频繁的执行 setData 等引起逻辑层-渲染层通信的操作。尤其是每次传输大量数据，会影响通信耗时。
        obj.scrollTop // 页面在垂直方向已滚动的距离（单位px）
    },
  	// 页面尺寸变化时执行,详见 响应显示区域变化
    onResize: function() { },
  	// 当前是 tab 页时，点击 tab 时触发
    onTabItemTap(item) {
        console.log(item.index)
        console.log(item.pagePath)
        console.log(item.text)
    },
  
    // 自定义事件响应函数
    viewTap: function() {
        this.setData({
            text: '这次要改变的数据	',
            'array[2].message':1,
            'a.b.c.d':2
        }, function() {
            // 引起的界面更新渲染完毕后的回调函数
        })
		},
  
    // 自由数据
    customData: {
        hi: 'MINA'
    }
})
```

## `page`页面间通信

如果一个页面由另一个页面通过 [`wx.navigateTo`](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateTo.html) 打开，这两个页面间将建立一条数据通道：

- 被打开的页面可以通过 `this.getOpenerEventChannel()` 方法来获得一个 `EventChannel` 对象；
- `wx.navigateTo` 的 `success` 回调中也包含一个 `EventChannel` 对象。

这两个 `EventChannel` 对象间可以使用 `emit` 和 `on` 方法相互发送、监听事件。

`index.js `文件

```js
const app = getApp()

Page({
  jump: function () {
    wx.navigateTo({
      url: './test',
      events: {
        acceptDataFromOpenedPage: function (data) {
          console.log(data)
        },
      },
      success: function (res) {
        res.eventChannel.emit('acceptDataFromOpenerPage', { data: 'send from opener page' })
      }
    })
  },
})
```

`test.js`文件

```js
Page({
  onLoad: function (option) {
    const eventChannel = this.getOpenerEventChannel()
    eventChannel.emit('acceptDataFromOpenedPage', { data: 'send from opened page' });
    eventChannel.on('acceptDataFromOpenerPage', function (data) {
      console.log(data)
    })
  }
})
```

## 使用 `behaviors` 来组件间代码复用

> `behaviors` 可以用来让多个页面**有相同的数据**字段和**方法**

`my-behavior.js`

```js
module.exports = Behavior({
  data: {
    sharedText: 'This is a piece of data shared between pages.'
  },
  methods: {
    sharedMethod: function() {
      this.data.sharedText === 'This is a piece of data shared between pages.'
    }
  }
})
```

在`page-a.js`中导入

```js
var myBehavior = require('./my-behavior.js')
Page({
  behaviors: [myBehavior], // 类似vue中的mixins
  onLoad: function() {
    this.data.sharedText === 'This is a piece of data shared between pages.'
  }
})
```

## 页面路由

### 方法：`getCurrentPages()`

> `getCurrentPages`方法用于获取当前页面栈，可以用来浏览历史回退

###  `Page.route`页面路由信息

到当前页面的路径，类型为`String`。

```javascript
Page({
  onShow: function() {
    console.log(this.route)
  }
})
```

### 路由方式

* 打开新页面： [wx.navigateTo](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateTo.html)
* 页面重定向：[wx.redirectTo](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.redirectTo.html)
* 页面返回：[wx.navigateBack](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html)
* Tab切换：[wx.switchTab](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.switchTab.html)
* 重启动：[wx.reLaunch](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.reLaunch.html)

具体使用方式：https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/route.html

##### 注意事项

- `navigateTo`, `redirectTo` 只能打开非 tabBar 页面。
- `switchTab` 只能打开 tabBar 页面。
- `reLaunch` 可以打开任意页面。
- 页面底部的 tabBar 由页面决定，即只要是定义为 tabBar 的页面，底部都有 tabBar。
- 调用页面路由带的参数可以在目标页面的`onLoad`中获取。