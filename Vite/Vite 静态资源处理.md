## Vite 静态资源处理

导入一个静态资源会返回解析后的 URL：

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

### 一、更改资源被引入的方式

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

### 二、动态导入（带变量）

```js
const module = await import(`./dir/${file}.js`);
```

### 三、 Glob 导入（仅字面量）

> **注意🔈**：所有 `import.meta.glob` 的参数都必须以字面量传入。你 **不** 可以在其中使用 变量 或 表达式。

#### 1. `import.meta.glob`

Vite 支持使用特殊的 `import.meta.glob` 函数从文件系统 <u>**懒加载**</u> 多个模块，通过动态导入实现，并会在构建时分离为独立的 **`chunk`**：

```js
const modules = import.meta.glob('./dir/*.js');

// 被转译为下面的样子
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js'),
}

// 所以你可以遍历 modules 对象的 key 值来访问相应的模块
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

如果要 **直接引入所有的模块**（例如依赖于这些模块中的副作用首先被应用），传入 `{ eager: true }` 作为第二个参数即可：

```js
const modules = import.meta.glob('./dir/*.js', { eager: true });

// 被转译为下面的样子
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1,
}
```

#### 2. 以`字符串` / `url`的形式导入

```js
const modules = import.meta.glob('./dir/*.js', { as: 'raw', eager: true })

// 被转译为下面的样子
const modules = {
  './dir/foo.js': 'export default "foo"\n',
  './dir/bar.js': 'export default "bar"\n',
}
```

```js
const modules = import.meta.glob('./dir/*.js', { as: 'url', eager: true })
```

#### 3. 匹配模式

##### 1. 多个匹配模式

```js
const modules = import.meta.glob(['./dir/*.js', './another/*.js'])
```

##### 2. 反面(忽略文件)匹配模式

> 以 `!` 作为前缀忽略结果中的一些文件

```js
const modules = import.meta.glob(['./dir/*.js', '!**/bar.js'])

// 
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
}
```

#### 4. 具名导入（只导入模块中的特定内容）

```js
const modules = import.meta.glob('./dir/*.js', {
  import: 'setup',
  eager: true, // 与 eager 一同存在时，甚至可以对这些模块进行 tree-shaking
})

// 被转译为下面的样子
const modules = {
  './dir/foo.js': () => import('./dir/foo.js').then((m) => m.setup),
  './dir/bar.js': () => import('./dir/bar.js').then((m) => m.setup),
}
```

设置 `import` 为 `default` 可以加载默认导出

```js
const modules = import.meta.glob('./dir/*.js', {
  import: 'default',
  eager: true,
})

// 被转译为下面的样子
import __glob__0_0 from './dir/foo.js'
import __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1,
}
```

#### 5. 对导入自定义查询

> 使用 `query` 选项来提供对导入的自定义查询，以供其他插件使用

```js
const modules = import.meta.glob('./dir/*.js', {
  query: { foo: 'bar', bar: true },
})

// 被转译为下面的样子
const modules = {
  './dir/foo.js': () => import('./dir/foo.js?foo=bar&bar=true'),
  './dir/bar.js': () => import('./dir/bar.js?foo=bar&bar=true'),
}
```

### 四、`WebAssembly `导入 ([链接](https://cn.vitejs.dev/guide/features.html#webassembly))

### 五、`Web Workers` 导入 ([链接](https://cn.vitejs.dev/guide/features.html#web-workers))

### 六、动态导入静态图片 `new URL(url, import.meta.url)`

[`import.meta.url`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) 是一个 ESM 的原生功能，会暴露当前模块的 URL。将它与原生的 [URL 构造器](https://developer.mozilla.org/en-US/docs/Web/API/URL) 组合使用，在一个 JavaScript 模块中，通过相对路径我们就能得到一个被完整解析的静态资源 URL：

```js
const imgUrl = new URL('./img.png', import.meta.url).href;

document.getElementById('hero-img').src = imgUrl;
```

通过**字符串模板**支持动态 URL：

```js
function getImageUrl(name) {
  return new URL(`./dir/${name}.png`, import.meta.url).href;
}
```

这个 URL 字符串必须是静态的，这样才好分析。否则代码将被原样保留、因而在 `build.target` 不支持 `import.meta.url` 时会导致运行时错误。

```js
// Vite 不会转换这个
const imgUrl = new URL(imagePath, import.meta.url).href
```

> 注意🔈：`import.meta.url`无法在 SSR 中使用
>
> 如果你正在以服务端渲染模式使用 Vite 则此模式不支持，因为 `import.meta.url` 在浏览器和 Node.js 中有不同的语义。服务端的产物也无法预先确定客户端主机 URL。
