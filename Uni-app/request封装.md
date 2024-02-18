```js
const BASE_URL = "http://localhost:3000";

export default myRequest = (options) => {
	return new Promise((resolve, reject) => {
		uni.request({
			url: BASE_URL + options.url,
			method: options.method || "GET",
			data: options.data || {},
			success: (res) => {
				if (res.data.status !== 0) {
					return uni.showToast({
						title: '获取数据失败!'
					})
				},
				resolve(res);
			},
			fail: (error) => {
				uni.showToast({
					title: '接口请求失败!'
				})
				reject(error)
			}
		})
	})
}
```

在`main.js`中注册

```js
import myRequest from "./api/myRequest.js";
Vue.prototpe.$myRequest = myRequest;
```

