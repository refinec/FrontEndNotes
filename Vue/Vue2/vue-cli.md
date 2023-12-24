# vue-cli

## CLI服务

在一个 Vue CLI 项目中，`@vue/cli-service` 安装了一个名为 `vue-cli-service` 的命令

使用默认 preset 的项目的 `package.json`：

```json
{
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  }
}
```

通过 npm 或 Yarn 调用这些 script：

```shell
npm run serve
# OR
yarn serve
# 新版npm 自带npx
npx vue-cli-service serve
```

### vue-cli-service serve

> 该命令会启动一个开发服务器 (基于 [webpack-dev-server](https://github.com/webpack/webpack-dev-server)) 并附带开箱即用的模块热重载 (Hot-Module-Replacement)。除了通过命令行参数，也可以使用 `vue.config.js` 里的 [devServer](https://cli.vuejs.org/zh/config/#devserver) 字段配置开发服务器

命令行参数 `[entry]` 将被指定为唯一入口，而非额外的追加入口。尝试使用 `[entry]` 覆盖 `config.pages` 中的 `entry` 将可能引发错误。

```text
用法：vue-cli-service serve [options] [entry]

选项：
  --open    在服务器启动时打开浏览器
  --copy    在服务器启动时将 URL 复制到剪切版
  --mode    指定环境模式 (默认值：development)
  --host    指定 host (默认值：0.0.0.0)
  --port    指定 port (默认值：8080)
  --https   使用 https (默认值：false)
```

### vue-cli-service build

> 该命令会在 `dist/` 目录产生一个可用于生产环境的包，带有 JS/CSS/HTML 的压缩，和为更好的缓存而做的自动的 vendor chunk splitting。它的 chunk manifest 会内联在 HTML 里。

```text
用法：vue-cli-service build [options] [entry|pattern]

选项：
  --mode        指定环境模式 (默认值：production)
  --dest        指定输出目录 (默认值：dist)
  --modern      面向现代浏览器带自动回退地构建应用
  --target      app | lib | wc | wc-async (默认值：app)
  --name        库或 Web Components 模式下的名字 (默认值：package.json 中的 "name" 字段或入口文件名)
  --no-clean    在构建项目之前不清除目标目录
  --report      生成 report.html 以帮助分析包内容
  --report-json 生成 report.json 以帮助分析包内容
  --watch       监听文件变化
```

- `--modern` 使用[现代模式](https://cli.vuejs.org/zh/guide/browser-compatibility.html#现代模式)构建应用，为现代浏览器交付原生支持的 ES2015 代码，并生成一个兼容老浏览器的包用来自动回退。
- `--target` 允许你将项目中的任何组件以一个库或 Web Components 组件的方式进行构建。更多细节请查阅[构建目标](https://cli.vuejs.org/zh/guide/build-targets.html)。
- `--report` 和 `--report-json` 会根据构建统计生成报告，它会帮助你分析包中包含的模块们的大小。

### vue-cli-service inspect

使用该命令审查一个 Vue CLI 项目的 webpack config。更多细节请查阅[审查 webpack config](https://cli.vuejs.org/zh/guide/webpack.html#审查项目的-webpack-config)。

```text
用法：vue-cli-service inspect [options] [...paths]

选项：
  --mode    指定环境模式 (默认值：development)
```

### 查看vue-cli-service所有的可用命令

有些 CLI 插件会向 `vue-cli-service` 注入额外的命令。例如 `@vue/cli-plugin-eslint` 会注入 `vue-cli-service lint` 命令。

* 运行该命令查看所有注入的命令：`npx vue-cli-service help`

* 学习每个命令可用的选项：`npx vue-cli-service help [command]`

### 缓存和并行处理

* `cache-loader` 会默认为 Vue/Babel/TypeScript 编译开启。文件会缓存在 `node_modules/.cache` 中,如果遇到了编译方面的问题，可先删掉缓存目录之后再试试看。

* `thread-loader` 会在多核 CPU 的机器上为 Babel/TypeScript 转译开启

## Git Hook

在安装之后，`@vue/cli-service` 也会安装 [yorkie](https://github.com/yyx990803/yorkie)(`yorkie` fork 自 [`husky`](https://github.com/typicode/husky) 并且与后者不兼容)，它会在 `package.json` 的 `gitHooks` 字段中方便地指定 Git hook：

```json
{
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
   "lint-staged": {
    "*.{js,vue}": [
      "vue-cli-service lint",
      "git add"
    ]
  }
}
```

## 浏览器兼容性

###  browserslist

`package.json` 文件里的 `browserslist` 字段 或`.browserslistrc` 文件指定了项目的目标浏览器的范围。这个值会被 [@babel/preset-env](https://new.babeljs.io/docs/en/next/babel-preset-env.html) 和 [Autoprefixer](https://github.com/postcss/autoprefixer) 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀。

查阅[browserslist](https://github.com/ai/browserslist)了解如何指定浏览器范围

### Polyfill

#### useBuiltIns: 'usage'

一个默认的 Vue CLI 项目会使用 [@vue/babel-preset-app](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app)，它通过 `@babel/preset-env` 和 `browserslist` 配置来决定项目需要的 polyfill。

默认情况下，它会把 [`useBuiltIns: 'usage'`](https://new.babeljs.io/docs/en/next/babel-preset-env.html#usebuiltins-usage) 传递给 `@babel/preset-env`，这样它会根据源代码中出现的语言特性自动检测需要的 polyfill。这确保了最终包里 polyfill 数量的最小化。然而，这也意味着**如果其中一个依赖需要特殊的 polyfill，默认情况下 Babel 无法将其检测出来。**

如果有依赖需要 polyfill，你有几种选择：

1. **如果该依赖基于一个目标环境不支持的 ES 版本撰写:** 将其添加到 `vue.config.js` 中的 [`transpileDependencies`](https://cli.vuejs.org/zh/config/#transpiledependencies) 选项。这会为该依赖同时开启语法转换和根据使用情况检测 polyfill。

2. **如果该依赖交付了 ES5 代码并显式地列出了需要的 polyfill:** 你可以使用 `@vue/babel-preset-app` 的 [polyfills](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app#polyfills) 选项预包含所需要的 polyfill。**注意 `es.promise` 将被默认包含，因为现在的库依赖 Promise 是非常普遍的。**

   ```js
   // babel.config.js
   module.exports = {
     presets: [
       ['@vue/app', {
         polyfills: [
           'es.promise',
           'es.symbol'
         ]
       }]
     ]
   }
   ```

   推荐以这种方式添加 polyfill 而不是在源代码中直接导入它们，因为如果这里列出的 polyfill 在 `browserslist` 的目标中不需要，则它会被自动排除。

3. **如果该依赖交付 ES5 代码，但使用了 ES6+ 特性且没有显式地列出需要的 polyfill (例如 Vuetify)：\**请使用 `useBuiltIns: 'entry'` 然后在入口文件添加 `import 'core-js/stable'; import 'regenerator-runtime/runtime';`。这会根据 `browserslist` 目标导入\**所有** polyfill，这样你就不用再担心依赖的 polyfill 问题了，但是因为包含了一些没有用到的 polyfill 所以最终的包大小可能会增加。

更多细节可查阅 [@babel/preset-env 文档](https://new.babeljs.io/docs/en/next/babel-preset-env.html#usebuiltins-usage)

#### 构建库或是 Web Component 时的 Polyfills

当使用 Vue CLI 来[构建一个库或是 Web Component](https://cli.vuejs.org/zh/guide/build-targets.html) 时，推荐给 `@vue/babel-preset-app` 传入 `useBuiltIns: false` 选项。这能够确保你的库或是组件不包含不必要的 polyfills。通常来说，打包 polyfills 应当是最终使用你的库的应用的责任

### 现代模式

有了 Babel 我们可以兼顾所有最新的 ES2015+ 语言特性，但也意味着我们需要交付转译和 polyfill 后的包以支持旧浏览器。这些转译后的包通常都比原生的 ES2015+ 代码会更冗长，运行更慢。现如今绝大多数现代浏览器都已经支持了原生的 ES2015，所以因为要支持更老的浏览器而为它们交付笨重的代码是一种浪费。

Vue CLI 提供了一个“现代模式”帮你解决这个问题。以如下命令为生产环境构建：

```bash
vue-cli-service build --modern
```

Vue CLI 会产生两个应用的版本：一个现代版的包，面向支持 [ES modules](https://jakearchibald.com/2017/es-modules-in-browsers/) 的现代浏览器，另一个旧版的包，面向不支持的旧浏览器。

最酷的是这里没有特殊的部署要求。其生成的 HTML 文件会自动使用 [Phillip Walton 精彩的博文](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)中讨论到的技术：

- 现代版的包会通过 `<script type="module">` 在被支持的浏览器中加载；它们还会使用 `<link rel="modulepreload">` 进行预加载。
- 旧版的包会通过 `<script nomodule>` 加载，并会被支持 ES modules 的浏览器忽略。
- 一个针对 Safari 10 中 `<script nomodule>` 的修复会被自动注入。

对于一个 Hello World 应用来说，现代版的包已经小了 16%。在生产环境下，现代版的包通常都会表现出显著的解析速度和运算速度，从而改善应用的加载性能。

```
<script type="module"> 需要配合始终开启的 CORS 进行加载。这意味着你的服务器必须返回诸如 Access-Control-Allow-Origin: * 的有效的 CORS 头。如果你想要通过认证来获取脚本，可使将 crossorigin 选项设置为 use-credentials。
同时，现代浏览器使用一段内联脚本来避免 Safari 10 重复加载脚本包，所以如果你在使用一套严格的 CSP，你需要这样显性地允许内联脚本：
```


```text
Content-Security-Policy: script-src 'self' 'sha256-4RS22DYeB7U14dra4KcQYxmwt5HkOInieXK1NUMBmQI='
```

## HTML

`public/index.html` 文件是一个会被 [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin) 处理的模板。在构建过程中，资源链接会被自动注入。另外，Vue CLI 也会自动注入 resource hint (`preload/prefetch`、manifest 和图标链接 (当用到 PWA 插件时) 以及构建过程中处理的 JavaScript 和 CSS 文件的资源链接。

因为 index 文件被用作模板，所以你可以使用 [lodash template](https://lodash.com/docs/4.17.10#template) 语法插入内容：

- `<%= VALUE %>` 用来做不转义插值；
- `<%- VALUE %>` 用来做 HTML 转义插值；
- `<% expression %>` 用来描述 JavaScript 流程控制。

除了[被 `html-webpack-plugin` 暴露的默认值](https://github.com/jantimon/html-webpack-plugin#writing-your-own-templates)之外，所有[客户端环境变量](https://cli.vuejs.org/zh/guide/mode-and-env.html#using-env-variables-in-client-side-code)也可以直接使用。例如，`BASE_URL` 的用法：

```html
<link rel="icon" href="<%= BASE_URL %>favicon.ico">
```

### Preload

`<link rel="preload">`是一种 resource hint，用来**指定页面加载后很快会被用到的资源，所以在页面加载的过程中，我们希望在浏览器开始主体渲染之前尽早 preload**。

默认情况下，一个 Vue CLI 应用会为所有初始化渲染需要的文件自动生成 preload 提示。

这些提示会被 [@vue/preload-webpack-plugin](https://github.com/vuejs/preload-webpack-plugin) 注入，并且可以通过 `chainWebpack` 的 `config.plugin('preload')` 进行修改和删除。

###  Prefetch

`<link rel="prefetch">`是一种 resource hint，用来告诉浏览器**在页面加载完成后，利用空闲时间提前获取用户未来可能会访问的内容**。

默认情况下，一个 Vue CLI 应用会为所有作为 async chunk 生成的 JavaScript 文件 ([通过动态 `import()` 按需 code splitting](https://webpack.js.org/guides/code-splitting/#dynamic-imports) 的产物) 自动生成 prefetch 提示。

这些提示会被 [@vue/preload-webpack-plugin](https://github.com/vuejs/preload-webpack-plugin) 注入，并且可以通过 `chainWebpack` 的 `config.plugin('prefetch')` 进行修改和删除。

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // 移除 prefetch 插件
    config.plugins.delete('prefetch')

    // 或者
    // 修改它的选项：
    config.plugin('prefetch').tap(options => {
      options[0].fileBlacklist = options[0].fileBlacklist || []
      options[0].fileBlacklist.push(/myasyncRoute(.)+?\.js$/)
      return options
    })
  }
}
```

当 prefetch 插件被禁用时，可以通过 webpack 的内联注释手动选定要提前获取的代码区块：

```js
import(/* webpackPrefetch: true */ './someAsyncComponent.vue')
```

**webpack 的运行时会在父级区块被加载之后注入 prefetch 链接。 **

**注意：** Prefetch 链接将会消耗带宽。如果你的应用很大且有很多 async chunk，而用户主要使用的是对带宽较敏感的移动端，那么你可能需要关掉 prefetch 链接并手动选择要提前获取的代码区块。

### 不生成 index

当基于已有的后端使用 Vue CLI 时，你可能不需要生成 `index.html`，这样生成的资源可以用于一个服务端渲染的页面。这时可以向 [`vue.config.js`](https://cli.vuejs.org/zh/config/#vue-config-js) 加入下列代码：

```js
// vue.config.js
module.exports = {
  // 去掉文件名中的 hash
  filenameHashing: false,
  // 删除 HTML 相关的 webpack 插件
  chainWebpack: config => {
    config.plugins.delete('html')
    config.plugins.delete('preload')
    config.plugins.delete('prefetch')
  }
}
```

然而这样做并不是很推荐，因为：

- 硬编码的文件名不利于实现高效率的缓存控制。
- 硬编码的文件名也无法很好的进行 code-splitting (代码分段)，因为无法用变化的文件名生成额外的 JavaScript 文件。
- 硬编码的文件名无法在[现代模式](https://cli.vuejs.org/zh/guide/browser-compatibility.html#现代模式)下工作。

考虑换用 [indexPath](https://cli.vuejs.org/zh/config/#indexpath) 选项将生成的 HTML 用作一个服务端框架的视图模板。

## 静态资源文件

静态资源可以通过两种方式进行处理：

- 在 JavaScript 被导入或在 template/CSS 中通过**相对路径被引用。这类引用会被 webpack 处理**。
- **放置在 `public` 目录下**或通过**绝对路径被引用**。这类资源将会直接被拷贝，而不会经过 webpack 的处理。

### 从相对路径导入

在 JavaScript、CSS 或 `*.vue` 文件中使用相对路径 (必须以 `.` 开头) 引用一个静态资源时，该资源将会被包含进入 webpack 的依赖图中。在其编译过程中，所有诸如 `<img src="...">`、`background: url(...)` 和 CSS `@import` 的资源 URL **都会被解析为一个模块依赖**。

例如，`url(./image.png)` 会被翻译为 `require('./image.png')`，而：

```html
<img src="./image.png">
```

将会被编译到：

```js
h('img', { attrs: { src: require('./image.png') }})
```

在其内部，我们通过 `file-loader` 用版本哈希值和正确的公共基础路径来决定最终的文件路径，再用 `url-loader` 将小于 4kb 的资源内联，以减少 HTTP 请求的数量。

你可以通过 [chainWebpack](https://cli.vuejs.org/zh/config/#chainwebpack) 调整内联文件的大小限制。例如，下列代码会将其限制设置为 10kb：

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('images')
        .use('url-loader')
          .loader('url-loader')
          .tap(options => Object.assign(options, { limit: 10240 }))
  }
}
```

### URL 转换规则

- 如果 URL 是一个绝对路径 (例如 `/images/foo.png`)，它将会被保留不变。

- 如果 URL 以 **`.`** 开头，它会作为一个相对模块请求被解释且基于你的文件系统中的目录结构进行解析。

- 如果 URL 以 **`~`** 开头，**其后的任何内容都会作为一个模块请求被解析。这意味着你甚至可以引用 Node 模块中的资源：**

  ```html
  <img src="~some-npm-package/foo.png">
  ```

- 如果 URL 以 **`@`** 开头，它也会作为一个模块请求被解析。它的用处在于 Vue CLI 默认会设置一个指向 `<projectRoot>/src` 的别名 `@`。**(仅作用于模版中)**

### public 文件夹

任何放置在 `public` 文件夹的静态资源都会被简单的复制，而不经过 webpack。需要通过绝对路径来引用它们。

推荐将资源作为你的模块依赖图的一部分导入，这样它们会通过 webpack 的处理并获得如下好处：

- **脚本和样式表会被压缩且打包在一起，从而避免额外的网络请求。**
- **文件丢失会直接在编译时报错，而不是到了用户端才产生 404 错误。**
- **最终生成的文件名包含了内容哈希，因此你不必担心浏览器会缓存它们的老版本。**

- 在 `public/index.html` 或其它通过 `html-webpack-plugin` 用作模板的 HTML 文件中，你需要通过 `<%= BASE_URL %>` 设置链接前缀：

  ```html
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  ```

- 在模板中，你首先需要向你的组件传入基础 URL：

  ```js
  data () {
    return {
      publicPath: process.env.BASE_URL
    }
  }
  ```

  然后：

  ```html
  <img :src="`${publicPath}my-image.png`">
  ```

### 何时使用 `public` 文件夹

- 当需要在构建输出中指定一个文件的名字。
- 有上千个图片，需要动态引用它们的路径。
- 有些库可能和 webpack 不兼容，这时除了将其用一个独立的 `<script>` 标签引入没有别的选择。

## CSS 相关

所有编译后的 CSS 都会通过 [css-loader](https://github.com/webpack-contrib/css-loader) 来解析其中的 `url()` 引用，并将这些引用作为模块请求来处理。因此可以根据本地的文件结构用相对路径来引用静态资源。

**注意：** **如果要引用一个 npm 依赖中的文件，或是想要用 webpack alias，则需要在路径前加上 `~` 的前缀来避免歧义**。

### 预处理器

```bash
# Sass
npm install -D sass-loader sass

# Less
npm install -D less-loader less

# Stylus
npm install -D stylus-loader stylus
```

然后你就可以导入相应的文件类型，或在 `*.vue` 文件中这样来使用：

```vue
<style lang="scss">
$color: red;
</style>
```

### 自动化导入文件

自动化导入文件 (用于颜色、变量、mixin……)，你可以使用 [style-resources-loader](https://github.com/yenshih/style-resources-loader)。下面关于 Stylus 的在每个单文件组件和 Stylus 文件中导入 `./src/styles/imports.styl` 的例子(也可以选择使用 [vue-cli-plugin-style-resources-loader](https://www.npmjs.com/package/vue-cli-plugin-style-resources-loader))：

```js
// vue.config.js
const path = require('path')

module.exports = {
  chainWebpack: config => {
    const types = ['vue-modules', 'vue', 'normal-modules', 'normal']
    types.forEach(type => addStyleResource(config.module.rule('stylus').oneOf(type)))
  },
}

function addStyleResource (rule) {
  rule.use('style-resource')
    .loader('style-resources-loader')
    .options({
      patterns: [
        path.resolve(__dirname, './src/styles/imports.styl'),
      ],
    })
}
```

### PostCSS

Vue CLI 内部使用了 PostCSS。vue默认开启了 [autoprefixer](https://github.com/postcss/autoprefixer)。如果要配置目标浏览器，可使用 `package.json` 的 [browserslist](https://cli.vuejs.org/zh/guide/browser-compatibility.html#browserslist) 字段

* 通过 `.postcssrc` 或任何 [postcss-load-config](https://github.com/michael-ciniawsky/postcss-load-config) 支持的配置源来配置 PostCSS。
* 通过 `vue.config.js` 中的 `css.loaderOptions.postcss` 配置 [postcss-loader](https://github.com/postcss/postcss-loader)

**注意：**  在生产环境构建中，Vue CLI 会优化 CSS 并基于目标浏览器抛弃不必要的浏览器前缀规则。因为默认开启了 `autoprefixer`，所以只需使用无前缀的 CSS 规则即可。

### CSS Modules

通过 `<style module>` 以开箱即用的方式[在 `*.vue` 文件中使用 CSS Modules](https://vue-loader.vuejs.org/zh/guide/css-modules.html)。

在 JavaScript 中作为 CSS Modules 导入 CSS 或其它预处理文件，该文件应该以 `.module.(css|less|sass|scss|styl)` 结尾：

```js
import styles from './foo.module.css'
// 所有支持的预处理器都一样工作
import sassStyles from './foo.module.scss'
```

如果你想去掉文件名中的 `.module`，可以设置 `vue.config.js` 中的 `css.requireModuleExtension` 为 `false`：

```js
// vue.config.js
module.exports = {
  css: {
    requireModuleExtension: false
  }
}
```

希望自定义生成的 CSS Modules 模块的类名，可以通过 `vue.config.js` 中的 `css.loaderOptions.css` 选项来实现。所有的 `css-loader` 选项在这里都是支持的，例如 `localIdentName` 和 `camelCase`：

```js
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      css: {
        // 注意：以下配置在 Vue CLI v4 与 v3 之间存在差异。
        // Vue CLI v3 用户可参考 css-loader v1 文档
        // https://github.com/webpack-contrib/css-loader/tree/v1.0.1
        modules: {
          localIdentName: '[name]-[hash]'
        },
        localsConvention: 'camelCaseOnly'
      }
    }
  }
}
```

### 向预处理器 Loader 传递选项

使用 `vue.config.js` 中的 `css.loaderOptions` 选项。比如你可以这样向所有 Sass/Less 样式传入共享的全局变量(这样做比使用 `chainWebpack` 手动指定 loader 更推荐，因为这些选项需要应用在使用了相应 loader 的多个地方)：

```js
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      // 给 sass-loader 传递选项
      sass: {
        // @/ 是 src/ 的别名
        // 所以这里假设你有 `src/variables.sass` 这个文件
        // 注意：在 sass-loader v8 中，这个选项名是 "prependData"
        additionalData: `@import "~@/variables.sass"`
      },
      // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
      // 因为 `scss` 语法在内部也是由 sass-loader 处理的
      // 但是在配置 `prependData` 选项的时候
      // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
      // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
      scss: {
        additionalData: `@import "~@/variables.scss";`
      },
      // 给 less-loader 传递 Less.js 相关选项
      less:{
        // http://lesscss.org/usage/#less-options-strict-units `Global Variables`
        // `primary` is global variables fields name
        globalVars: {
          primary: '#fff'
        }
      }
    }
  }
}
```

Loader 可以通过 `loaderOptions` 配置，包括：

- [css-loader](https://github.com/webpack-contrib/css-loader)
- [postcss-loader](https://github.com/postcss/postcss-loader)
- [sass-loader](https://github.com/webpack-contrib/sass-loader)
- [less-loader](https://github.com/webpack-contrib/less-loader)
- [stylus-loader](https://github.com/shama/stylus-loader)

## webpack相关

### 审查项目的webpack配置

`@vue/cli-service` 对 webpack 配置进行了抽象，所以理解配置中包含的东西会比较困难，尤其是当你打算自行对其调整的时候。所以`vue-cli-service` 暴露了 `inspect` 命令用于审查解析好的 webpack 配置。全局的 `vue` 可执行程序同样提供了 `inspect` 命令，但这个命令只是简单的把 `vue-cli-service inspect` 代理到了项目中。

该命令会将解析出来的 webpack 配置、包括链式访问规则和插件的提示打印到 stdout。

* 可以将其输出重定向到一个文件以便进行查阅，但它输出的并不是一个有效的 webpack 配置文件，而是一个用于审查的被序列化的格式：

  ```sh
  vue inspect > output.js
  ```

* 通过指定一个路径来审查配置的一小部分：

  ```sh
  # 只审查第一条规则
  vue inspect module.rules.0
  ```

* 或者指向一个规则或插件的名字：

  ```sh
  vue inspect --rule vue
  vue inspect --plugin html
  ```

* 列出所有规则和插件的名字：

  ```sh
  vue inspect --rules
  vue inspect --plugins
  ```

### 以一个文件的方式使用解析好的配置

有些外部工具可能需要通过一个文件访问解析好的 webpack 配置，比如那些需要提供 webpack 配置路径的 IDE 或 CLI。在这种情况下你可以使用如下路径：

```text
<projectRoot>/node_modules/@vue/cli-service/webpack.config.js
```

该文件会动态解析并输出 `vue-cli-service` 命令中使用的相同的 webpack 配置，包括那些来自插件甚至是你自定义的配置。

## 模式和环境变量

### 模式

在**package.json**里的**scripts配置项**中添加**--mode xxx**来选择不同环境

一个 Vue CLI 项目有三个模式：

- `development` 模式用于 `vue-cli-service serve`
- `test` 模式用于 `vue-cli-service test:unit`
- `production` 模式用于 `vue-cli-service build` 和 `vue-cli-service test:e2e`

可以通过传递 `--mode` 选项参数为命令行覆写默认的模式。例如在构建命令中使用开发环境变量：

```text
vue-cli-service build --mode development
```

**注意：** 如果在环境中有默认的 `NODE_ENV`，应该移除它或在运行 `vue-cli-service` 命令的时候明确地设置 `NODE_ENV`。

### 环境变量

可以在项目根目录中放置下列文件来指定环境变量：

```bash
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略
```

而一个环境文件只包含环境变量的“键=值”对：

```text
FOO=bar
VUE_APP_NOT_SECRET_CODE=some_value
```

**注意：** 不要在应用程序中存储任何机密信息（例如私有 API 密钥），环境变量会随着构建打包嵌入到输出代码

**注意：** <u>只有 **`NODE_ENV`**，**`BASE_URL`** 和以 **`VUE_APP_`** 开头</u>的变量将通过 `webpack.DefinePlugin` 静态地嵌入到*客户端侧*的代码中，代码中可以通过**process.env.VUE_APP_BASE_API**访问。这是为了避免意外公开机器上可能具有相同名称的私钥。

要了解解析环境文件规则的细节，参考 [dotenv](https://github.com/motdotla/dotenv#rules)。也可使用 [dotenv-expand](https://github.com/motdotla/dotenv-expand) 来实现变量扩展 (Vue CLI 3.5+ 支持)。例如：

被载入的变量将会对 `vue-cli-service` 的所有命令、插件和依赖可用。

```bash
FOO=foo
BAR=bar

CONCAT=$FOO$BAR # CONCAT=foobar
```

* **.env serve **默认的环境变量

  ```javascript
  NODE_ENV = "development"
  BASE_URL = "./"
  VUE_APP_PUBLIC_PATH = "./"
  VUE_APP_API = "https://test.staven630.com/api"
  ```

* **.env.production build**默认的环境变量

  ```javascript
  NODE_ENV = "production"
  BASE_URL = "https://prod.staven630.com/"
  VUE_APP_PUBLIC_PATH = "https://prod.oss.com/staven-blog"
  VUE_APP_API = "https://prod.staven630.com/api"
  
  ACCESS_KEY_ID = "xxxxxxxxxxxxx"
  ACCESS_KEY_SECRET = "xxxxxxxxxxxxx"
  REGION = "oss-cn-hangzhou"
  BUCKET = "staven-prod"
  PREFIX = "staven-blog"
  ```

* **.env.crm** 用于自定义 build 环境配置（预发服务器）

  ```javascript
  NODE_ENV = "production"
  BASE_URL = "https://crm.staven630.com/"
  VUE_APP_PUBLIC_PATH = "https://crm.oss.com/staven-blog"
  VUE_APP_API = "https://crm.staven630.com/api"
  
  ACCESS_KEY_ID = "xxxxxxxxxxxxx"
  ACCESS_KEY_SECRET = "xxxxxxxxxxxxx"
  REGION = "oss-cn-hangzhou"
  BUCKET = "staven-crm"
  PREFIX = "staven-blog"
  
  IS_ANALYZE = true;
  ```

```json
// package.json

"scripts": {
  "serve": "vue-cli-service serve",
  "build": "vue-cli-service build",
  "crm": "vue-cli-service build --mode crm",
  "lint": "vue-cli-service lint"
}
```

**使用环境变量**

```vue
<template>
  <div class="home">
    <!-- template中使用环境变量 -->
     API: {{ api }}
  </div>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      api: process.env.VUE_APP_API
    };
  },
  mounted() {
    // js代码中使用环境变量
    console.log("BASE_URL: ", process.env.BASE_URL);
    console.log("VUE_APP_API: ", process.env.VUE_APP_API);
  }
};
</script>
```

### 环境文件加载优先级

* 为一个特定模式准备的环境文件 (如 **`.env.production`**) 将会比一般的环境文件 (例如 **`.env`**) 拥有更高的优先级。

* 此外，Vue CLI 启动时已经存在的环境变量拥有最高优先级，并不会被 `.env` 文件覆写。
* `.env` 环境文件是通过运行 `vue-cli-service` 命令载入的，因此环境文件发生变化，需要重启服务。

### 示例：Staging 模式

假设一个应用包含以下 `.env` 文件：

```text
VUE_APP_TITLE=My App
```

和 `.env.staging` 文件：

```text
NODE_ENV=production
VUE_APP_TITLE=My App (staging)
```

- `vue-cli-service build` 会加载可能存在的 `.env`、`.env.production` 和 `.env.production.local` 文件然后构建出生产环境应用。
- `vue-cli-service build --mode staging` 会在 staging 模式下加载可能存在的 `.env`、`.env.staging` 和 `.env.staging.local` 文件然后构建出生产环境应用。

这两种情况下，根据 `NODE_ENV`，构建出的应用都是生产环境应用，但是在 staging 版本中，`process.env.VUE_APP_TITLE` 被覆写成了另一个值。

### 在客户端侧代码中使用环境变量

* 只有以 `VUE_APP_` 开头的变量会被 `webpack.DefinePlugin` 静态嵌入到客户端侧的包中。在构建过程中，`process.env.VUE_APP_SECRET` 将会被相应的值所取代。在 `VUE_APP_SECRET=secret` 的情况下，它会被替换为 `"secret"`。所以可以在应用的代码中这样访问它们：

  ```javascript
  console.log(process.env.VUE_APP_SECRET)
  ```

* 除了 **`VUE_APP_*`** 变量之外，应用代码中始终可用的还有两个特殊的变量：
  * **`NODE_ENV`** - 会是 `"development"`、`"production"` 或 `"test"` 中的一个。具体的值取决于应用运行的[模式](https://cli.vuejs.org/zh/guide/mode-and-env.html#模式)。
  * **`BASE_URL`** - 会和 `vue.config.js` 中的 `publicPath` 选项相符，即你的应用会部署到的基础路径。

  所有解析出来的环境变量都可以在 `public/index.html` 中以 [HTML 插值](https://cli.vuejs.org/zh/guide/html-and-static-assets.html#插值)中介绍的方式使用。

* 可以在 `vue.config.js` 文件中计算环境变量，它们仍然需要以 `VUE_APP_` 前缀开头。

  ```js
  process.env.VUE_APP_VERSION = require('./package.json').version
  
  module.exports = {
    // config
  }
  ```

### 只在本地有效的变量

有一些不应该提交到代码仓库中的变量，尤其是项目托管在公共仓库时。这种情况下应该使用一个 **`.env.local`** 文件取而代之。**本地环境文件默认会被忽略，且出现在 `.gitignore` 中**。

**`.local` 也可以加在指定模式的环境文件上，比如 `.env.development.local` 将会在 development 模式下被载入，且被 git 忽略。**

## 部署

### 本地预览

`dist` 目录需要启动一个 HTTP 服务器来访问 (除非你已经将 `publicPath` 配置为了一个相对的值)，所以以 `file://` 协议直接打开 `dist/index.html` 是不会工作的。在本地预览生产环境构建最简单的方式就是使用一个 Node.js 静态文件服务器，例如 [serve](https://github.com/zeit/serve)：

```bash
npm install -g serve
# -s 参数的意思是将其架设在 Single-Page Application 模式下
# 这个模式会处理即将提到的路由问题
serve -s dist
```

### 使用 `history.pushState` 的路由

在 `history` 模式下使用 Vue Router，是无法搭配简单的静态文件服务器的。例如，如果你使用 Vue Router 为 `/todos/42/` 定义了一个路由，开发服务器已经配置了相应的 `localhost:3000/todos/42` 响应，但是一个为生产环境构建架设的简单的静态服务器会却会返回 404。

为了解决这个问题，需要配置生产环境服务器，将任何没有匹配到静态文件的请求回退到 `index.html`。Vue Router 的文档提供了[常用服务器配置指引](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。

### CORS

如果前端静态内容是部署在与后端 API 不同的域名上，需要适当地配置 [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)。

### PWA

如果使用了 PWA 插件，那么应用必须架设在 HTTPS 上，这样 [Service Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API) 才能被正确注册。

## 单元测试

### Jest

更多细节可查阅 [@vue/cli-plugin-unit-jest](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-jest)。

### Mocha (配合 `mocha-webpack`)

更多细节可查阅 [@vue/cli-plugin-unit-mocha](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-mocha)。

## E2E 测试

### Cypress

更多细节可查阅 [@vue/cli-plugin-e2e-cypress](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-cypress)。

### Nightwatch

更多细节可查阅 [@vue/cli-plugin-e2e-nightwatch](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-nightwatch)。

## Babel

Babel 可以通过 `babel.config.js` 进行配置。

> Vue CLI 使用了 Babel 7 中的新配置格式 `babel.config.js`。和 `.babelrc` 或 `package.json` 中的 `babel` 字段不同，这个配置文件不会使用基于文件位置的方案，而是会一致地运用到项目根目录以下的所有文件，包括 `node_modules` 内部的依赖。我们推荐在 Vue CLI 项目中始终使用 `babel.config.js` 取代其它格式。

所有的 Vue CLI 应用都使用 `@vue/babel-preset-app`，它包含了 `babel-preset-env`、JSX 支持以及为最小化包体积优化过的配置。通过[它的文档](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app)可以查阅到更多细节和 preset 选项。查阅 [Polyfill](https://cli.vuejs.org/zh/guide/browser-compatibility.html#polyfill) 

## ESLint

ESLint 可以通过 `.eslintrc` 或 `package.json` 中的 `eslintConfig` 字段来配置。

更多细节查阅 [@vue/cli-plugin-eslint](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint)

## TypeScript

TypeScript 可以通过 `tsconfig.json` 来配置。

更多细节可查阅 [@vue/cli-plugin-typescript](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-typescript)

