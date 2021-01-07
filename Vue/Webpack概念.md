# Webpack概念

## 简单配置

```js
# webpack.config.js 
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  # 模式
  mode: "production", // "production" | "development" | "none"  
    mode: "production", // enable many optimizations for production builds
  	mode: "development", // enabled useful tools for development
  	mode: "none", // no defaults
  	// Chosen mode tells webpack to use its built-in optimizations accordingly.
  # 入口
  // 单页应用(SPA)：一个入口起点，多页应用(MPA)：多个入口起点。
  entry: "./app/entry", // string | object | array  
    entry: ["./app/entry1", "./app/entry2"],
  	entry: {
    	a: "./app/entry-a",
    	b: ["./app/entry-b1", "./app/entry-b2"]
  	},
      // 默认为 './src'
      // 这里应用程序开始执行
      // webpack 开始打包
  # 出口
  output: { // webpack 如何输出结果的相关选项
    path: path.resolve(__dirname, "dist"), // string
    // 所有输出文件的目标路径
    // 必须是绝对路径（使用 Node.js 的 path 模块）
    filename: "bundle.js", // string    
        filename: "[name].js", // 用于多个入口点(entry point)（出口点？）
    	filename: "[chunkhash].js", // 用于长效缓存
    	// 「入口分块(entry chunk)」的文件名模板
    publicPath: "/assets/", // string    
        publicPath: "",
    	publicPath: "https://cdn.example.com/",
    	// 输出解析文件的目录，url 相对于 HTML 页面
    library: "MyLibrary", // string,
    // 导出库(exported library)的名称
    libraryTarget: "umd", // 通用模块定义        
        libraryTarget: "umd2", // 通用模块定义
        libraryTarget: "commonjs2", // exported with module.exports
        libraryTarget: "commonjs", // 作为 exports 的属性导出
        libraryTarget: "amd", // 使用 AMD 定义方法来定义
        libraryTarget: "this", // 在 this 上设置属性
        libraryTarget: "var", // 变量定义于根作用域下
        libraryTarget: "assign", // 盲分配(blind assignment)
        libraryTarget: "window", // 在 window 对象上设置属性
        libraryTarget: "global", // property set to global object
        libraryTarget: "jsonp", // jsonp wrapper
    	// 导出库(exported library)的类型
    /* 高级输出配置（点击显示） */    
    pathinfo: true, // boolean
    // 在生成代码时，引入相关的模块、导出、请求等有帮助的路径信息。
    chunkFilename: "[id].js",
    chunkFilename: "[chunkhash].js", // 长效缓存(/guides/caching)
    // 「附加分块(additional chunk)」的文件名模板
    jsonpFunction: "myWebpackJsonp", // string
    // 用于加载分块的 JSONP 函数名
    sourceMapFilename: "[file].map", // string
    sourceMapFilename: "sourcemaps/[file].map", // string
    // 「source map 位置」的文件名模板
    devtoolModuleFilenameTemplate: "webpack:///[resource-path]", // string
    // 「devtool 中模块」的文件名模板
    devtoolFallbackModuleFilenameTemplate: "webpack:///[resource-path]?[hash]", // string
    // 「devtool 中模块」的文件名模板（用于冲突）
    umdNamedDefine: true, // boolean
    // 在 UMD 库中使用命名的 AMD 模块
    crossOriginLoading: "use-credentials", // 枚举
    crossOriginLoading: "anonymous",
    crossOriginLoading: false,
    // 指定运行时如何发出跨域请求问题
    /* 专家级输出配置（自行承担风险） */  
  },
  module: {
    # 添加loader
    // 关于模块配置
    rules: [
      // 模块规则（配置 loader、解析器等选项）
      {
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],
        // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
        // test 和 include 具有相同的作用，都是必须匹配选项
        // exclude 是必不匹配选项（优先于 test 和 include）
        // 最佳实践：
        // - 只在 test 和 文件名匹配 中使用正则表达式
        // - 在 include 和 exclude 中使用绝对路径数组
        // - 尽量避免 exclude，更倾向于使用 include
        issuer: { test, include, exclude },
        // issuer 条件（导入源）
        enforce: "pre",
        enforce: "post",
        // 标识应用这些规则，即使规则覆盖（高级选项）
        loader: "babel-loader",
        // 应该应用的 loader，它相对上下文解析
        // 为了更清晰，`-loader` 后缀在 webpack 2 中不再是可选的
        // 查看 webpack 1 升级指南。
        options: {
          presets: ["es2015"]
        },
        // loader 的可选项
      },
	 { test: /\.txt$/, use: 'raw-loader' },
      {   // 从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束
        test:/\.css$/,
            use:[
                { loader:'style-loader'},
                {
                    loader:'css-loader',
                    options:{
                        modules:true
                    }
                },
                {
                    loader:'sass-loader'
                }
            ]
      }
      {
        test: /\.html$/,
        use: [
          // 应用多个 loader 和选项
          "htmllint-loader",
          {
            loader: "html-loader",
            options: {
              /* ... */
            }
          }
        ]
      },
      { oneOf: [ /* rules */ ] },
      // 只使用这些嵌套规则之一
      { rules: [ /* rules */ ] },
      // 使用所有这些嵌套规则（合并可用条件）
      { resource: { and: [ /* 条件 */ ] } },
      // 仅当所有条件都匹配时才匹配
      { resource: { or: [ /* 条件 */ ] } },
      { resource: [ /* 条件 */ ] },
      // 任意条件匹配时匹配（默认为数组）
      { resource: { not: /* 条件 */ } }
      // 条件不匹配时匹配
    ],
    /* 高级模块配置（点击展示） */    
    noParse: [
      /special-library\.js$/
    ],
    // 不解析这里的模块
    unknownContextRequest: ".",
    unknownContextRecursive: true,
    unknownContextRegExp: /^\.\/.*$/,
    unknownContextCritical: true,
    exprContextRequest: ".",
    exprContextRegExp: /^\.\/.*$/,
    exprContextRecursive: true,
    exprContextCritical: true,
    wrappedContextRegExp: /.*/,
    wrappedContextRecursive: true,
    wrappedContextCritical: false,
    // specifies default behavior for dynamic requests
  },
  resolve: {
    // 解析模块请求的选项
    // （不适用于对 loader 解析）
    modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    // 用于查找模块的目录
    extensions: [".js", ".json", ".jsx", ".css"],
    // 使用的扩展名
    alias: {
      // 模块别名列表
      "module": "new-module",
      // 起别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"
      "only-module$": "new-module",
      // 起别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"
      "module": path.resolve(__dirname, "app/third/module.js"),
      // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
      // 模块别名相对于当前上下文导入
    },
    /* 可供选择的别名语法（点击展示） */    
    alias: [
      {
        name: "module1",
        // 旧的请求
        alias: "new-module",
        // 新的请求
        onlyModule: true
        // 如果为 true，只有 "module" 是别名
        // 如果为 false，"module/inner/path" 也是别名
      }
    ],
    /* 高级解析选项（点击展示） */    
    symlinks: true,
    // 遵循符号链接(symlinks)到新位置
    descriptionFiles: ["package.json"],
    // 从 package 描述中读取的文件
    mainFields: ["main"],
    // 从描述文件中读取的属性
    // 当请求文件夹时
    aliasFields: ["browser"],
    // 从描述文件中读取的属性
    // 以对此 package 的请求起别名
    enforceExtension: false,
    // 如果为 true，请求必不包括扩展名
    // 如果为 false，请求可以包括扩展名
    moduleExtensions: ["-module"],
    enforceModuleExtension: false,
    // 类似 extensions/enforceExtension，但是用模块名替换文件
    unsafeCache: true,
    unsafeCache: {},
    // 为解析的请求启用缓存
    // 这是不安全，因为文件夹结构可能会改动
    // 但是性能改善是很大的
    cachePredicate: (path, request) => true,
    // predicate function which selects requests for caching
    plugins: [
      // ...
    ]
    // 应用于解析器的附加插件
  },
  performance: {
    hints: "warning", // 枚举    
        hints: "error", // 性能提示中抛出错误
    	hints: false, // 关闭性能提示
    maxAssetSize: 200000, // 整数类型（以字节为单位）
    maxEntrypointSize: 400000, // 整数类型（以字节为单位）
    assetFilter: function(assetFilename) {
      // 提供资源文件名的断言函数
      return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
  },
  devtool: "source-map", // enum  
    devtool: "inline-source-map", // 嵌入到源文件中
  	devtool: "eval-source-map", // 将 SourceMap 嵌入到每个模块中
  	devtool: "hidden-source-map", // SourceMap 不在源文件中引用
  	devtool: "cheap-source-map", // 没有模块映射(module mappings)的 SourceMap 低级变体(cheap-variant)
  	devtool: "cheap-module-source-map", // 有模块映射(module mappings)的 SourceMap 低级变体
  	devtool: "eval", // 没有模块映射，而是命名模块。以牺牲细节达到最快。
  	// 通过在浏览器调试工具(browser devtools)中添加元信息(meta info)增强调试
  	// 牺牲了构建速度的 `source-map' 是最详细的。
  context: __dirname, // string（绝对路径！）
  // webpack 的主目录
  // entry 和 module.rules.loader 选项
  // 相对于此目录解析
  target: "web", // 枚举  
    target: "webworker", // WebWorker
  	target: "node", // node.js 通过 require
  	target: "async-node", // Node.js 通过 fs 和 vm
  	target: "node-webkit", // nw.js
  	target: "electron-main", // electron，主进程(main process)
  	target: "electron-renderer", // electron，渲染进程(renderer process)
  	target: (compiler) => { /* ... */ }, // 自定义
  // bundle 应该运行的环境
  // 更改 块加载行为(chunk loading behavior) 和 可用模块(available module)
  externals: ["react", /^@angular\//],  
    externals: "react", // string（精确匹配）
  	externals: /^[a-z\-]+($|\/)/, // 正则
  	externals: { // 对象
    	angular: "this angular", // this["angular"]
    	react: { // UMD
      		commonjs: "react",
      		commonjs2: "react",
      		amd: "react",
      		root: "React"
    	}
  	},
  	externals: (request) => { /* ... */ return "commonjs " + request }
  	// 不要遵循/打包这些模块，而是在运行时从环境中请求他们
  	serve: { //object
    	port: 1337,
    	content: './dist',
    	// ...
  	},
  	// 为 webpack-serve 提供选项
  stats: "errors-only",  
    stats: { //object
    	assets: true,
    	colors: true,
    	errors: true,
    	errorDetails: true,
    	hash: true,
    	// ...
  	},
  	// 精确控制要显示的 bundle 信息
  devServer: {
    proxy: { // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
    compress: true, // enable gzip compression
    historyApiFallback: true, // true for index.html upon 404, object for multiple paths
    hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
    https: false, // true for self-signed, object for cert authority
    noInfo: true, // only errors & warns on hot reload
    // ...
  },
  # 插件
  plugins: [
      new webpack.ProgressPlugin(),
      // html-webpack-plugin 为应用程序生成 HTML 一个文件，并自动注入所有生成的 bundle。
      new HtmlWebpackPlugin({template: './src/index.html'})
  ],
  // 附加插件列表
  /* 高级配置（点击展示） */  
  resolveLoader: { /* 等同于 resolve */ }
  // 独立解析选项的 loader
  parallelism: 1, // number
  // 限制并行处理模块的数量
  profile: true, // boolean
  // 捕获时机信息
  bail: true, //boolean
  // 在第一个错误出错时抛出，而不是无视错误。
  cache: false, // boolean
  // 禁用/启用缓存
  watch: true, // boolean
  // 启用观察
  watchOptions: {
    aggregateTimeout: 1000, // in ms
    // 将多个更改聚合到单个重构建(rebuild)
    poll: true,
    poll: 500, // 间隔单位 ms
    // 启用轮询观察模式
    // 必须用在不通知更改的文件系统中
    // 即 nfs shares（译者注：Network FileSystem，最大的功能就是可以透過網路，讓不同的機器、不同的作業系統、可以彼此分享個別的檔案 ( share file )）
  },
  node: {
    // Polyfills and mocks to run Node.js-
    // environment code in non-Node environments.
    console: false, // boolean | "mock"
    global: true, // boolean | "mock"
    process: true, // boolean
    __filename: "mock", // boolean | "mock"
    __dirname: "mock", // boolean | "mock"
    Buffer: true, // boolean | "mock"
    setImmediate: true // boolean | "mock" | "empty"
  },
  recordsPath: path.resolve(__dirname, "build/records.json"),
  recordsInputPath: path.resolve(__dirname, "build/records.json"),
  recordsOutputPath: path.resolve(__dirname, "build/records.json"),
  // TODO
}
```

### 使用自定义配置文件

出于某些原因要根据特定情况使用自定义配置文件，则可以通过命令行使用该`--config`标志来更改此配置文件。

```json
# package.json
"scripts": {
  "build": "webpack --config prod.config.js"
}
```

### 配置文件生成器（可使用）

生成定制Webpack配置：https://generatewebpackconfig.netlify.app/

用于创建Webpack配置的可视化工具：https://createapp.dev/

## 入口entry

项目的打包入口，默认值是 `./src/index.js`，可指定一个（或多个）不同的入口起点。

如果传入一个字符串或字符串数组，chunk 会被命名为 `main`。

如果传入一个对象，则每个键(key)会是 chunk 的名称，该值描述了 chunk 的入口起点。

### 单个入口写法

用法：`entry: string|Array<string>`

```javascript
module.exports = {
  entry: './path/to/my/entry/file.js'
  # 或者
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

### 对象写法(多入口)

用法：`entry: {[entryChunkName: string]: string|Array<string>}`

```javascript
module.exports = {
  entry: {
    # 多页面应用程序，告诉 webpack 需要三个独立分离的依赖图
    app: './src/app.js',
    adminApp: './src/adminApp.js'
  }
};
```

在多页面应用程序中，服务器会传输一个新的 HTML 文档给你的客户端。页面重新加载此新文档，并且资源被重新下载。然而，这给了我们特殊的机会去做很多事：

- 使用 `optimization.splitChunks` 为页面间共享的应用程序代码创建 bundle。由于入口起点增多，多页应用能够复用入口起点之间的大量代码/模块，从而可以极大地从这些技术中受益。

**根据经验：每个 HTML 文档只使用一个入口起点。**

## 输出output

项目的打包文件的存放出口，默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist` 文件夹中。

如果配置创建了多个单独的 "chunk"（如使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用 [占位符(substitutions)](https://v4.webpack.docschina.org/configuration/output#output-filename) 来确保每个文件具有唯一的名称。

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
};

// 写入到硬盘：./dist/app.js, ./dist/search.js
```

**对资源使用 CDN 和 hash 的复杂示例：**

```javascript
module.exports = {
  //...
  output: {
    path: '/home/proj/cdn/assets/[hash]',
    publicPath: 'http://cdn.example.com/assets/[hash]/'
  }
};
```

在编译时，不知道最终输出文件的 `publicPath` 是什么地址，则可以将其留空，并且在运行时通过入口起点文件中的 `__webpack_public_path__` 动态设置。

```javascript
__webpack_public_path__ = myRuntimePublicPath;
// 应用程序入口的其余部分
```

## loader

>  webpack 只能理解 JavaScript 和 JSON 文件。**loader** 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 [模块](https://v4.webpack.docschina.org/concepts/modules)，以供应用程序使用，以及被添加到依赖图中。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript 或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 `import` CSS文件！

**注意：** *使用正则表达式匹配文件时，不要为它添加引号。也就是说，*`/\.txt$/` *与* `'/\.txt$/'`*/* `"/\.txt$/"` 不一样。前者指示 webpack 匹配任何以 .txt 结尾的文件，后者指示 webpack 匹配具有绝对路径 '.txt' 的单个文件。

### 使用loader（3种）

1. 配置（推荐）：在 **webpack.config.js** 文件中指定 loader。

   在`module.rules` webpack 配置中指定多个 loader。loader 从右到左地取值(evaluate)/执行(execute)。下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束。

   ```javascript
   module.exports = {
     module: {
       rules: [
         {
           test: /\.css$/,
           use: [
             { loader: 'style-loader' },
             {
               loader: 'css-loader',
               options: {
                 modules: true
               }
             },
             { loader: 'sass-loader' }
           ]
         }
       ]
     }
   };
   ```

2. 内联：在每个 `import` 语句中显式指定 loader。

   在 `import` 语句或任何 [等同于 "import" 的方法](https://v4.webpack.docschina.org/api/module-methods) 中指定 loader。

   使用 `!` 将资源中的 loader 分开。每个部分都会相对于当前目录解析。使用 `!` 为整个规则添加前缀，可以覆盖配置中的所有 loader 定义。选项可以传递查询参数，例如 `?key=value&foo=bar`，或者一个 JSON 对象，例如 `?{"key":"value","foo":"bar"}`。

   ```js
   import Styles from 'style-loader!css-loader?modules!./styles.css';
   ```

3. CLI：在 shell 命令中指定它们。

   下面实例对 `.jade` 文件使用 `jade-loader`，以及对 `.css` 文件使用 [`style-loader`](https://v4.webpack.docschina.org/loaders/style-loader) 和 [`css-loader`](https://v4.webpack.docschina.org/loaders/css-loader)。

   ```sh
   webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
   ```

## 插件plugin

>  loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务,在于解决 loader 无法实现的其他事。包括：打包优化，资源管理，注入环境变量。

### 剖析

webpack **插件**是一个具有 [`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法的 JavaScript 对象。`apply` 方法会被 webpack compiler 调用，并且 compiler 对象可在**整个**编译生命周期访问。

```javascript
# ConsoleLogOnBuildWebpackPlugin.js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';
class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
      // compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中复用。
    compiler.hooks.run.tap(pluginName, compilation => {
      console.log('webpack 构建过程开始！');
    });
  }
}
```

### 用法

**webpack 提供许多开箱可用的插件 https://v4.webpack.docschina.org/plugins **,使用一个插件，只需要 `require()` 它，然后把它添加到 `plugins` 数组中。

多数插件可以通过选项(option)自定义，**插件**可以携带参数/选项。也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要向 `plugins` 属性传入 `new` 实例。

### Node API

在使用 Node API 时，还可以通过配置中的 `plugins` 属性传入插件。

```javascript
# some-node-script.js
const webpack = require('webpack'); //访问 webpack 运行时(runtime)
const configuration = require('./webpack.config.js');

let compiler = webpack(configuration);
new webpack.ProgressPlugin().apply(compiler);
compiler.run(function(err, stats) {
  // ...
});
```

## 模式mode

> 通过选择 `development`(默认), `production` 、 `none` 之中的一个，来设置 `mode` 参数，可以启用 webpack 内置在相应环境下的优化。

在配置对象中提供 `mode` 选项：

```javascript
module.exports = {
  mode: 'production'
};
```

或者从 [CLI](https://v4.webpack.docschina.org/api/cli/) 参数中传递：

```bash
webpack --mode=production
```

| 选项        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| development | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `development`。启用 `NamedChunksPlugin` 和 `NamedModulesPlugin`。 |
| production  | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `production`。启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `TerserPlugin`。 |
| none        | 退出任何默认优化选项                                         |

**注意：** *设置* `NODE_ENV` *并不会自动地设置* `mode`。

如果要根据 webpack.config.js 中的 **mode** 变量更改打包行为，则必须将配置导出为一个函数，而不是导出为一个对象：

```javascript
var config = {
  entry: './app.js'
  //...
};

module.exports = (env, argv) => {
  if (argv.mode === 'development') {
    config.devtool = 'source-map';
  }
  if (argv.mode === 'production') {
    //...
  }
  return config;
};
```

## 部署目标

> 因为服务器和浏览器代码都可以用 JavaScript 编写，所以 webpack 提供了多种部署 target(目标)

* 想要设置 `target` 属性，只需要在webpack 配置中设置 target 的值。

```javascript
# webpack.config.js
module.exports = {
  // 使用 node，webpack 会编译为用于类 Node.js 环境（使用 Node.js 的 require ，而不是使用任意内置模块（如 fs 或 path）来加载 chunk）。
  target: 'node'
};
```

*  webpack **不支持**向 `target` 传入多个字符串，但可以通过打包两个单独配置，来创建出一个同构的 library：

```javascript
# webpack.config.js
const path = require('path');
const serverConfig = {
  target: 'node',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.node.js'
  }
  //…
};
const clientConfig = {
  target: 'web', // <=== 默认是 'web'，可省略
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.js'
  }
  //…
};
module.exports = [ serverConfig, clientConfig ]; // 在 dist 文件夹下创建 lib.js 和 lib.node.js 文件
```

### 示例资源

- **[compare-webpack-target-bundles](https://github.com/TheLarkInn/compare-webpack-target-bundles)**：有关「测试和查看」不同的 webpack *target* 的大量资源。也有大量 bug 报告。
- **[Boilerplate of Electron-React Application](https://github.com/chentsulin/electron-react-boilerplate)**：一个 electron 主进程和渲染进程构建过程的很好的例子。



## 浏览器兼容性browser compatibility

webpack 支持所有符合 [ES5 标准](https://kangax.github.io/compat-table/es5/) 的浏览器（不支持 IE8 及以下版本）。webpack 的 `import()` 和 `require.ensure()` 需要 `Promise`。如果想要支持旧版本浏览器，在使用这些表达式之前，还需要 [提前加载 polyfill](https://v4.webpack.docschina.org/guides/shimming/)。

