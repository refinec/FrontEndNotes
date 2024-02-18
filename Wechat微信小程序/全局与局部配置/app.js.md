## 应用生命周期

> 每个小程序都需要在 `app.js` 中调用 `App` 方法注册小程序实例，绑定**生命周期回调函数**、**错误监听**和**页面不存在监听函数**等。
>

```js
App({
  	// 1.小程序启动时做一些初始操作。全局只触发一次。
    onLaunch(options) {
        // 参数也可以使用 wx.getLaunchOptionsSync 获取
    },
  
  	// 2.监听小程序 启动 或 从后台进入前台显示时触发.
    onShow (options) {
        // 也可以使用 wx.onAppShow 绑定监听
    },
  
  	// 3.监听小程序 从前台进入后台时触发
    onHide () {
        // 也可以使用 wx.onAppHide 绑定监听
    },
  	
  	// 4.小程序发生 脚本错误 或 API 调用报错时触发。
    onError (msg) {
        // 也可以使用 wx.onError 绑定监听。
        console.log(msg)
    },
  
  	// 5.小程序要 打开的页面不存在 时触发
    onPageNotFound(res){
        // 也可以使用 wx.onPageNotFound 绑定监听
        wx.redirectTo({ // 如果是 tabbar 页面，则使用 wx.switchTab
            url: 'pages/...'
        }) 
    },
  	
  	// 6.小程序有未处理的 Promise 拒绝时触发。
    onUnhandledRejection(callback){
        // 也可以使用 wx.onUnhandledRejection 绑定监听。
    },
  
  	// 7.系统切换主题时触发。
    onThemeChange(callback){
        // 也可以使用 wx.onThemeChange 绑定监听
    },
    
    // 添加 自定义函数 或 数据变量，用 this 可以访问
    globalData: 'I am global data',
})
```

## 方法

> 整个小程序只有一个 App 实例，是全部页面共享的。开发者可以通过 **`getApp`** 方法获取到全局唯一的 App 实例，**获取**App上的**数据**或**调用**开发者注册在 `App` 上的**函数**。

在某个`xxx.js`文件中访问 **全局变量** 或 **全局函数**

```js
const appInstance = getApp();
console.log(appInstance.globalData); // I am global data
```

