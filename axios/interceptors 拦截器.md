## 拦截器

> 在**请求**或**响应**被 **`then`** 或 **`catch `** 处理前拦截它们

添加请求拦截器：

```javascript
axios.interceptors.request.use(function(config){
    // 在发送请求之前做些什么,如公共配置
    return config;
},function(error){
    // 对请求错误做些什么
    return Promise.reject(error)
});
```

添加响应拦截器：

```js
axios.interceptors.response.use(function(response){
    // 对响应数据做点什么
    return response;
},function(error){
    // 对响应错误做点什么
    return Promise.reject(error)
})
```

稍后移除拦截器：

```javascript
var myInterceptor = axios.interceptors.request.use(function(){/*..*/})
axios.interceptors.request.eject(myInterceptor)
```

## 示例

```javascript
/**
 * 添加axios请求拦截器
 */
requestInterceptor() {
    HTTP.interceptors.request.use(
        (config) => {
            // loadingShow() 请求前展示loading
            // 重写公共url和请求头
            config.baseURL = this.$store.getters["fm/settings/baseUrl"];
            config.headers = this.$store.getters["fm/settings/headers"];

            this.$store.commit("fm/messages/addLoading");

            return config;
        },
        (error) => {
			// 请求错误 一般http状态码以4开头，常见：401超时，404 not found
            this.$store.commit("fm/messages/subtractLoading");
            return Promise.reject(error);
        }
    );
}

/**
 * 添加axios响应拦截器
 */
responseInterceptor() {
    HTTP.interceptors.response.use(
        (response) => {
			// loadingHide() 请求后隐藏loading
            this.$store.commit("fm/messages/subtractLoading");

            // 如果有消息文本，创建提醒
            if (Object.prototype.hasOwnProperty.call(response.data, "result")) {
                if (response.data.result.message) {
                    const message = {
                        status: response.data.result.status,
                        message: Object.prototype.hasOwnProperty.call(
                            this.lang.response,
                            response.data.result.message
                        )
                        ? this.lang.response[response.data.result.message]
                        : response.data.result.message,
                    };

                    // 显示提醒
                    EventBus.$emit("addNotification", message);

                    // 提交消息
                    this.$store.commit("fm/messages/setActionResult", message);
                }
            }

            return response;
        },
        (error) => {
			// 响应错误处理 一般http状态码以5开头，500 系统错误， 502 系统重启
            this.$store.commit("fm/messages/subtractLoading");

            const errorMessage = {
                status: 0,
                message: "",
            };

            const errorNotificationMessage = {
                status: "error",
                message: "",
            };

            // 添加消息
            if (error.response) {
                errorMessage.status = error.response.status;

                if (error.response.data.message) {
                    const trMessage = Object.prototype.hasOwnProperty.call(
                        this.lang.response,
                        error.response.data.message
                    )
                    ? this.lang.response[error.response.data.message]
                    : error.response.data.message;

                    errorMessage.message = trMessage;
                    errorNotificationMessage.message = trMessage;
                } else {
                    errorMessage.message = error.response.statusText;
                    errorNotificationMessage.message = error.response.statusText;
                }
            } else if (error.request) {
                errorMessage.status = error.request.status;
                errorMessage.message = error.request.statusText || "Network error";
                errorNotificationMessage.message =
                    error.request.statusText || "Network error";
            } else {
                errorMessage.message = error.message;
                errorNotificationMessage.message = error.message;
            }

            // 设置错误消息
            this.$store.commit("fm/messages/setError", errorMessage);

            // 显示提示
            EventBus.$emit("addNotification", errorNotificationMessage);

            return Promise.reject(error);
        }
    );
}
```



