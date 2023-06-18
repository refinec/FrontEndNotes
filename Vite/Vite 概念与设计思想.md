## Vite 概念与设计思想

* Vite 由两部分组成：

  * 一个开发服务，服务与开发环境，ESM + HMR
  * 一套构建指令，服务于生产环境，用Rollup

* Vite 将模块区分为**依赖**和**源码**两类，提升开发服务启动时间

  > **依赖：**在开发时不会变动的纯JavaScript， Vite 会使用**esbuild**（esbuild用go编写）预构建依赖
  >
  > **源码：** 通常包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS 或者 Vue/Svelte 组件），时常会被编辑，如JSX、CSS 或者 Vue SFC 等。同时，并不是所有的源码都需要同时被加载（例如基于路由拆分的代码模块）。

* Vite 以原生 ESM 方式提供源码，让浏览器接管打包工作

### 1.vite如何保持快速更新

Vite 利用 HTTP 头来加速整个页面的重新加载（再次让浏览器为我们做更多事情）：

* **源码模块**的请求会根据 `304 Not Modified` 进行**协商缓存**
* **依赖模块**请求则会通过 `Cache-Control: max-age=31536000,immutable` 进行**强缓存**，因此一旦被缓存它们将不需要再次请求。

### 2. `index.html`入口文件 与 项目根目录

> 在开发期间 Vite 是一个服务器，而 `index.html` 是该 Vite 项目的入口文件。Vite 也支持多个 `.html` 作入口点的 [多页面应用模式](https://cn.vitejs.dev/guide/build.html#multi-page-app)。

#### 指定替代根目录

`vite` 以当前工作目录（ `package.json`所有目录）作为根目录启动开发服务器。你也可以通过 `vite serve some/sub/dir` 来指定一个替代的根目录。注意 Vite 同时会解析项目根目录下的 [配置文件（即 **`vite.config.js`**）](https://cn.vitejs.dev/config/#configuring-vite)，因此如果根目录被改变了，你需要将配置文件 (`vite.config.js`) 移动到新的根目录下。

