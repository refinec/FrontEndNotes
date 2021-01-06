# Webpack概念

## 配置

```javascript
# webpack.config.js 

const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  # 模式
  mode: 'production', // development, production 或 none 之中的一个
  # 入口
  entry: './path/to/my/entry/file.js',
  # 出口
  output:{
    path: path.resolve(__dirname, 'dist'),
    filename:'xx.bundle.js'
  },
  module:{
      # 添加loader
      rules:[
          { test: /\.txt$/, use: 'raw-loader' }
      ]
  },
  # 插件
  plugins:[
      // html-webpack-plugin 为应用程序生成 HTML 一个文件，并自动注入所有生成的 bundle。
      new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

## 入口entry

项目的打包入口，默认值是 `./src/index.js`，可指定一个（或多个）不同的入口起点。

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

   

2. 内联：在每个 `import` 语句中显式指定 loader。

   

3. CLI：在 shell 命令中指定它们。

## 插件plugin

>  loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

**webpack 提供许多开箱可用的插件 https://v4.webpack.docschina.org/plugins **

想要使用一个插件，只需要 `require()` 它，然后把它添加到 `plugins` 数组中。

多数插件可以通过选项(option)自定义。也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建它的一个实例。

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

## 浏览器兼容性browser compatibility

webpack 支持所有符合 [ES5 标准](https://kangax.github.io/compat-table/es5/) 的浏览器（不支持 IE8 及以下版本）。webpack 的 `import()` 和 `require.ensure()` 需要 `Promise`。如果想要支持旧版本浏览器，在使用这些表达式之前，还需要 [提前加载 polyfill](https://v4.webpack.docschina.org/guides/shimming/)。

