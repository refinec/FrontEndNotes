# Webpack配置

### webpack需要掌握的核心概念

`Entry`:webpack开始构建的入口模块

`Output`: 如何命名输出文件，以及输出目录，比如常见的dist目录。

`Loaders`:作用在于解析文件，将无法处理的非js文件，处理成webpack能够处理的模块。

`Plugins`:更多的是优化，提取精华(公共模块去重)，压缩处理(css/js/html)等，对webpack功能的扩展。

`Chunk`: 个人觉得这个是webpack 4 的Code Splitting产物，抛弃了webpack3的`CommonsChunkPlugin`,它最大的特点就是配置简单，当你设置 `mode` 是 `production`，那么 webpack 4 就会自动开启 `Code Splitting`，可以完成将某些公共模块去重，打包成一个单独的`chunk`

### 局部安装webpack

```javascript
 npm install webpack webpack-cli -D   // 局部安装
```

* 查看 webpack 版本号

  `npx webpack -v`

* 查看webpack包版本

  `npm info webpack   // 查看webpack包版本`

### webpack配置文件

> webpack.config.js就是webpack的默认配置文件，我们可以自定义配置文件，比如文件的入口，出口。

```javascript
const path = require('path')
module.exports = {
    entry : './index.js',
    output : {
        filename : 'bundle.js',
        path : path.join(__dirname, 'dist')
    }
}
```

配置完之后：在命令行中运行`npx webpack`，就会去找webpack.config.js文件中的配置信息

* **修改配置文件名称**

  ```javascript
  npx webpack --config webpack.config.js
  // --config 后面就是你自己配置的webpack文件信息
  ```

* **npm scripts**

  在package.json文件中配置scripts命令:

  ```json
  "scripts": {
      "dev": "npx webpack --config webpack.config.js"
  }
  ```

### webpack打包三种命令

- webpack index.js (全局)
- npx webpack index.js
- npm run dev

### 一、webpack核心概念loader

> **loader就是一个打包的方案，它知道对于某个特定的文件该如何去打包**

#### 1. 配置file-loader

> 更多的静态资源的打包配置，可以看官网是如何使用的，👉（[静态loader管理资源](https://juejin.cn/post/6859888538004783118)）

webpack是默认知道如何打包js文件的，但是对于一些，比如**图片**，**字体图标**的模块，webpack就不知道如何打包了,那就去配置文件webpack.config.js配置module：

```javascript
npm install file-loader -D


const path = require('path')
module.exports = {
    mode: 'production',
    entry: './src/index.js',
    // 配置loader
    module: {
        rules: [{
            test: [/\.(png|jpg|gif)$/,/\.(woff|woff2|eot|ttf|otf)$/],
            use: {
                loader: 'file-loader',
                // 配置options，将文件打包名称不改变，并且加个后缀
                options:{
                    // name就是原始名称,hash使用的是MD5算法,ext就是后缀
                    name: '[name]_[hash].[ext]',
                    // 将图片这些模块都打包到dist目录下的images下
                    outputPath:'images/'
                }
            }
        }]
    },
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    }
}
```

#### 2. 配置url-loader

> 对于图片的模块打包，我们同样可以去使用**url-loader**

唯一的区别就在于，要打包的图片是否会打包到images目录下，还是以Base64格式打包到bundle.js文件中，这个就看***limit***配置项了

* 当你打包的图片大小比**limit**配置的参数大，那么跟**file-loader**一样
* 当图片较小时，那么就会以**Base64**打包到**bundle.js**文件中

```javascript
npm install --save-dev url-loader

{
    loader: 'url-loader',
        options: {
            name: '[name]_[hash].[ext]',
            outputPath: 'images/',
            limit : 102400  //100KB
        }
}

```

#### 3. 配置css-loader

> 引入了一个css模块，这个时候，就需要去下载相应的模块loader

- **css-loader**主要作用就是将多个css文件整合到一起，形成一个css文件。
- **style-loader** 会把整合的css部分挂载到head标签中

```javascript
npm install css-loader style-loader -D   // 下载对应的模块

{
    test: /\.css$/,
    use: ['style-loader','css-loader']
}
```

之后，在index.js 导入样式就可以生效：

```javascript
import acator from './头像.jpg'
import './index.css'
const img = new Image()
img.src = acator
img.classList.add('imgtitle')
document.body.appendChild(img)

.imgtitle{
    width: 100px;
    height: 100px;
}
```

#### 4. 配置sass-loader

***注意***：模块的加载就是从右像左来的，所以先加载sass-loader翻译成css文件，然后使用css-loader打包成一个css文件，在通过style-loader挂载到页面上去

```javascript
// 安装sass-loader，需要同时安装node-sass
npm install sass-loader node-sass --save-dev

{
    test: /\.scss$/,
    use: ['style-loader','css-loader','sass-loader']
}
```

接下来又有新的问题了，如果在scss文件中使用css3新特新的话，就需要加上厂商前缀，使用**postcss-loader**

#### 5. 配置postcss-loader

> 这个loader解决的就是加上厂商前缀，我们看webpack官网是怎么做的👉[点这里](https://www.webpackjs.com/loaders/postcss-loader/)

```javascript
npm i -D postcss-loader autoprefixer
```

还需要建一个**postcss.config.js**，这个配置文件(**位置跟webpack.config.js一个位置**)配置如下信息:

```javascript
// postcss.config.js
// 需要配置这个插件信息
module.exports = {
    plugins: [
        require('autoprefixer')({
            overrideBrowserslist: [
                "Android 4.1",
                "iOS 7.1",
                "Chrome > 31",
                "ff > 31",
                "ie >= 8"
            ]
        })
    ]
};
```

重新设置sass-loader,最后就可以加上厂商前缀了

```javascript
{
    test: /\.scss$/,
    use: ['style-loader','css-loader','sass-loader','postcss-loader']
}
```

**(重要)** 有时候，遇到这样子的一个问题，你在某个scss文件中又导入新的scss文件，这个时候，打包的话，它就不会帮你重新安装**postcss-loader**开始打包，这个时候，我们应该如何去设置呢，我们先来看例子:

```scss
// index.scss
@import './creare.scss';
body {
    .imgtitle {
        width: 100px;
        height: 100px;
        transform: translate(100px, 100px);
    }
}
```

所以，需要在css-loader中配置options，加入**importLoaders :2**， 这样子就会走postcss-loader，和sass-loader，这样子的语法，**无论你是在js中引入scss文件，还是在scss中引入scss文件，都会重新依次从下往上执行所以loader**

- `importLoaders:2`该配置信息解决的就是在scss文件中又引入scss文件，会重新从postcss-loader开始打包
- `modules:true`会作用域当前的css环境中，样式不会全局引入，语法上也需要使用如下引入`import style from './index.scss'`

```javascript
{
    test: /\.scss$/,
    use: ['style-loader',
              {
                  loader: 'css-loader',
                  options:{
                      importLoaders:2,
                      modules : true // 希望你的css样式作用的是当前的模块中，而不是全局
                  }
              },
              'sass-loader',
              'postcss-loader'
             ]
}

```

```javascript
// index.js
import acator from './头像.jpg'
import create from './create'

import style from './index.scss'  // 通过modules:true 避免了css作用域create中的模块
const img = new Image()
img.src = acator
img.classList.add(style.imgtitle)
document.body.appendChild(img)

create()

```

create模块是什么呢?这个create模块，就是创建一个img标签，并且设置单独的样式。给`modules : true`后，我们需要接着往下做的就是import语法上有些改变,然后通过style这个对象变量中去找，找到scss中设置的名称即可:

```javascript
import style from './index.scss'
```

### 二、webpack核心概念plugins

> plugins: **可以在webpack运行到某个时刻的时候,帮你做一些事情.**

#### HtmlWebpackPlugin

> **这个插件的作用，就是为你生成一个HTML文件，然后将打包好的js文件自动引入到这个html文件中** [官网](https://www.webpackjs.com/plugins/html-webpack-plugin/)

```javascript
cnpm install --save-dev html-webpack-plugin

// webpack.config.js
var HtmlWebpackPlugin = require('html-webpack-plugin');
var path = require('path');

var webpackConfig = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js'
  },
  plugins: [
      new HtmlWebpackPlugin({
            template: 'src/index.html'  // 以src/目录下的index.html为模板打包
        })
    ],
};

```

#### CleanWebpackPlugin

> 帮你删除某个目录的文件,是在打包前删除所有上一次打包好的文件

```javascript
cnpm i clean-webpack-plugin -D
//"clean-webpack-plugin": "^3.0.0",我的是这个版本

const {CleanWebpackPlugin} = require('clean-webpack-plugin');

// plugins新增加这一项，webpack4版本不需要配置路径
plugins: [ new CleanWebpackPlugin()]

```

配置**clean-webpack-plugin**的话,需要去对于网站上查看如何配置的,可以点这里👉 [npm上](https://www.npmjs.com/package/clean-webpack-plugin)

### webpack核心概念

#### entry和output基本配置

> 有时候,需要多个入口文件,那么我们又是怎么去完成的呢?这个时候,就需要来看一看entry和output配置项.webpack官网也是有文档的,[out点这里](https://www.webpackjs.com/configuration/output/)以及[entry点这里](https://www.webpackjs.com/configuration/entry-context/)

* entry这样子配置就可以接受多个打包的文件入口,同时的话,output输出文件的filename需要使用占位符name
* 这样子就会生成两个文件,不会报错,对于的名字就是entry名称对应
* 如果后台已经将资源挂载到了cdn上,那么你的publicPath就会把路径前做修改加上publicPath值

```javascript
entry: {
        index :'./src/index.js',
        bundle : './src/create.js',
    },
output: {
        filename: '[name].js',
        publicPath: "https://cdn.example.com/assets/",
        path: path.join(__dirname, 'dist')
    }    
```

#### 使用devtool配置source-map

> devtool配置source-map,解决的问题就是,当你代码出现问题时,会映射到你的原文件目录下的错误,并非是打包好的错误,这点也很显然,如果不设置的话,只会显示打包后bundle.js文件中报错,对应查找错误而言,很不利.
>
> [devtool](https://www.webpackjs.com/configuration/devtool/)

```javascript
devtool:'inline-cheap-source-map'
```

- development环境下,配置 `devtool:'cheap-module-eval-source-map'`
- production环境下,配置 `devtool:'cheap-module-source-map'`

#### 使用webpack-dev-server

> 用于快速开发应用程序 [webpack-dev-server](https://www.webpackjs.com/configuration/dev-server/)

**作用：** 可以开启一个服务器,而且可以实时去监听打包文件是否改变,改变的话,就会出现去更新.

**注意：** 使用webpack-dev-server打包的话,不会生成dist目录,而是将**你的文件打包到内存中**

```javascript
cnpm i clean-webpack-plugin -D

devServer: {
        contentBase: path.join(__dirname, "dist"),   // dist目录开启服务器
        compress: true,    // 是否使用gzip压缩
        port: 9000,    // 端口号
        open : true,   // 自动打开网页
         hot: true,   // 开启热更新
        // hotOnly: true,
    },

```

很多的配置项,可以去官方文档查看,比如proxy代理等配置项,更多文档[点击这里](https://www.webpackjs.com/configuration/dev-server/)

然后在package.json中scripts配置项如下：

```javascript
"start": "webpack-dev-server"
```

##### 模块热替换(hot module replacement) [点这里查看HMR](https://www.webpackjs.com/guides/hot-module-replacement/)

> 模块热替换(HMR - Hot Module Replacement)功能会在应用程序运行过程中替换、添加或删除[模块](https://www.webpackjs.com/concepts/modules/)，而无需重新加载整个页面.顾名思义它说的就是,多个模块之前,当你修改一个模块,而不想重新加载整个页面时,就可以使用`hot module replacement`

然后加入两个插件,这个插件时webpack自带的,所以不需要下载:

```javascript
const webpack = require('webpack')
plugins: [
        new webpack.NamedModulesPlugin(),  // 可配置也可不配置,添加了 NamedModulesPlugin，以便更容易查看要修补(patch)的依赖
        new webpack.HotModuleReplacementPlugin() // 这个是必须配置的插件
    ],

```

**注意：** 对于css的内容修改,css-loader底层会帮我们做好实时热更新,对于JS模块的话,我们需要手动的去配置

```javascript
// module.hot.accept(module1,callback) 表示的就是接受一个需要实时热更新的模块,当内容发生变化时,会帮你检测到,然后执行回调函数
if(module.hot){
    module.hot.accept('./print',()=>{
        print()
    })
}
```

#### Babel处理ES6语法

```javascript
npm install --save-dev babel-loader @babel/core
// @babel/core 是babel中的一个核心库

npm install --save-dev @babel/preset-env
// preset-env 这个模块就是将语法翻译成es5语法,这个模块包括了所有翻译成es5语法规则
// 有了preset-env这个模块后,我们就会发现我们写的const语法被翻译成成var

npm install --save @babel/polyfill
// 将Promise,map等低版本中没有实现的语法,用polyfill来实现.

```

```javascript
module: {
  rules: [
    {
            test: /\.js$/,
   // exclude参数: node_modules目录下的js文件不需要做转es5语法,也就是排除一些目录
            exclude: /node_modules/,
            loader: "babel-loader",
            options: {
                "presets": [
                    [
                        "@babel/preset-env",
                        {
// "useBuiltIns"参数: 只会对我们index.js当前要打包的文件中使用过的语法,比如Promise,map做polyfill,其他es6未出现的语法,我们暂时不去做polyfill,这样子,打包后的文件就减少体积了
                            "useBuiltIns": "usage"
                        }
                    ]
                ]
            }
        }
  ]
}

```

需要`@babel/polyfill`模块,对Promise,map进行补充,完成该功能,在js文件最开始导入:

```javascript
import "@babel/polyfill";
```

但是用完这个以后,因为`@babel/polyfill`为了弥补Promise,map等语法的功能,该模块就需要**自己去实现Promise,map等语法**的功能,打包的文件体积瞬间增加了10多倍之多对`@babel/polyfill`参数做一些配置即可:`"useBuiltIns": "usage"` 。配置看官方文档,[点这里](https://www.babeljs.cn/)



**(重要)**当你生成第三方模块时,或者是生成UI组件库时,使用polyfill解决问题,就会出现问题了,上面的场景使用babel会污染环境,这个时候,我们需要换一种方案来解决：

**@babel/plugin-transform-runtime** 这个库就能解决我们的问题,它作用是将 helper 和 polyfill 都改为从一个统一的地方引入，并且引入的对象和全局变量是完全隔离的,避免了全局的污染。那我们先安装需要的库

```javascript
npm install --save-dev @babel/plugin-transform-runtime
npm install --save @babel/runtime
```

这个时候可以在根目录下建一个`.babelrc`文件,将原本要在options中的配置信息写在`.babelrc`文件:

```javascript
{
    
    "plugins": [
      [
        "@babel/plugin-transform-runtime",
        {
          "corejs": 2,
          "helpers": true,
          "regenerator": true,
          "useESModules": false
        }
      ]
    ]
  }

// 当你的 "corejs": 2,需要安装下面这个
npm install --save @babel/runtime-corejs2

```

这样子的话,在使用语法时,就不需要去通过`import "@babel/polyfill";`去完成了,直接正常写就行了,而且从打包的体积来看,其实可以接受的。

### webpack高级概念



#### 使用tree shaking

> tree树，shaking摇动，那么你可以把程序想成一颗树。绿色表示实际用到的源码和 library，是树上活的树叶。灰色表示无用的代码，是秋天树上枯萎的树叶。为了除去死去的树叶，你必须摇动这棵树，使它们落下.
>
> 通俗意义而言，当你引入一个模块时，你可能用到的只是其中的某些功能，这个时候，我们不希望这些`无用`的代码打包到项目中去。通过tree-shaking，就能将没有使用的模块摇掉，这样达到了删除无用代码的目的

对于原理篇，可以看看这篇[Tree-Shaking性能优化实践 - 原理篇](https://juejin.im/post/6844903544756109319)

**webpack4默认的production下是会进行tree-shaking的**

`optimization.usedExports`使webpack确定每个模块导出项（exports）的使用情况。依赖于`optimization.providedExports`的配置。`optimization.usedExports`收集到的信息会被其他优化项或产出代码使用到（模块未用到的导出项不会被导出，在语法完全兼容的情况下会把导出名称混淆为单个char）。为了最小化代码体积，未用到的的导出项目（exports）会被删除。生产环境(production)默认开启.

```javascript
module.exports = {
  //...
  optimization: {
    usedExports: true
  }
};
```

##### **将文件标记为无副作用(side-effect-free)**

> 有时候，当我们的模块不是达到很纯粹，这个时候，webpack就无法识别出哪些代码需要删除，所以，此时有必要向 webpack 的 compiler 提供提示哪些代码是“纯粹部分”

通过 package.json 的 `"sideEffects"` 属性来实现,如果所有代码都不包含副作用，我们就可以简单地将该属性标记为 `false`，来告知 webpack，它可以安全地删除未用到的 export 导出:

```javascript
{
  "name": "webpack-demo",
  "sideEffects": false
}

```

**注意：** *任何导入的文件都会受到 tree shaking 的影响。这意味着，如果在项目中使用类似* `css-loader` *并导入 CSS 文件，则需要将其添加到 side effect 列表中，以免在生产模式中无意中将它删除*

```javascript
{
  "name": "webpack-demo",
  "sideEffects": [
    "*.css"
  ]
}
```

##### **压缩输出**

> 通过如上方式，我们已经可以通过 `import` 和 `export` 语法，找出那些需要删除的“未使用代码(dead code)”，然而，我们不只是要找出，还需要在 bundle 中删除它们。为此，我们将使用 `-p`(production) 这个 webpack 编译标记，来启用 **uglifyjs** 压缩插件

**从 webpack 4 开始，也可以通过 `"mode"` 配置选项轻松切换到压缩输出，只需设置为 `"production"`**

##### 总结

* 为了使用tree-shaking的话，需要使用ES Module语法，也就是使用 ES2015 模块语法（即 `import` 和 `export`）

* 在项目 `package.json` 文件中，添加一个 "sideEffects" 入口

* 引入一个能够删除未引用代码(dead code)的压缩工具(minifier)（例如 `UglifyJSPlugin`），当然了，webpack4开始，以及支持压缩输出了

#### *development*和*production*环境构建

##### 使用webpack-merge配置公共代码

```javascript
cnpm install --save-dev webpack-merge
```

目录结构

```javascript
 webpack-demo
  |- build
    |- webpack.common.js  //三个新webpack配置文件
    |- webpack.dev.js    //三个新webpack配置文件
    |- webpack.prod.js  //三个新webpack配置文件
  |- package.json
  |-postcss.config.js
  |-.babelrc
  |- /dist
  |- /src
    |- index.js
    |- math.js
  |- /node_modules
```

```javascript
// webpack.common.js 公共代码

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const commonConfig = {
    entry: {
        main: './src/index.js',
    },
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader"
        }, {
            test: /\.(jpg|gif|png)$/,
            use: {
                loader: 'url-loader',
                options: {
                    name: '[name]_[hash].[ext]',
                    outputPath: 'images/',
                    limit: 1024 //100KB
                }
            }
        }, {
            test: /\.css$/,
            use: ['style-loader', 'css-loader', 'postcss-loader']
        }, {
            test: /\.scss$/,
            use: ['style-loader',
                {
                    loader: 'css-loader',
                    options: {
                        importLoaders: 2,
                        modules: true
                    }
                },
                'sass-loader',
                'postcss-loader'
            ]
        }, {
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            use: [
                'file-loader'
            ]
        }]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: 'src/index.html' // 以src/目录下的index.html为模板打包
        }),
        new CleanWebpackPlugin({
            // 不需要做任何的配置
        }),
    ],
    output: {
        filename: '[name].js',
        // publicPath: "https://cdn.example.com/assets/",
        path: path.join(__dirname, '../dist')
    }
}

module.exports = commonConfig
```

```javascript
// webpack.dev.js 开发环境

const path = require('path')
const webpack = require('webpack')
const {merge} = require('webpack-merge')
const commonConfig = require('./webpack.common')

const devConfig = {
    mode: 'development',
    devtool: 'cheap-module-eval-source-map',
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        compress: true,
        port: 9000,
        open: true,
        hot: true,
        // hotOnly: true,
    },
    plugins: [
        new webpack.NamedModulesPlugin(),
        new webpack.HotModuleReplacementPlugin(),
    ],
    optimization:{
        usedExports: true
    }
}

module.exports = merge(commonConfig, devConfig)
```

```javascript
// webpack.prod.js 生产环境

const {merge} = require('webpack-merge')
const commomConfig = require('./webpack.common')
const prodConfig = {
    mode: 'production',
    devtool: 'cheap-module-source-map',
}

module.exports = merge(commomConfig, prodConfig)
```

##### NPM Scripts

把 `scripts` 重新指向到新配置。我们将 `npm run dev` 定义为*开发环境*脚本，并在其中使用 `webpack-dev-server`，将 `npm run build` 定义为*生产环境*脚本：

```javascript
{
    "name": "webpack-demo",
        "scripts": {
            "dev": "webpack-dev-server --config ./build/webpack.dev.js",
            "build": "webpack --config ./build/webpack.prod.js",
            "start": "npx webpack --config ./build/webpack.dev.js"
        },
}
```

#### SplitChunksPlugin代码分隔

> 当你有多个入口文件，或者是打包文件需要做一个划分，举个例子，比如第三方库lodash，jquery等库需要打包到一个目录下，自己的业务逻辑代码需要打包到一个文件下，这个时候，就需要提取公共模块了，也就需要SplitChunksPlugin这个插件,配置**optimization.splitChunks**

```javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
        chunks: "async",
        minSize: 30000,
        minChunks: 1,
        maxAsyncRequests: 5,
        maxInitialRequests: 3,
        automaticNameDelimiter: '~',
        name: true,
        cacheGroups: {
            vendors: { 
                test: /[\\/]node_modules[\\/]/,  //匹配node_modules中的模块
                priority: -10  //优先级,当模块同时命中多个缓存组的规则时，分配到优先级高的缓存组
            },
        default: {
                minChunks: 2, //覆盖外层的全局属性
                priority: -20,
                reuseExistingChunk: true  //是否复用已经从原代码块中分割出来的模块
            }
        }
    }
  },
};
```

**参数讲解：**

在**cacheGroups**外层的属性设置适用于所有的缓存组，不过每个缓存组内部都可以**重新**设置它们的值

`	chunks: "async"` 这个属性设置的是以**什么类型**的代码经行分隔，有三个值

- `initial` 入口代码块
- `all` 全部
- `async` 按需加载的代码块

`minSize: 30000` 模块大小超过30kb的模块才会提取

`minChunks: 1`, 当某个模块至少被多少个模块引用时，才会被提取成新的chunk

`maxAsyncRequests: 5`,分割后，按需加载的代码块最多允许的并行请求数

`maxInitialRequests: 3·` 分割后，入口代码块最多允许的并行请求数

`automaticNameDelimiter: "~"`代码块命名分割符

`name: true,  `每个缓存组打包得到的代码块的名称

`cacheGroups` 缓存组，定制相应的规则。

自己根据实际情况去设置相应的规则，每个缓存组根据规则将匹配的模块会分配到代码块（chunk）中，每个缓存组的打包结果可以是单一 chunk，也可以是多个 chunk。实际项目中如何代码分隔：[SplitChunk代码实例](https://juejin.im/post/6844904183917871117#comment)

#### Lazy-loding懒加载和Chunk

##### import异步加载模块

> 当我需要按需引入某个模块时，这个时候，我们就可以使用懒加载，其实实现的方案就是import语法，在达到某个条件时，我们才会去请求资源

1. 借助插件，完成对import语法的识别

   `cnpm install --save-dev @babel/plugin-syntax-dynamic-import`

2. 然后在`.babelrc`文件下配置，增加一个插件

   ```javascript
   {
     "plugins": ["@babel/plugin-syntax-dynamic-import"]
   }
   ```

3. 使用import按需加载模块

   ```javascript
   // create.js
   async function create() {
       const {
           default: _
       } = await import(/*webpackChunkName:"lodash"*/'lodash')
       let element = document.createElement('div')
       element.innerHTML = _.join(['TianTian', 'lee'], '-')
       return element
   }
   
   function demo() {
       document.addEventListener('click', function () {
           create().then(element => {
               document.body.appendChild(element)
           })
       })
   }
   
   export default demo;
   
   //当你点击页面后，会触发create函数，然后加载loadsh库，最后再页面中懒加载lodash，打包是正常打包，但是呢，有些资源，可以当你触发某些条件，再去加载
   ```

##### Chunk

Chunk是Webpack打包过程中，一堆module的集合。Webpack通过引用关系逐个打包模块，这些module就形成了一个Chunk

**产生Chunk的三种途径**: 

- entry入口
- 异步加载模块
- 代码分割（code spliting）

#### CSS代码压缩提取

> 在线上的环境中，我们需要去将我们的CSS文件单独的打包到一个Chunk下，所以需要借助一定的插件，完成这个工作

css代码提取[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)

将css提取为独立的文件插件，支持按需加载的css和sourceMap,我们可以查看GitHub官方，来看看它的[文档](https://github.com/webpack-contrib/mini-css-extract-plugin)

**目前缺失功能，HMR.所以，可以把它运用到生产环境**

`npm install --save-dev mini-css-extract-plugin`

需要注意的一点是，当你的webpack版本是4版本的时候，需要去package.json中配置`sideEffects`属性，这样子就**避免了把css文件作为Tree-shaking**

```json
{
  "name": "webpack-demo",
  "sideEffects": [
  	"*.css"
  ]
}
```

```javascript
// webpack.prod.js

const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const {
    merge
} = require('webpack-merge')
const commomConfig = require('./webpack.common')

const prodConfig = {
    mode: 'production',
    devtool: 'cheap-module-source-map',
    plugins: [
        new MiniCssExtractPlugin({
            filename:'[name].[hash].css',
            chunkFilename: '[id].[hash].css',
        })
    ],
    module: {
        rules: [{
            test: /\.(sa|sc|c)ss$/,
            use: [
                MiniCssExtractPlugin.loader,
                'css-loader',
                'postcss-loader',
                'sass-loader',
            ],
        }]
    }
}

module.exports = merge(commomConfig, prodConfig)

```

当你在js中引入css模块时，最后在dist目录下，看到了css单独的Chunk的话，说明css代码提取成功了，接下来就是对**css代码的压缩**

**注意：**webpack4默认在生产环境下，是不会去压缩css代码的，所以我们需要下载对于的plugin

#### [optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin) css代码压缩会对打包后的css代码经行代码压缩

`npm install --save-dev optimize-css-assets-webpack-plugin`

接下来就是设置 **optimization.minimizer** ，这里需要注意的就是，此时设置optimization.minimizer会覆盖webpack默认提供的规则，比如**JS代码就不会再去压缩了，所以需要uglifyjs-webpack-plugin插件解决**

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const {
    merge
} = require('webpack-merge')
const commomConfig = require('./webpack.common')

const prodConfig = {
    mode: 'production',
    devtool: 'cheap-module-source-map',
    optimization: {
        minimizer: [
            new UglifyJsPlugin({
                sourceMap: true,
                parallel: true, // 启用多线程并行运行提高编译速度
            }),
            new OptimizeCSSAssetsPlugin({}),
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            // 类似 webpackOptions.output里面的配置 可以忽略
            filename: '[name].[hash].css',
            chunkFilename: '[id].[hash].css'
        })
    ],
    module: {
        rules: [{
            test: /\.(sa|sc|c)ss$/,
            use: [{
                    loader: MiniCssExtractPlugin.loader,
                    options: {
                        // 这里可以指定一个 publicPath
                        // 默认使用 webpackOptions.output中的publicPathcss
                        // 举个例子,后台支持把css代码块放入cdn
                        publicPath: "https://cdn.example.com/css/"
                    },
                },
                'css-loader',
                'postcss-loader',
                'sass-loader',
            ],
        }]
    },

}

module.exports = merge(commomConfig, prodConfig)

```

**uglifyjs-webpack-plugin代码压缩**

> 这个插件解决的问题，就是当你需要去optimization.minimizer中设置，这样子会覆盖**webpack基本配置**，原本JS代码压缩的功能就会被覆盖，所以我们需要下载它

`npm install -D uglifyjs-webpack-plugin` 更多配置看[官网文档](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)

```javascript
// webpack.prod.js  
// 对于开发者环境而言，对css代码提取，以及打包是没有意义的，统一对于js代码压缩，也会降低效率，也是不推荐这么去做

const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
const {
    merge
} = require('webpack-merge')
const commonConfig = require('./webpack.common')

const prodConfig = {
    mode: 'production',
    devtool: 'cheap-module-source-map',
    optimization: {
        minimizer: [
            new UglifyJsPlugin({
                sourceMap: true,
                parallel: true,  // 启用多线程并行运行提高编译速度
            }),
            new OptimizeCSSAssetsPlugin({}),
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            // 类似 webpackOptions.output里面的配置 可以忽略
            filename: '[name].[hash].css',
            chunkFilename: '[id].[hash].css'
        })
    ],
    module: {
        rules: [{
            test: /\.(sa|sc|c)ss$/,
            use: [{
                    loader: MiniCssExtractPlugin.loader,
                    options: {
                        // 这里可以指定一个 publicPath
                        // 默认使用 webpackOptions.output中的publicPathcss
                        // 举个例子,后台支持把css代码块放入cdn
                        publicPath: "https://cdn.example.com/css/"
                    },
                },
                'css-loader',
                'postcss-loader',
                'sass-loader',
            ],
        }]
    },

}

module.exports = merge(commonConfig, prodConfig)

```

### contenthash解决浏览器缓存

> 当你打包一个项目即将上线时，有一个需求，你只是修改了部分的文件，只希望用户对于其他的文件，依旧去采用浏览器缓存中的文件，所以这个时候，我们需要用到**contenthash**

**webpack中关于hash，有三种**:

 1. **hash**

    hash，主要用于开发环境中，在构建的过程中，当你的项目有一个文件发现了改变，整个项目的hash值就会做修改(整个项目的hash值是一样的)，这样子，每次更新，文件都不会让浏览器缓存文件，保证了文件的更新率，提高开发效率

 2. **chunkhash**

    跟打包的chunk有关，具体来说`webpack`是根据入口`entry`配置文件来分析其依赖项并由此来构建该`entry的chunk`，并生成对应的`hash`值。不同的`chunk`会有不同的`hash`值

    在生产环境中，我们会把第三方或者公用类库进行单独打包，所以不改动公共库的代码，该`chunk`的`hash`就不会变，可以合理的使用浏览器缓存了

    但是这个中hash的方法其实是存在问题的，生产环境中我们会用`webpack`的插件，将`css`代码打单独提取出来打包。这时候`chunkhash`的方式就不够灵活，因为只要同一个`chunk`里面的js修改后，`css`的`chunk`的`hash`也会跟随着改动。因此我们需要`contenthash`

 3. **contenthash**

    `contenthash`表示由文件内容产生的`hash`值，内容不同产生的`contenthash`值也不一样。生产环境中，通常做法是把项目中`css`都抽离出对应的`css`文件来加以引用

    对于webpack，旧版本而言，即便每次你npm run build，**内容不做修改的话，contenthash值还是会有所改变**，这个是因为，当你在模块之间存在相互之间的引用关系，有一个**manifest文件**

> manifest文件是用来引导所以模块的交互，manifest文件包含了加载和处理模块的逻辑，举个例子，你的**第三方库打包后的文件，我们称之为vendors**，你的逻辑代码称为**main**，当你webpack生成一个bundle时，它同时会去维护一个manifest文件，你可以理解成每个bundle文件存在这里信息，所以每个bundle之间的manifest信息有不同，这样子我们就需要将manifest文件给提取出来。

这个时候，需要在**optimization**中增加一个配置:

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      // ...
    },
    runtimeChunk: {// 解决的问题是老版本中内容不发生改变的话,contenthash依旧会发生改变
      name: 'manifest'
    }
  }
}
```

```javascript
// webpack.prod.js

output: {
    filename: '[name].[contenthash].js',
    chunkFilename:'[vendors].[contenthash].js',
    // publicPath: "https://cdn.example.com/assets/",
    path: path.join(__dirname, '../dist')
}
```

在**webpack.dev.js**中只需要将contenthash改为hash就行

### (重要)shimming 全局变量(ProvidePlugin)

> 简单翻译就是`垫片`，它解决的场景有哪些呢，举个例子，当你再使用第三方库，此时需要引入它，又或者是你有很多的第三方库或者是自己写的库，每个js文件都需要依赖它，让人很繁琐，这个时候，shimming就派上用场

使用 `ProvidePlugin` 插件，这个webpack是内置的，shimming依赖的就是这个插件

使用 [`ProvidePlugin`](https://www.webpackjs.com/plugins/provide-plugin) 后，能够在通过 webpack 编译的每个模块中，通过访问一个变量来获取到 package 包

增加一个Plugin配置:

```javascript
new webpack.ProvidePlugin({
			// 这里设置的就是你相应的规则了
			// 等价于在你使用lodash模块中语句👇
			// import _ from 'lodash'
            _: 'lodash'
})
```

```javascript
// array_add.js
export const Arr_add = arr=>{
    let str = _.join(arr,'++');
    return str;
}

```

这样子没有正常导入lodash库的话，是会报错的，但是我们使用了ProvidePlugin插件，使得它会提供相应的lodash包，注意到的就是，避免多个lodash包被打包多次，可以使用**CommonsChunkPlugin**插件，webpack4已经抛弃它了，使用的是**splitChunksPlugin**插件取代它