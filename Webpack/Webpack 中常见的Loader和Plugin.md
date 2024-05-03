## Loader

默认情况下，在遇到`import`获`require`加载模块时，webpack只支持对`js`和`json`文件打包

`loader`使用方式如下：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/, // 文件解析规则
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
}
```

常见的`loader`如下：

* `style-loader`

  将 `css` 添加到DOM的内敛样式标签`style`里

* `css-loader`

  分析`css`模块之间的关系，并合成一个`css`

* `less-loader` 或 `sass-loader`

  处理`less` 或 `sass`

* `postcss-loader`

  用`postcss`来处理`css`

* `file-loader`

  分发文件到`output`目录并返回该文件相对路径

* `url-loader`

  和`file-loader`类似，但是当文件小于设定的`limit`时可以返回一个`base64`格式`Data Url`内嵌到代码中

* `html-minify-loader`

  压缩HTML

* `babel-loader`

  用`babel`来转换ES6语法到ES5

* `image-webpack-loader`

  图片大小压缩

## Plugin

> 其本质是一个具有`apply`方法的js对象，`apply`方法会被`webpack compiler`调用，并且在整个生命周期都可以访问`compiler`对象
>
> ```js
> const PluginName = 'ConsoleLogOnBuildWebpackPlugin';
> 
> class ConsoleLogOnBuildWebpackPlugin {
>   apply(compiler) {
>     compiler.hooks.run.tap(PluginName, (compilation) => {
>       console.log('webpack 构建过程开始!')
>     })
>   }
> }
> module.exports = ConsoleLogOnBuildWebpackPlugin;
> ```

通过配置文件导出对象中`plugins`属性传入`new`实例对象。

```json
const webpack = require('webpack');
module.exports = {
  plugins: [
    new webpack.ProgressPlugin()
  ]
}
```

常见的`plugin`如下：

* `html-webpack-plugin`

  打包结束后，生成一个`html`文件，并把打包生成的`js`模块引入到该`html`文件中

* `clean-webpack-plugin`

  清除构建目录

* `mini-css-extract-plugin`

  提取`css`到一个单独的文件中

* `DefinePlugin`

  创建在编译的时候需要用到的环境变量

  ```js
  const { DefinePlugin } = require('webpack');
  
  module.exports = {
    plugins: [
      new DefinePlugin({ BASE_URL: './' })
    ]
  }
  ```

  ```html
  <link href="<%=BASE_URL%>favicon.ico>"
  ```

* `copy-webpack-plugin`

  复制文件或目录到执行区域，如`vue`的打包过程中，如果我们将一些文件放到`public`的目录下，那么这个目录会被复制到`dist`文件夹中

* `compression-webpack-plugin`

  文件大小压缩

* `purgecss-plugin-webpack`

  `css`的`tree shaking`