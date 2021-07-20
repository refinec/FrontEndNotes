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
    // 「source map 位置」的文件名模板，自定义生成 source map 的文件名
    devtoolModuleFilenameTemplate: "webpack:///[resource-path]", // string
    // 「devtool 中模块」的文件名模板， source map 中不包含源代码内容。浏览器通常会尝试从 web 服务器或文件系统加载源代码。确保正确设置以匹配源代码的 url。
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

   在`module.rules` webpack 配置中指定多个 loader。loader **从右到左**地取值(evaluate)/执行(execute)。下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束。

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

>  loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务,在于解决 loader 无法实现的其他事。包括：**打包优化，资源管理，注入环境变量**。

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
| production  | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 ` `。启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `TerserPlugin`。 |
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

## 开发环境

### 使用source map找到错误源代码

使用`devtool`选项，来如何生成 source map。另外使用 [`SourceMapDevToolPlugin`](https://v4.webpack.docschina.org/plugins/source-map-dev-tool-plugin) 进行更细粒度的配置，用 [`source-map-loader`](https://v4.webpack.docschina.org/loaders/source-map-loader) 来处理已有的 source map。

**注意：** *可以直接使用* `SourceMapDevToolPlugin`*/*`EvalSourceMapDevToolPlugin` *来替代使用* `devtool` *选项，* 但*切勿同时使用* `devtool` *选项*，`devtool`  *选项在内部添加过这些插件，所以会最终将应用两次插件。*

**devtool的选项：** 关于下面的选项*webpack 仓库中包含一个* [显示所有 `devtool` 变体效果的示例](https://github.com/webpack/webpack/tree/master/examples/source-map)*。*

| **devtool**                    | **构建速度** | **重新构建速度** | **生产环境** | **品质(quality)**      |
| ------------------------------ | ------------ | ---------------- | ------------ | ---------------------- |
| (none)                         | +++          | +++              | yes          | 打包后的代码           |
| eval                           | +++          | +++              | no           | 生成后的代码           |
| cheap-eval-source-map          | +            | ++               | no           | 转换过的代码（仅限行） |
| cheap-module-eval-source-map   | o            | ++               | no           | 原始源代码（仅限行）   |
| eval-source-map                | \--          | +                | no           | 原始源代码             |
| cheap-source-map               | +            | o                | yes          | 转换过的代码（仅限行） |
| cheap-module-source-map        | o            | -                | yes          | 原始源代码（仅限行）   |
| inline-cheap-source-map        | +            | o                | no           | 转换过的代码（仅限行） |
| inline-cheap-module-source-map | o            | -                | no           | 原始源代码（仅限行）   |
| source-map                     | \--          | \--              | yes          | 原始源代码             |
| inline-source-map              | \--          | \--              | no           | 原始源代码             |
| hidden-source-map              | \--          | \--              | yes          | 原始源代码             |
| nosources-source-map           | \--          | \--              | yes          | 无源代码内容           |

* **打包后的代码**

  将所有生成的代码视为一大块代码,看不到相互分离的模块

* **生成后的代码**

  每个模块相互分离，并用模块名称进行注释。

  可以看到 webpack 生成的代码。示例：会看到类似 `var module__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(42); module__WEBPACK_IMPORTED_MODULE_1__.a();`，而不是 `import {test} from "module"; test();`。

* **转换过的代码**

  每个模块相互分离，并用模块名称进行注释。

  可以看到 webpack 转换前、loader 转译后的代码。示例：你会看到类似 `import {test} from "module"; var A = function(_test) { ... }(test);`，而不是 `import {test} from "module"; class A extends test {}`。

* **原始源代码**

  每个模块相互分离，并用模块名称进行注释。

  会看到转译之前的代码，正如编写它时。取决于 loader 支持。

* **无源代码内容**

  source map 中不包含源代码内容。

  浏览器通常会尝试从 web 服务器或文件系统加载源代码。所以必须确保正确设置 [`output.devtoolModuleFilenameTemplate`](https://v4.webpack.docschina.org/configuration/output/#output-devtoolmodulefilenametemplate)，以匹配源代码的 url。

* **（仅限行）**

  source map 被简化为每行一个映射。这通常意味着每个语句只有一个映射（假设你使用这种方式）。这会妨碍你在语句级别上调试执行，也会妨碍你在每行的一些列上设置断点。与压缩后的代码组合后，映射关系是不可能实现的，因为压缩工具通常只会输出一行。

#### 适合开发环境的devtool选项

* **eval**

  每个模块都使用 `eval()` 执行，并且都有 `//@ sourceURL`。此选项会非常快地构建。

  缺点是，由于会映射到转换后的代码，而不是映射到原始代码（没有从 loader 中获取 source map），所以不能正确的显示行数。

* **eval-source-map**

  每个模块使用 `eval()` 执行，并且 source map 转换为 DataUrl 后添加到 `eval()` 中。初始化 source map 时比较慢，但是会在重新构建时提供比较快的速度，并且生成实际的文件。行数能够正确映射，因为会映射到原始代码中。<u>它会生成用于开发环境的最佳品质的 source map</u>。

* **cheap-eval-source-map**

  类似 `eval-source-map`，每个模块使用 `eval()` 执行。这是 "cheap(低开销)" 的 source map，因为它没有生成列映射(column mapping)，只是映射行数。它会忽略源自 loader 的 source map，并且仅显示转译后的代码，就像 `eval` devtool。

* **cheap-module-eval-source-map**

  类似 `cheap-eval-source-map`，并且，在这种情况下，源自 loader 的 source map 会得到更好的处理结果。然而，loader source map 会被简化为每行一个映射(mapping)。

#### 适合生产环境的devtool选项

* **none**

  省略 `devtool` 选项。不生成 source map

* **source-map**

   整个 source map 作为一个单独的文件生成。它为 bundle 添加了一个引用注释，以便开发工具知道在哪里可以找到它。

  注意*应该将服务器配置为，不允许普通用户访问 source map 文件！*

* **hidden-source-map**

  与 `source-map` 相同，但不会为 bundle 添加引用注释。

  如果只想 source map 映射那些源自错误报告的错误堆栈跟踪信息，但不想为浏览器开发工具暴露 source map，这个选项会很有用。

  注意*不应将 source map 文件部署到 web 服务器。而是只将其用于错误报告工具。*

* **nosources-source-map**

  创建的 source map 不包含 `sourcesContent(源代码内容)`。它可以用来映射客户端上的堆栈跟踪，而无须暴露所有的源代码。

  可以将 source map 文件部署到 web 服务器。*这仍然会暴露反编译后的文件名和结构，但它不会暴露原始代码。*

**注意：** *在使用* `terser-webpack-plugin` *时，你必须提供* `sourceMap：true` *选项来启用 source map 支持。*

#### 特定场景

以下选项对于开发环境和生产环境并不理想。他们是一些特定场景下需要的，例如，针对一些第三方工具。

* **inline-source-map**

  source map 转换为 DataUrl 后添加到 bundle 中

* **cheap-source-map**

  没有列映射(column mapping)的 source map，忽略 loader source map。

* **inline-cheap-source-map**

  类似 `cheap-source-map`，但是 source map 转换为 DataUrl 后添加到 bundle 中。

* **cheap-module-source-map**

  没有列映射(column mapping)的 source map，将 loader source map 简化为每行一个映射(mapping)。

* **inline-cheap-module-source-map**

   类似 `cheap-module-source-map`，但是 source map 转换为 DataUrl 添加到 bundle 中。

### 解决手动输入`npm run xxx`的麻烦

webpack 提供几种可选方式，帮助在代码发生变化后自动编译代码：

#### webpack watch mode(webpack 观察模式)

```javascript
 # package.json
"scripts": {
    "watch":"webpack --watch"
  },
```

在命令行中运行 `npm run watch`，每次修改会自动编译代码。

唯一的缺点是，为了看到修改后的实际效果，需要刷新浏览器。

#### webpack-dev-server

***配置文档：*** https://v4.webpack.docschina.org/configuration/dev-server

`webpack-dev-server` 为你提供了一个简单的 web server，并且具有 live reloading(实时重新加载) 功能。

`npm install --save-dev webpack-dev-server`

```javascript
# webpack.config.js
module.exports = {
    devServer:{
      contentBase: './dist', // 告知 webpack-dev-server，将 dist 目录下的文件 serve 到 localhost:8080 下
    }
}
```

*webpack-dev-server 在编译之后不会写入到任何输出文件。而是将 bundle 文件保留在内存中，然后将它们 serve 到 server 中，就好像它们是挂载在 server 根路径上的真实文件一样。如果希望页面在其他不同路径中找到 bundle 文件，则可以通过 dev server 配置中的* [`publicPath`](https://v4.webpack.docschina.org/configuration/dev-server/#devserver-publicpath-) *选项进行修改。*

```json
# package.json  
"scripts": {
    "start": "webpack-dev-server --open"
},
```

运行 `npm start`，浏览器自动加载页面,更改任何源文件并保存它们，web server 将在编译代码后自动重新加载。

##### 启用热模块更新HMR

更新 webpack-dev-server 配置，然后使用 webpack 内置的 HMR 插件。*如果你在技术选型中使用了* `webpack-dev-middleware` *而没有使用* `webpack-dev-server`*，请使用* [`webpack-hot-middleware`](https://github.com/webpack-contrib/webpack-hot-middleware) *package，以在你的自定义 server 或应用程序上启用 HMR。*

```javascript
# webpack.config.js
const webpack = require('webpack');
module.exports={
    devServer: {
        contentBase: './dist',
        hot: true
    },
    plugins:[
        new webpack.HotModuleReplacementPlugin()
    ]
}
```

```json
# package.json
{
  "scripts": {
    "start": "webpack-dev-server --hotOnly"
  },
}
```

修改 `index.js` 文件，以便在 `print.js` 内部发生变更时，告诉 webpack 接受 updated module

```javascript
  if (module.hot) {
    // 在 print.js 内部发生变更时，告诉 webpack 接受 updated module
    module.hot.accept('./print.js',function () {
      console.log('Accepting the updated printMe module!');
      printMe();
    })
  }
```

**通过 Node.js API **

在 Node.js API 中使用 webpack dev server 时，不要将 dev server 选项放在 webpack 配置对象(webpack config object)中。而是，在创建时，将其作为第二个参数传递。例如：

```
new WebpackDevServer(compiler, options)
```

想要启用 HMR，还需要修改 webpack 配置对象，使其包含 HMR 入口起点。webpack-dev-server的`addDevServerEntrypoints` 的方法可以通过使用这个方法来实现。

```javascript
# dev-server.js
const webpackDevServer = require('webpack-dev-server');
const webpack = require('webpack');

const config = require('./webpack.config.js');
const options = {
  contentBase: './dist',
  hot: true,
  host: 'localhost'
};

webpackDevServer.addDevServerEntrypoints(config, options);
const compiler = webpack(config);
const server = new webpackDevServer(compiler, options);

server.listen(5000, 'localhost', () => {
  console.log('dev server listening on port 5000');
});
```

借助于 `style-loader`，使用模块热替换来加载 CSS 实际上极其简单。此 loader 在幕后使用了 `module.hot.accept`，在 CSS 依赖模块更新之后，会将其 patch(修补) 到 `<style>` 标签中

```javascript
# webpack.config.js
module: {
    rules: [
        {
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
        }
    ]
},
```

社区还提供许多其他 loader 和示例，可以使 HMR 与各种框架和库平滑地进行交互……

- [React Hot Loader](https://github.com/gaearon/react-hot-loader)：实时调整 react 组件。
- [Vue Loader](https://github.com/vuejs/vue-loader)：此 loader 支持 vue 组件的 HMR，提供开箱即用体验。
- [Elm Hot Loader](https://github.com/fluxxu/elm-hot-loader)：支持 Elm 编程语言的 HMR。
- [Angular HMR](https://github.com/gdi2290/angular-hmr)：没有必要使用 loader！直接修改 NgModule 主文件就够了，它可以完全控制 HMR API。

#### webpack-dev-middleware

`webpack-dev-middleware` 是一个封装器(wrapper)，它可以把 webpack 处理过的文件发送到一个 server。`webpack-dev-server` 在内部使用了它，然而它也可以作为一个单独的 package 来使用，以便根据需求进行更多自定义设置。

下面是一个 webpack-dev-middleware 配合 express server 的示例:

​	安装：`npm install --save-dev express webpack-dev-middleware`

```javascript
# webpack.config.js
  module.exports = {
    mode: 'development',
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
      publicPath: '/', // 在 server 脚本使用 publicPath，以确保文件资源能够正确地 serve 在 http://localhost:3000 下
    }
  };
```

```javascript
# server.js
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');

const app = express();
const config = require('./webpack.config.js');
const compiler = webpack(config);

// 告诉 express 使用 webpack-dev-middleware，
// 以及将 webpack.config.js 配置文件作为基础配置
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath
}));

// 将文件 serve 到 port 3000。
app.listen(3000, function () {
  console.log('Example app listening on port 3000!\n');
});
```

```javascript
# package.json
{
    "scripts": {
        "server": "node server.js",
    },
}
```

**注意：**某些编辑器具有 "safe write(安全写入)" 功能，会影响重新编译：

- **Sublime Text 3**：在用户首选项(user preferences)中添加 `atomic_save: 'false'`。
- **JetBrains IDEs (e.g. WebStorm)**：在 `Preferences > Appearance & Behavior > System Settings` 中取消选中 "Use safe write"。
- **Vim**：在设置(settings)中增加 `:set backupcopy=yes`。

## tree shaking

>  移除 JavaScript 上下文中的未引用代码(dead-code)。

```javascript
# webpack.config.js
module.exports = {
     mode: 'development', // 要删除无用的代码需要设置为"production"，并删除optimization属性
    optimization: { // 找出需要删除的“未引用代码(dead code)”
        usedExports: true
    }
}
```

### 将文件标记为 side-effect-free(无副作用)

可以在 [`module.rules` 配置选项](https://v4.webpack.docschina.org/configuration/module/#module-rules) 中设置 `"sideEffects"`。

也可通过 package.json 的 `"sideEffects"` 属性，来实现这种方式。

* 如果所有代码都不包含 side effect，就可以简单地将该属性标记为 `false`，来告知 webpack，它可以安全地删除未用到的 export。

  ```json
  {
    "name": "your-project",
    "sideEffects": false
  }
  ```

* 如果代码确实有一些副作用，可以改为提供一个数组，数组方式支持相对路径、绝对路径和 glob 模式匹配相关文件。这些文件将不会进行tree shaking。

  ```json
  {
    "name": "your-project",
    "sideEffects": [
      "./src/some-side-effectful-file.js"
    ]
  }
  ```

* *所有导入文件都会受到 tree shaking 的影响。这意味着，如果在项目中使用类似* `css-loader` *并 import 一个 CSS 文件，则需要将其添加到 side effect 列表中，以免在生产模式中无意中将它删除：*

  ```json
  {
    "name": "your-project",
    "sideEffects": [
      "./src/some-side-effectful-file.js",
      "*.css"
    ]
  }
  ```

通过 `import` 和 `export` 语法，上面的操作已经找出需要删除的“未引用代码(dead code)”，然而，不仅仅是要找出，还要在 bundle 中删除它们。为此，我们需要将 `mode` 配置选项设置为 [`production`](https://v4.webpack.docschina.org/concepts/mode/#mode-production)。*运行 tree shaking 需要* [ModuleConcatenationPlugin](https://v4.webpack.docschina.org/plugins/module-concatenation-plugin)*。通过* `mode: "production"` *可以添加此插件。如果没有使用 mode 设置，记得手动添加* [ModuleConcatenationPlugin](https://v4.webpack.docschina.org/plugins/module-concatenation-plugin)*。*

### 结论

使用 *tree shaking* 必须注意以下：

* 使用 ES2015 模块语法（即 `import` 和 `export`）
* 确保没有 compiler 将 ES2015 模块语法转换为 CommonJS 模块（这也是流行的 Babel preset 中 @babel/preset-env 的默认行为 - 更多详细信息请查看 [文档](https://babel.docschina.org/docs/en/babel-preset-env#modules)）。
* 在项目 `package.json` 文件中，添加一个 "sideEffects" 属性。
* 通过将 `mode` 选项设置为 [`production`](https://v4.webpack.docschina.org/concepts/mode/#mode-production)，启用 minification(代码压缩) 和 tree shaking。

## 生产环境

*development(开发环境)* 和 *production(生产环境)* 这两个环境下的构建目标存在着巨大差异。所以建议为每个环境编写**彼此独立的 webpack 配置**，并把它们之间公共通用的配置保留出来，使用一个名为 [`webpack-merge`](https://github.com/survivejs/webpack-merge) 的工具来合并配置。

1. 安装：`npm install --save-dev webpack-merge`

2. 添加`webpack.common.js`、`webpack.dev.js`、`webpack.prod.js`三个文件

   ```javascript
   # webpack.common.js
   const path = require('path')
   const { CleanWebpackPlugin } = require('clean-webpack-plugin')
   const HtmlWebpackPlugin = require('html-webpack-plugin')
   
   module.exports={
       entry:{
           app: './src/index.js'
       },
       plugins:[
           new CleanWebpackPlugin(),
           new HtmlWebpackPlugin({
               title:'管理输出'
           })
       ],
       output:{
           filename: '[name].bundle.js',
           path: path.resolve(__dirname, 'dist')
       }
   }
   ```

   ```javascript
   # webpack.dev.js
   const merge = require('webpack-merge')
   const common = require('./webpack.common')
   
   module.exports = merge(common, {
       mode:'development',
       devtool:'inline-source-map',
       devServer:{
           contentBase:'./dist'
       }
   })
   ```

   ```javascript
   # webpack.prod.js
   const merge = require('webpack-merge')
   const common = require('./webpack.common')
   
   module.exports = merge(common, {
       mode:'production',
       devtool: 'source-map', // 避免在生产中使用 inline-*** 和 eval-***，因为它们会增加 bundle 体积大小，并降低整体性能
   })
   ```

   ```json
   # package.json
       "scripts": {
        "start": "webpack-dev-server --open --config webpack.dev.js",
        "build": "webpack --config webpack.prod.js"
       },
   ```

**注意：** 从 webpack v4 开始, 指定 [`mode`](https://v4.webpack.docschina.org/concepts/mode/) 会自动地配置 [`DefinePlugin`](https://v4.webpack.docschina.org/plugins/define-plugin)；还要注意，**任何位于 `/src` 的本地代码都可以使用到 process.env.NODE_ENV 环境变量**。

## (重要)代码分离

> 把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。

常用的代码分离方法有三种：

- 入口起点：使用 [`entry`](https://v4.webpack.docschina.org/configuration/entry-context) 配置手动地分离代码。
- 防止重复：使用 [`SplitChunksPlugin`](https://v4.webpack.docschina.org/plugins/split-chunks-plugin/) 去重和分离 chunk。
- 动态导入：通过模块中的内联函数调用来分离代码。

### 入口起点entry points

```javascript
module.exports = {
  mode: 'development',
  entry: {
    index: './src/index.js',
    another: './src/another-module.js'
  }
 }

...
            Asset     Size   Chunks             Chunk Names
another.bundle.js  550 KiB  another  [emitted]  another
  index.bundle.js  550 KiB    index  [emitted]  index
Entrypoint index = index.bundle.js
Entrypoint another = another.bundle.js
...
```

**缺点：** 

1. 如果入口 chunk 之间包含一些重复的模块，那些重复模块都会被引入到各个 bundle 中。
2. 不够灵活，并且不能动态地将核心应用程序逻辑中的代码拆分出来。

### 防止重复 prevent duplication

[`SplitChunksPlugin`](https://v4.webpack.docschina.org/plugins/split-chunks-plugin/) 插件可以将公共的依赖模块提取到已有的 entry chunk 中，或者提取到一个新生成的 chunk。

使用 [`optimization.splitChunks`](https://v4.webpack.docschina.org/plugins/split-chunks-plugin/#optimization-splitchunks) 配置选项:

```javascript
optimization:{
    splitChunks:{
        chunks: 'all'
    }
}
```

**对于代码分离很有帮助的 plugin 和 loader**：

- [`mini-css-extract-plugin`](https://v4.webpack.docschina.org/plugins/mini-css-extract-plugin)：用于将 CSS 从主应用程序中分离。
- [`bundle-loader`](https://v4.webpack.docschina.org/loaders/bundle-loader)：用于分离代码和延迟加载生成的 bundle。
- [`promise-loader`](https://github.com/gaearon/promise-loader)：类似于 `bundle-loader` ，但是使用了 promise API。

### 动态导入 dynamic  imports

#### 1. (推荐)使用`import()`语法实现动态导入

> `import()` *调用会在内部用到* [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)*。如果在旧版本浏览器中使用* `import()`*，记得使用一个 polyfill 库（例如* [es6-promise](https://github.com/stefanpenner/es6-promise) *或* [promise-polyfill](https://github.com/taylorhakes/promise-polyfill)*），来 shim* `Promise`*。*

```javascript
module.exports = {
    entry: {
        index: './src/dynamicImport.js'
    },
    output: {
        filename: '[name].bundle.js',
        chunkFilename: '[name].bundle.js', // chunkFilename决定 non-entry chunk(非入口 chunk) 的名称
        path: path.resolve(__dirname, 'dist')
    },
}
```

```javascript
# dynamicImport.js
/**
 * 通过 dynamic import(动态导入) 来分离出一个 chunk
 */
async function getComponent(){
// function getComponent() {
    // return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default:_ })=>{
    //     let element = document.createElement('div');
    //     element.innerHTML = _.join(['hello', 'webpack'], ' ');
    //     return element;
    // }).catch(error => '加载lodash出现错误!');
    let element = document.createElement('div');
    const { default:_ } = await import(/* webpackChunkName:"lodash" */ 'lodash'); // import() 会返回一个 promise，因此它可以和 async 函数一起使用。
    element.innerHTML = _.join(['hello', 'webpack'], ' ');
    return element;
}
getComponent().then(component => {
    document.body.appendChild(component);
})
```

* 使用 `default` 的原因是，从 webpack v4 开始，在 import CommonJS 模块时，不会再将导入模块解析为 `module.exports` 的值，而是为 CommonJS 模块创建一个 artificial namespace object(人工命名空间对象)

* 注释中我们提供了 `webpackChunkName`。这样会将拆分出来的 bundle 命名为 `lodash.bundle.js`，而不是 `[id].bundle.js`。

#### 2. 使用 webpack 特定的 [`require.ensure`](https://v4.webpack.docschina.org/api/module-methods#require-ensure)



### 预取/预加载模块 prefetch/preload module

在声明 import 时，使用下面这些内置指令，可以让 webpack 输出 "resource hint(资源提示)"，来告知浏览器：

- **prefetch(预取)：将来某些导航下可能需要的资源**

  ```javascript
  # LoginButton.js  prefetch 的简单示例
  import(/* webpackPrefetch: true */ 'LoginModal'); // 在HomePage 组件内部渲染一个 LoginButton 组件，然后在点击后按需加载 LoginModal 组件。
  ```

  这会生成 `<link rel="prefetch" href="login-modal-chunk.js">` 并追加到页面头部，指示着浏览器在闲置时间预取 `login-modal-chunk.js` 文件。 *只要父 chunk 完成加载，webpack 就会添加 prefetch hint(预取提示)。*

- **preload(预加载)：当前导航下可能需要资源**

  与 prefetch 指令相比，preload 指令有许多不同之处：

  - preload chunk 会在父 chunk 加载时，以并行方式开始加载。prefetch chunk 会在父 chunk 加载结束后开始加载。
  - preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载。
  - preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻。
  - 浏览器支持程度不同。

  ```javascript
  # ChartComponent.js
  // 假想这里的图表组件 ChartComponent 组件需要依赖体积巨大的 ChartingLibrary 库。它会在渲染时显示一个 LoadingIndicator(加载进度条) 组件，然后立即按需导入 ChartingLibrary：
  import(/* webpackPreload: true */ 'ChartingLibrary');
  ```

  **注意：** *不正确地使用 webpackPreload 会有损性能，请谨慎使用。*

### bundle 分析

- [webpack-chart](https://alexkuz.github.io/webpack-chart/)：webpack stats 可交互饼图。
- [webpack-visualizer](https://chrisbateman.github.io/webpack-visualizer/)：可视化并分析你的 bundle，检查哪些模块占用空间，哪些可能是重复使用的。
- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)：一个 plugin 和 CLI 工具，它将 bundle 内容展示为便捷的、交互式、可缩放的树状图形式。
- [webpack bundle optimize helper](https://webpack.jakoblind.no/optimize)：此工具会分析你的 bundle，并为你提供可操作的改进措施建议，以减少 bundle 体积大小。

## (重要)懒加载

> 在用户点击交互的时候，再加载那个代码块

```javascript
# lazyLoading.js
import _ from 'lodash'
function component(){
    let element = document.createElement('div');
    let button = document.createElement('button');
    let br = document.createElement('br');
    button.innerHTML = 'Click me and at the console.';
    element.appendChild(br);
    element.appendChild(button);
    // 请注意，由于涉及网络请求，因此需要在生产级站点/应用程序中显示一些加载指示。
    button.onclick = e => import(/* webpackChunkName: "lazyLoading" */ './print').then( module =>{
        let printMe =  module.default;
        printMe();
    })
    return element;
}
document.body.appendChild(component());
```

**注意：**  当调用 ES6 模块的 import() 方法（引入模块）时，必须指向模块的 .default 值，因为它才是 promise 被处理后返回的实际的 module 对象。

许多框架和类库对于如何用它们自己的方式来实现（懒加载）都有自己的建议。这里有一些例子：

- React: [Code Splitting and Lazy Loading](https://reacttraining.com/react-router/web/guides/code-splitting)
- Vue: [Lazy Load in Vue using Webpack's code splitting](https://alexjoverm.github.io/2017/07/16/Lazy-load-in-Vue-using-Webpack-s-code-splitting/)
- AngularJS: [AngularJS + Webpack = lazyLoad](https://medium.com/@var_bin/angularjs-webpack-lazyload-bb7977f390dd) by [@var_bincom](https://twitter.com/var_bincom)

## 缓存

### 输出文件名contenthash

```javascript
# webpack.config.js
module.exports = {
    output: {
        filename:'[name].[contenthash].js', // 当资源内容发生变化时，[contenthash] 也会发生变化。
    }
}
```

### 提取引导模板 extracting boilerplate(提取第三方库)

[`SplitChunksPlugin`](https://v4.webpack.docschina.org/plugins/split-chunks-plugin/) 可以用于将模块分离到单独的 bundle 中。

使用 [`optimization.runtimeChunk`](https://v4.webpack.docschina.org/configuration/optimization/#optimization-runtimechunk) 选项将 runtime 代码拆分为一个单独的 chunk。将其设置为 `single` 来为所有 chunk 创建一个 runtime bundle，另外**将第三方库(library)（例如 `lodash` 或 `react`）提取到单独的 `vendor` chunk 文件中，是比较推荐的做法**：

```javascript
# webpack.config.js
module.exports = {
    optimization: {
        runtimeChunk: 'single', // 将 runtime 代码拆分为一个单独的 chunk
        splitChunks:{ // 将第三方库library提取到单独的vendor chunk文件中
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name:'vendors',
                    chunks: 'all'
                }
            }
        }
    }
}
```

**注意：** 每次打包时，如果修改了某个文件的代码，则所有文件的contenhash都发生变化。这是因为每个 [`module.id`](https://v4.webpack.docschina.org/api/module-variables#module-id-commonjs-) 会默认地基于解析顺序(resolve order)进行增量。也就是说，当解析顺序发生变化，ID 也会随之改变。

```bash
...
                           Asset       Size  Chunks                    Chunk Names
  runtime.1400d5af64fc1b7b3a45.js    5.85 kB      0  [emitted]         runtime
  vendor.a7561fb0e9a071baadb9.js     541 kB       1  [emitted]  [big]  vendor
    main.b746e3eb72875af2caa9.js    1.22 kB       2  [emitted]         main
                      index.html  352 bytes          [emitted]
...
```

因此，简要概括：

- `main` bundle 会随着自身的新增内容的修改，而发生变化。
- `vendor` bundle 会随着自身的 `module.id` 的变化，而发生变化。
- `manifest` bundle 会因为现在包含一个新模块的引用，而发生变化。

但是，第三方库的vender hash不应该发生变化(个人测试这个问题在webpack4不存在，不用插件)。可用2个插件修复，使得该hash不变。

1. **NamedModulesPlugin**

   使用模块的路径，而不是一个数字 identifier。此插件有助于在开发环境下产生更加可读的输出结果，然而其执行时间会有些长。

2. **[`HashedModuleIdsPlugin`](https://v4.webpack.docschina.org/plugins/hashed-module-ids-plugin)**

   推荐用于生产环境构建

   ```javascript
   # webpack.config.js
   const webpack = require('webpack');
   module.exports = {
       plugins:[
           ...
   new webpack.HashedModuleIdsPlugin() // 生产环境插件。启用contenthash修复，vendor第三方chunk hash不变
   
       ]
   }
   
   ```

## shim预置依赖

> *shim 是一个库(library)，它将一个新的 API 引入到一个旧的环境中，而且仅靠旧的环境中已有的手段实现。polyfill 就是一个用在浏览器 API 上的 shim。我们通常的做法是先检查当前浏览器是否支持某个 API，如果不支持的话就按需加载对应的 polyfill。然后新旧浏览器就都可以使用这个 API 了。*

### shim 预置全局变量(提取第三方库提供的全局变量)

使用 [`ProvidePlugin`](https://v4.webpack.docschina.org/plugins/provide-plugin) 插件，在 webpack 编译的每个模块中，通过访问一个变量来获取一个 package。

```javascript
# webpack.config.js
const webpack = require('webpack');
module.exports = {
    plugins:[
        // 该插件告诉webpack遇到了至少一处用到 _ 变量的模块实例，
        // 那将 lodash package 引入进来，并将其提供给需要用到它的模块。舍去了 import _ from 'lodash'
        new webpack.ProvidePlugin({
            _:'lodash',
            join: ['lodash', 'join'] // 还可以使用 ProvidePlugin 暴露出某个模块中单个导出，通过配置一个“数组路径”（例如 [module, child, ...children?]）实现此功能。这样就能很好的与 tree shaking 配合，将 lodash library 中的其余没有用到的导出去除
        })
    ]
}
```

### 细粒度 shim(模块中this的指向)

一些遗留模块依赖的 `this` 指向的是 `window` 对象。

```javascript
# index.js
  function component() {
   // 假设处于 `window` 上下文
   this.alert('Hmmm, this probably isn\'t a great idea...')
  }
```

当模块运行在 CommonJS 上下文中，这将会变成一个问题，也就是说此时的 `this` 指向的是 `module.exports`。在这种情况下，你可以通过使用 [`imports-loader`](https://v4.webpack.docschina.org/loaders/imports-loader/) 覆盖 `this` 指向：

```javascript
# webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: require.resolve('index.js'),
                use: 'imports-loader?this=>window'
            }
        ]
    },
}
```

### 全局 export

```javascript
# globals.js
var file = 'blah.txt';
var helpers = {
  test: function() { console.log('test something'); },
  parse: function() { console.log('parse something'); }
};
```

使用 [`exports-loader`](https://v4.webpack.docschina.org/loaders/exports-loader/)，将一个全局变量作为一个普通的模块来导出。例如，为了将 `file` 导出为 `file` 以及将 `helpers.parse` 导出为 `parse`，做如下调整：

```javascript
# webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: require.resolve('globals.js'),
                use: 'exports-loader?file,parse=helpers.parse'
            }
        ]
    },
}
```

现在在 entry 入口文件中（即 `src/index.js`），能 `import { file, parse } from './globals.js';` ，一切将顺利运行。

### 加载 polyfill

安装`npm install --save babel-polyfill`

使用 `import` 将其引入到的主 bundle 文件:

```javascript
# index.js
import 'babel-polyfill';
...
```

**注意：** *没有将* `import` *绑定到某个变量是因为 polyfill 直接基于自身执行，并且是在基础代码执行之前，这样通过这些预置就可以假定已经具有某些原生功能。*这种方式优先考虑正确性，而不考虑 bundle 体积大小。为了安全和可靠，polyfill/shim 必须**运行于所有其他代码之前**，而且需要同步加载，或者说，需要在所有 polyfill/shim 加载之后，再去加载所有应用程序代码。 

如果决定承受损坏的风险，则加入 [`whatwg-fetch`](https://github.com/github/fetch) polyfill：

安装`npm install --save whatwg-fetch`

```javascript
# polyfills.js
import 'babel-polyfill';
import 'whatwg-fetch';

# webpack.config.js
module.exports = {
    entry: {
        polyfills: './src/polyfills.js',
        index: './src/index.js'
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
}
```

如上配置之后，可以在代码中添加一些逻辑，条件地加载新的 `polyfills.bundle.js` 文件。根据需要支持的技术和浏览器来决定是否加载。

```html
# index.html
<head>
    <title>Getting Started</title>
    <script>
        var modernBrowser = (
            'fetch' in window &&
            'assign' in Object
        );
        if ( !modernBrowser ) { // 如果这些属性不存在，则使用polyfill
            var scriptElement = document.createElement('script');
            scriptElement.async = false;
            scriptElement.src = '/polyfills.bundle.js';
            document.head.appendChild(scriptElement);
        }
    </script>
</head>
```

```javascript
# index.js
fetch('https://jsonplaceholder.typicode.com/users')
    .then(response => response.json())
    .then(json => {
    console.log('We retrieved some data! AND we\'re confident it will work on a variety of browser distributions.')
    console.log(json)
}).catch(error => console.error('Something went wrong when fetching this data: ', error))
```

**`babel-preset-env`** package 通过 [browserslist](https://github.com/browserslist/browserslist) 来转译那些你浏览器中不支持的特性。这个 preset 使用 [`useBuiltIns`](https://babel.docschina.org/docs/en/babel-preset-env#usebuiltins) 选项，默认值是 `false`，这种方式可以将全局 `babel-polyfill` 导入，改进为更细粒度的 `import` 格式：

查看 [仓库](https://github.com/babel/babel-preset-env) 以获取更多信息。

```js
import 'core-js/modules/es7.string.pad-start';
import 'core-js/modules/es7.string.pad-end';
import 'core-js/modules/web.timers';
import 'core-js/modules/web.immediate';
import 'core-js/modules/web.dom.iterable';
```

### 处理遗留模块的工具

[`script-loader`](https://v4.webpack.docschina.org/loaders/script-loader/) 会在 global context(全局上下文) 中对代码进行 eval 取值，这类似于通过一个 `script` 标签引入脚本。在这种模式下，每个正常的 library 都应该能运行。对 `require`, `module` 等取值是 undefined。*在使用* `script-loader` *时，模块将转为一个字符串，然后添加到 bundle 中。它不会被* `webpack` *压缩，所以你应该选择一个 min 版本。而且，使用此 loader 添加的 library 也没有* `devtool` *支持。*

这些遗留模块如果没有 AMD/CommonJS 版本，但你也想将他们加入 `dist` 文件，则可以使用 [`noParse`](https://v4.webpack.docschina.org/configuration/module/#module-noparse) 来标识出这个模块。这样就能使 webpack 将引入这些模块，但是不进行转化(parse)，以及不解析(resolve) `require()` 和 `import` 语句。这种用法还会提高构建性能。*任何需要 AST 的功能（例如* `ProvidePlugin`*）都不起作用。*

最后，一些模块支持多种 [模块格式](https://v4.webpack.docschina.org/concepts/modules)，例如一个混合有 AMD、CommonJS 和 legacy(遗留) 的模块。在大多数这样的模块中，会首先检查 `define`，然后使用一些怪异代码导出一些属性。在这些情况下，可以通过 [`imports-loader`](https://v4.webpack.docschina.org/loaders/imports-loader/) 设置 `define=>false` 来强制 CommonJS 路径。

## PWA渐进式网络应用程序

1. 安装：

   使用[http-server](https://www.npmjs.com/package/http-server) package(`npm install http-server -D`)

   使用名为 [Workbox](https://github.com/GoogleChrome/workbox) 的 Google 项目为应用程序添加离线体验(`npm install workbox-webpack-plugin -D`)

2. 使用

   ```json
   # package.json
   {
     "scripts": {
       "build": "webpack",
       "start": "http-server dist"
     }
   }
   ```

   ```javascript
   # webpack.config.js
     const path = require('path');
     const HtmlWebpackPlugin = require('html-webpack-plugin');
     const CleanWebpackPlugin = require('clean-webpack-plugin');
     const WorkboxPlugin = require('workbox-webpack-plugin');
   
     module.exports = {
       entry: {
         app: './src/index.js',
         print: './src/print.js'
       },
       plugins: [
         new CleanWebpackPlugin(['dist']),
         new HtmlWebpackPlugin({
          title: '渐进式网络应用程序'
        })
        }),
        new WorkboxPlugin.GenerateSW({
          // 这些选项帮助快速启用 ServiceWorkers
          // 不允许遗留任何“旧的” ServiceWorkers
          clientsClaim: true,
          skipWaiting: true
        })
       ],
       output: {
         filename: '[name].bundle.js',
         path: path.resolve(__dirname, 'dist')
       }
     };
   ```

   ```javascript
   # index.js
     import _ from 'lodash';
     import printMe from './print.js';
   // 注册 Service Worker，使其出场并开始表演
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('/service-worker.js').then(registration => {
          console.log('SW registered: ', registration);
        }).catch(registrationError => {
         console.log('SW registration failed: ', registrationError);
        });
      });
    }
   ```

3. 运行

   * 执行`npm run build`来构建包含注册代码版本的应用程序。

     会生成`service-worker.js`和`workbox-xxxxx.js`的两个文件。`ervice-worker.js` 是 Service Worker 文件，`workbox-xxxxx.js` 是 `service-worker.js` 引用的文件，所以它也可以运行。

   * 执行`npm start`将构建结果 serve 到服务下，停止 server 并刷新页面。如果浏览器能够支持 Service Worker，可以看到应用程序还在正常运行。然而，server 已经**停止** serve 整个 dist 文件夹，此刻是 Service Worker 在进行 serve。

## TypeScript

1. 安装 TypeScript compiler 和 loader：

   `npm install --save-dev typescript ts-loader`

2. 配置文件

   查看 [TypeScript 官方文档](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) 了解更多关于 `tsconfig.json` 的配置选项

   ```json
   # tsconfig.json
   {
       // 设置一个基本的配置，来支持 JSX，并将 TypeScript 编译到 ES5
       "compilerOptions":{
           "outDir": "./dist/",
           "sourceMap": true, // 启用 source map,将内联的 source map 输出到编译后的 JavaScript 文件中
           "noImplicitAny": true,
           "module": "es6",
           "target": "es5",
           "jsx": "react",
           "allowJs":true
       }
   }
   ```

   ```javascript
   # webpack.config.js
   const path = require('path');
   
   module.exports = {
     entry: './src/index.ts',
     devtool: 'inline-source-map', // 告诉 webpack 提取这些typescript的 source map，并内联到最终的 bundle 中
     module: {
       rules: [
         {
           test: /\.tsx?$/,
           use: 'ts-loader',
           exclude: /node_modules/
         }
       ]
     },
     resolve: {
       extensions: [ '.tsx', '.ts', '.js' ]
     },
     output: {
       filename: 'bundle.js',
       path: path.resolve(__dirname, 'dist')
     }
   };
   ```

### 使用第三方库

在从 npm 安装 third party library(第三方库) 时，一定要同时安装此 library 的类型声明文件(typing definition)。可以从 [TypeSearch](http://microsoft.github.io/TypeSearch/) 中找到并安装这些第三方库的类型声明文件。

如安装 lodash 类型声明文件：`npm install --save-dev @types/lodash`

### 导入其他资源

在 TypeScript 中使用非代码资源(non-code asset)，需要告诉 TypeScript 推断导入资源的类型。

```javascript
# custom.d.ts 该文件用来表示项目中 TypeScript 的自定义类型声明
declare module "*.svg" { // 为 .svg 文件设置一个声明
  const content: any;
  export default content;
}
```

通过指定任何以 `.svg` 结尾的导入(import)，将 SVG 声明(declare) 为一个新的模块(module)，并将模块的 `content` 定义为 `any`。我们可以通过将类型定义为字符串，来更加显式地将它声明为一个 url。同样的概念适用于其他资源，包括 CSS, SCSS, JSON 等

## 生成环境变量

在 `webpack.config.js` 中可以访问到在 webpack 命令行 [环境配置](https://v4.webpack.docschina.org/api/cli/#environment-options) 的 `--env` 参数中传入的任意数量的环境变量。

```sh
# 如果设置 env 变量，却没有赋值，--env.production 默认表示将 --env.production 设置为 true。
webpack --env.NODE_ENV=local --env.production --progress
```

通常，`module.exports` 指向配置对象。要使用 `env` 变量，必须将 `module.exports` 转换成一个函数：

```javascript
# webpack.config.js
const path = require('path');

module.exports = env => {
  // Use env.<YOUR VARIABLE> here:
  console.log('NODE_ENV: ', env.NODE_ENV); // 'local'
  console.log('Production: ', env.production); // true

  return {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
};
```

## 改进构建/编译性能(重要)

### loader

对最少数量的必要模块使用 loader。

不应该如下:

```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader'
      }
    ]
  }
};
```

应该使用 `include` 字段仅将 loader 应用在实际需要将其转换的模块所处路径：

```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'babel-loader'
      }
    ]
  }
};
```

### 解析

以下步骤可以提高解析速度:

- 减少 `resolve.modules`, `resolve.extensions`, `resolve.mainFiles`, `resolve.descriptionFiles` 中 items 数量，因为他们会增加文件系统调用的次数。
- 如果不使用 symlinks（例如 `npm link` 或者 `yarn link`），可以设置 `resolve.symlinks: false`。
- 如果使用自定义 resolve plugin 规则，并且没有指定 context 上下文，可以设置 `resolve.cacheWithContext: false`。

### dll

使用 `DllPlugin` 为更改不频繁的代码生成单独编译结果。这可以提高应用程序的编译速度，尽管它确实增加了构建过程的复杂度。

### 小即是快(smaller = faster) 

减少编译结果的整体大小，以提高构建性能。尽量保持 chunk 体积小。

- 使用数量更少/体积更小的 library。
- 在多页面应用程序中使用 `CommonsChunkPlugin`，并开启 `async` 模式。
- 移除未引用代码。
- 只编译你当前正在开发的那些代码。

### worker 池(worker pool) 

`thread-loader` 可以将非常消耗资源的 loader 分流给一个 worker pool。

**注意：** *不要使用太多的 worker，因为 Node.js 的 runtime 和 loader 都有启动开销。最小化 worker 和 main process(主进程) 之间的模块传输。进程间通讯(IPC, inter process communication)是非常消耗资源的。*

### 持久化缓存

使用 `cache-loader` 启用持久化缓存。使用 `package.json` 中的 `"postinstall"` 清除缓存目录。

### 开发环境下的构建配置

#### 增量编译

使用 webpack 的 watch mode(监听模式)。而不使用其他工具来 watch 文件和调用 webpack 。内置的 watch mode 会记录时间戳并将此信息传递给 compilation 以使缓存失效。

在某些配置环境中，watch mode 会回退到 poll mode(轮询模式)。监听许多文件会导致 CPU 大量负载。在这些情况下，可以使用 `watchOptions.poll` 来增加轮询的间隔。

#### 在内存中编译

下面几个工具通过在内存中（而不是写入磁盘）编译和 serve 资源来提高性能：

- `webpack-dev-server`
- `webpack-hot-middleware`
- `webpack-dev-middleware`

#### stats.toJson 加速

webpack 4 默认使用 `stats.toJson()` 输出大量数据。除非在增量步骤中做必要的统计，否则请避免获取 `stats` 对象的部分内容。`webpack-dev-server` 在 v3.1.3 以后的版本，包含一个重要的性能修复，即最小化每个增量构建步骤中，从 `stats` 对象获取的数据量。

#### devtool 

不同的 `devtool` 设置会导致不同的性能差异。

- `"eval"` 具有最好的性能，但并不能帮助你转译代码。
- 如果能接受稍差一些的 map 质量，可以使用 `cheap-source-map` 变体配置来提高性能
- 使用 `eval-source-map` 变体配置进行增量编译。

在大多数情况下，最佳选择是 `cheap-module-eval-source-map`。

#### 避免使用在生产环境下才会用到的工具

某些 utility, plugin 和 loader 都只用于生产环境。例如，在开发环境下使用 `TerserPlugin` 来 minify(压缩) 和 mangle(混淆破坏) 代码是没有意义的。通常在开发环境下，应该排除以下这些工具：

- `TerserPlugin`
- `ExtractTextPlugin`
- `[hash]`/`[chunkhash]`
- `AggressiveSplittingPlugin`
- `AggressiveMergingPlugin`
- `ModuleConcatenationPlugin`

#### 最小化 entry chunk

webpack 只会在文件系统中生成已经更新的 chunk。某些配置选项（HMR, `output.chunkFilename` 的 `[name]`/`[chunkhash]`, `[hash]`）来说，对更新的 chunk 和 entry chunk 不会生效。

确保在生成 entry chunk 时，尽量减少其体积以提高性能。下面的代码块将只提取包含 runtime 的 chunk，*其他 chunk 都作为其子 chunk*:

```js
new CommonsChunkPlugin({
  name: 'manifest',
  minChunks: Infinity
});
```

#### 避免额外的优化步骤

webpack 通过执行额外的算法任务，来优化输出结果的体积和加载性能。这些优化适用于小型代码库，但是在大型代码库中却非常耗费性能：

```js
module.exports = {
  // ...
  optimization: {
    removeAvailableModules: false,
    removeEmptyChunks: false,
    splitChunks: false,
  }
};
```

#### *关闭输出结果携带路径信息

webpack 会在输出的 bundle 中生成路径信息。然而，在打包数千个模块的项目中，这会导致造成垃圾回收性能压力。在 `options.output.pathinfo` 设置中关闭：

```js
module.exports = {
  // ...
  output: {
    pathinfo: false
  }
};
```

#### 使用 TypeScript 内置 watch mode API

现在，`ts-loader` 已经开始使用 TypeScript 内置 watch mode API，可以明显减少每次迭代时重新构建的模块数量。`experimentalWatchApi` 与普通 TypeScript watch mode 共享同样的逻辑，并且在开发环境使用时非常稳定。此外开启 `transpileOnly`，用于真正快速增量构建。

```js
module.exports = {
  // ...
  test: /\.tsx?$/,
  use: [
    {
      loader: 'ts-loader',
      options: {
        transpileOnly: true,
        experimentalWatchApi: true,
      },
    },
  ],
};
```

为了重新获得类型检查，请使用 [`ForkTsCheckerWebpackPlugin`](https://www.npmjs.com/package/fork-ts-checker-webpack-plugin)。

ts-loader 的 github 仓库中有一个 [完整示例](https://github.com/TypeStrong/ts-loader/tree/master/examples/fast-incremental-builds)。

### 生产环境下的构建配置

> **不要为了很小的性能收益，牺牲应用程序的质量！***注意，在大多数情况下，优化代码质量比构建性能更重要。*

#### 多个 compilation(编译时) 

在进行多个 compilation 时，以下工具有帮助：

- [`parallel-webpack`](https://github.com/trivago/parallel-webpack)：它允许在 worker 池中运行 compilation。
- `cache-loader`：可以在多个 compilation 之间共享缓存。

#### 工具相关问题

* **Babel**

  最小化项目中的 preset/plugins 数量。

* **TypeScript**

  - 在单独的进程中使用 `fork-ts-checker-webpack-plugin` 进行类型检查。
  - 配置 loader 跳过类型检查。
  - 使用 `ts-loader` 时，设置 `happyPackMode: true` / `transpileOnly: true`。

* **Sass**

  `node-sass` 中有个来自 Node.js 线程池的阻塞线程的 bug。 当使用 `thread-loader` 时，需要设置 `workerParallelJobs: 2`。

## 内容安全策略CSP

webpack 能够为其加载的所有脚本添加 `nonce`。要启用此功能，需要在引入的入口脚本中设置一个 `__webpack_nonce__` 变量，这可以为每个唯一的页面视图生成和提供一个唯一的基于 hash 的 nonce。`nonce` 应该是一个 base64 编码的字符串。

如在 entry 文件中：

```js
// ...
__webpack_nonce__ = 'c29tZSBjb29sIHN0cmluZyB3aWxsIHBvcCB1cCAxMjM=';
// ...
```

`nonce`属性使得可以将某些内联`script`和`style`元素“白名单” ，同时避免使用CSP`unsafe-inline`指令（这将允许*所有*内联`script`/ `style`），因此，您仍然保留了CSP的关键特性，即通常不允许内联script/style。该`nonce`属性是一种告诉浏览器的方式，即某个（恶意）第三方没有将特定脚本或样式元素的内联内容注入到文档中，而是由控制文件提供服务的服务器的人有意地放入了文档中。

该属性包括以下步骤：

1. 对于您的Web服务器收到的针对特定文档的每个请求，请让您的后端从加密安全的随机数生成器中生成至少128位数据的随机base64编码的字符串；例如`EDNnf03nceIOfn39fn3e9h3sdfa`。那是你的现时

2. 以步骤1中生成的随机数为例，对于要内联`script`/的`style`“白名单”，让您的后端代码`nonce`在通过网络发送文档之前将属性插入文档中，并以该随机数为值：

   ```javascript
    <script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">…</script>
   ```

3. 将步骤1中生成的随机数添加`nonce-`到它的前面，并使您的后端生成一个CSP标头，其中包含`script-src`或的源列表中的值`style-src`：

   ```http
    Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
   ```

因此，使用随机数的机制是一种替代方法，而不是让您的后端生成内联`script`或`style`您要允许的内联内容的哈希，然后在CSP标头中的适当源列表中指定该哈希。

请注意，由于浏览器无法（无法）检查在两次页面请求之间发送的现时值是否发生了变化，因此尽管不建议这样做，但有可能跳过上面的1并且后端不对现时值进行动态处理，在这种情况下，可以仅将`nonce`具有静态值的属性放入文档的HTML源中，并发送具有相同随机数值的静态CSP标头。

但是，您不希望以这种方式使用静态随机数的原因是，它完全违背了使用随机数的全部目的，因为如果您要使用这样的静态随机数，到那时，您不妨使用`unsafe-inline`。

默认情况下不启用 CSP。需要与文档(document)一起发送相应的 `CSP` header 或 meta 标签 `<meta http-equiv="Content-Security-Policy" ...>`，以告知浏览器启用 CSP。以下是一个包含 CDN 白名单 URL 的 CSP header 的示例：

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;

Content-Security-Policy: script-src https://host1.com https://host2.com #将这两个来源指定为有效来源

Content-Security-Policy: script-src 'self' https://apis.google.com

若有一个从内容交付网络（如https://cdn.example.net）加载所有资源的应用，并清楚您不需要任何帧内容或插件，则政策可能类似如下：
Content-Security-Policy: default-src https://cdn.example.net; child-src 'none'; object-src 'none'
```

`script-src` 是一条指令，其用于控制脚本对于某个特定页面所享有的一组权限。 我们已指定 `'self'` 作为一个有效的脚本来源，指定 `https://apis.google.com` 作为另一个有效的脚本来源。浏览器通过 HTTPS 以及当前页面的来源从 `apis.google.com` 尽职地下载和执行 JavaScript。定义此政策后，浏览器只会引发一个错误，而不会加载来自任何其他来源的脚本。 当狡猾的攻击者设法将代码注入您的网站时，他们只会看到一条错误消息，而不是他们期待的成功。

其余资源指令：(针对特定应用使用任意数量的指令，只需在 HTTP 标头中列出每条指令，并使用分号将它们隔开。)

- **`base-uri`** 用于限制可在页面的 `<base>` 元素中显示的网址。
- **`child-src`** 用于列出适用于工作线程和嵌入的帧内容的网址。例如：`child-src https://youtube.com` 将启用来自 YouTube（而非其他来源）的嵌入视频。 使用此指令替代已弃用的 **`frame-src`** 指令。
- **`connect-src`** 用于限制可（通过 XHR、WebSockets 和 EventSource）连接的来源。
- **`font-src`** 用于指定可提供网页字体的来源。Google 的网页字体可通过 `font-src https://themes.googleusercontent.com` 启用。
- **`form-action`** 用于列出可从 `<form>` 标记提交的有效端点。
- **`frame-ancestors`** 用于指定可嵌入当前页面的来源。此指令适用于 `<frame>`、`<iframe>`、`<embed>` 和 `<applet>` 标记。此指令不能在 `<meta>` 标记中使用，并仅适用于非 HTML 资源。
- **`frame-src`** 已弃用。请改用 **`child-src`**。
- **`img-src`** 用于定义可从中加载图像的来源。
- **`media-src`** 用于限制允许传输视频和音频的来源。
- **`object-src`** 可对 Flash 和其他插件进行控制。
- **`plugin-types`** 用于限制页面可以调用的插件种类。
- **`report-uri`** 用于指定在违反内容安全政策时浏览器向其发送报告的网址。此指令不能用于 `<meta>` 标记。
- **`style-src`** 是 `script-src` 版的样式表。
- **`upgrade-insecure-requests`** 指示 User Agent 将 HTTP 更改为 HTTPS，重写网址架构。 该指令适用于具有大量旧网址（需要重写）的网站。

**注意：**默认情况下指令在运行时假定 `*` 作为有效来源（可以从任意位置加载，没有任何限制）。可以通过指定一个 **`default-src`** 指令替换此默认行为。 此指令用于定义您未指定的大多数指令的默认值。 一般情况下，这适用于以 `-src` 结尾的任意指令。 如果将 `default-src` 设为 `https://example.com`，并且您未能指定一个 `font-src` 指令，那么，您可以从 `https://example.com` 加载字体，而不能从任何其他地方加载。

策略需要包含 [`default-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)或[`script-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src)指令，以防止运行内联脚本以及阻止使用`eval()`。策略需要包含 [`default-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)或[`style-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src)指令，以限制从style元素或元素中应用内联样式。`style`属性。有针对各种项目类型的特定指令，因此每种类型都可以有自己的策略，包括字体，框架，图像，音频和视频媒体，脚本和工作程序。

以下指令不使用 `default-src` 作为回退指令。请记住，如果不对其进行设置，则等同于允许加载任何内容。

- `base-uri`
- `form-action`
- `frame-ancestors`
- `plugin-types`
- `report-uri`
- `sandbox`

要启用CSP，您需要配置Web服务器以返回 [`Content-Security-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)HTTP标头。

或者

CSP 首选的传输机制是一个 HTTP 标头。在标记中直接设置一个页面政策会非常有用。 使用一个 `http-equiv` 属性通过 `<meta>` 标记进行此设置：

```html
<meta http-equiv="Content-Security-Policy" content="default-src https://cdn.example.net; child-src 'none'; object-src 'none'">
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

该政策不能用于 frame-ancestors、report-uri 或 sandbox。

### 示例

```http
# 网站管理员希望所有内容都来自网站本身的来源（不包括子域）
Content-Security-Policy: default-src 'self'
# 网站管理员希望允许来自受信任域及其所有子域的内容（不必与设置CSP的域相同）
Content-Security-Policy: default-src 'self' trusted.com *.trusted.com
# 网站管理员希望允许Web应用程序的用户在其自己的内容中包括来自任何来源的图像，但将音频或视频媒体限制为受信任的提供程序，并且将所有脚本仅限制为承载受信任代码的特定服务.默认情况下，仅允许从文档的原始内容开始，但以下情况除外：
1.图像可以从任何位置加载（请注意通配符“ *”）。
2.仅允许来自media1.com和media2.com的媒体（不允许来自这些站点的子域）。
3.可执行脚本只允许来自userscripts.example.com。
Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com
# 在线银行网站的网站管理员希望确保使用TLS加载其所有内容，以防止攻击者窃听请求。该服务器仅允许访问通过单一来源onlinebanking.jumbobank.com通过HTTPS专门加载的文档。
Content-Security-Policy: default-src https://onlinebanking.jumbobank.com
# 网站站点的网站管理员希望允许电子邮件中的HTML以及从任何地方加载的图像，但不允许JavaScript或其他潜在危险的内容。请注意，此示例未指定script-src，该站点使用default-src伪指令指定的设置，这意味着只能从原始服务器加载脚本。
Content-Security-Policy: default-src 'self' *.mailsite.com; img-src *
```

### 进一步阅读

- [Google developers中的CSP解释](https://developers.google.com/web/fundamentals/security/csp/)
- [nonce 设计目的解释](https://stackoverflow.com/questions/42922784/what-s-the-purpose-of-the-html-nonce-attribute-for-script-and-style-elements)
- [白名单的不安全性和内容安全政策的未来](https://research.google.com/pubs/pub45542.html)
- [使用 CSP, Hash, Nonce 和 Report URI 锁定你的网站脚本](https://www.troyhunt.com/locking-down-your-website-scripts-with-csp-hashes-nonces-and-report-uri/)
- [MDN 的 CSP 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

## 开发 - Vagrant

如果你在开发一个更加高级的项目，并且使用 [Vagrant](https://www.vagrantup.com/) 来实现在虚拟机(Virtual Machine)上运行你的开发环境，那你可能会需要在虚拟机中运行 webpack。

https://v4.webpack.docschina.org/guides/development-vagrant/

## 管理依赖

> es6 modules、commonjs、amd

### 带表达式的 require 语句

如果你的 request 含有表达式(expressions)，就会创建一个上下文(context)，因为在编译时(compile time)并不清楚**具体**导入哪个模块,如：`require('./template/' + name + '.ejs');`

webpack 解析 `require()` 调用，然后提取出如下一些信息：

```code
Directory: ./template
Regular expression: /^.*\.ejs$/
```

生成一个 context module(上下文模块)。它包含**目录下的所有模块**的引用，是通过一个 request 解析出来的正则表达式，去匹配目录下所有符合的模块，然后都 require 进来。此 context module 包含一个 map 对象，会把 request 中所有模块翻译成对应的模块 id。

```json
{
  "./table.ejs": 42,
  "./table-row.ejs": 43,
  "./directory/folder.ejs": 44
}
```

此 context module 还包含一些访问这个 map 对象的 runtime 逻辑。

这意味着 webpack 能够支持动态地 require，但会导致所有可能用到的模块都包含在 bundle 中。

### `require.context`

可以通过 `require.context()` 函数来创建自己的 context。该函数传入三个参数：一个要搜索的目录，一个标记表示是否还搜索其子目录， 以及一个匹配文件的正则表达式。*传递给* `require.context` *的参数必须是字面量*。webpack 会在构建中解析代码中的 `require.context()` 。

```javascript
# 语法
require.context(directory, useSubdirectories = false, regExp = /^\.\//);
# 示例
require.context('./test', false, /\.test\.js$/);
// （创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。

require.context('../', true, /\.stories\.js$/);
// （创建出）一个 context，其中所有文件都来自父文件夹及其所有子级文件夹，request 以 `.stories.js` 结尾。
```

### context module API

一个 context module 会导出一个（require）函数，此函数可以接收一个参数：request。

此导出函数有三个属性：`resolve`, `keys`, `id`。

* `id` 是 context module 里面所包含的模块 id. 它可能在你使用 `module.hot.accept` 时会用到。

- `resolve` 是一个函数，它返回 request 被解析后得到的模块 id。
- `keys` 也是一个函数，它返回一个数组，由所有可能被此 context module 处理的请求（如下面第二段代码中的 key）组成。

如果想引入一个文件夹下面的所有文件，或者引入能匹配一个正则表达式的所有文件，这个功能就会很有帮助，例如：

```javascript
function importAll (r) {
  r.keys().forEach(r);
}

importAll(require.context('../components/', true, /\.js$/));
var cache = {};

function importAll (r) {
  r.keys().forEach(key => cache[key] = r(key));
}

importAll(require.context('../components/', true, /\.js$/));
// 在构建时(build-time)，所有被 require 的模块都会被填充到 cache 对象中。
```

## 公共路径publicpath

`publicPath` 配置选项在各种场景中都非常有用。你可以通过它来指定应用程序中所有资源的基础路径。

实质上，发送到 `output.path` 目录的每个文件，都将从 `output.publicPath` 位置引用。这也包括（通过 [代码分离](https://v4.webpack.docschina.org/guides/code-splitting/) 创建的）子 chunk 和作为依赖图一部分的所有其他资源（例如 image, font 等）。

分为两种情况：

### 基于环境设置

在开发环境中，通常有一个 `assets/` 文件夹，它与索引页面位于同一级别。这没太大问题，但是，如果我们将所有静态资源托管至 CDN，然后想在生产环境中使用呢？想要解决这个问题，可以直接使用一个有着悠久历史的 environment variable(环境变量)。假设我们有一个变量 `ASSET_PATH`：

```js
import webpack from 'webpack';

// 尝试使用环境变量，否则使用根路径
const ASSET_PATH = process.env.ASSET_PATH || '/';

export default {
  output: {
    publicPath: ASSET_PATH
  },

  plugins: [
    // 这可以帮助我们在代码中安全地使用环境变量
    new webpack.DefinePlugin({
      'process.env.ASSET_PATH': JSON.stringify(ASSET_PATH)
    })
  ]
};
```

### 在运行时设置 publicPath

webpack 暴露了一个名为 `__webpack_public_path__` 的全局变量。所以在应用程序的 entry point 中，可以直接如下设置：

```js
__webpack_public_path__ = process.env.ASSET_PATH;
```

由于在配置中使用了 `DefinePlugin`， `process.env.ASSET_PATH` 将始终都被定义， 因此我们可以安全地使用。

> 注意，如果在 entry 文件中使用 ES2015 module import，则会在 import 之后进行 `__webpack_public_path__` 赋值。在这种情况下，你必须将 public path 赋值移至一个专用模块中，然后将它的 import 语句放置到 entry.js 最上面：

```js
// entry.js
import './public-path';
import './app';
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

