通过`webpack`优化前端的手段有：

* `JS`代码压缩

* `CSS`代码压缩

  使用`css-minimizer-webpack-plugin`

* `HTML`文件代码压缩

  使用`html-webpack-plugin`

* 文件大小压缩

  使用`compression-webpack-plugin`

* 图片压缩

  使用`image-webpack-loader`

* `Tree Shaking`

  * `JS`

    在`package.json`中设置`sideEffects`属性，值为`false`表示`webpack`可以安全的删除未用到的`exports`。

    如果有些文件需要保留，可以设置为数组的形式：

    ```json
    "sideEffects": [
      "./src/util/format.js",
      "*.css" // 所有css文件
    ]
    ```

  * `CSS`

    使用`purgecss-webpack-plugin`

* 代码分离

  使用内置的`splitChunksPlugin`来分离出更小的`bundle`，以及控制资源加载的优先级，提供代码的加载性能

* 内联 `chunk`

  通过`InlineChunkHtmlPlugin`插件将一些`chunk`模块内联到`html`，如`runtime`的代码（对模块进行解析、加载、模块信息相关的代码），代码量不大，但必须加载

  ```js
  const InlineChunkHtmlPlugin = require("react-dev-utils/InlineChunkHtmlPlugin");
  const HtmlWebpackPlugin = require("html-webpack-plugin");
  module.exports = {
      plugins: [
          new InlineChunkHtmlPlugin(HtmlWebpackPlugin, [/runtime-.+[.]js/]),
      ]
  }
  ```

  