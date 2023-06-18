## Vite 静态资源处理

导入一个静态资源会返回解析后的 URL：

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

### 更改资源被引入的方式

添加一些特殊的**查询参数**可以更改资源被引入的方式：

```js
// 1. 显式加载资源为一个 URL
import assetAsURL from './asset.js?url'

// 2. 以字符串形式加载资源
import assetAsString from './shader.glsl?raw'

// 3. 加载为 Web Worker
import Worker from './worker.js?worker'

// 4. 在构建时 Web Worker 内联为 base64 字符串
import InlineWorker from './worker.js?worker&inline'

```

