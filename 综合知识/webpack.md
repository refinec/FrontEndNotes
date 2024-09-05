## 1. webpack与grunt、gulp的不同？

* [grunt](https://link.zhihu.com/?target=https%3A//www.gruntjs.net/)和[gulp](https://link.zhihu.com/?target=https%3A//www.gulpjs.com.cn/)是基于**任务和流**（Task、Stream）的。类似jQuery，找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程。

* **webpack**是基于**入口**的。webpack会自动地**递归解析入口所需要加载的所有资源文件**，然后**用不同的Loader来处理不同的文件，用Plugin来扩展webpack功能**。

- 从构建思路来说

  gulp和grunt需要开发者将整个前端构建过程拆分成多个`Task`，并合理控制所有`Task`的调用关系
  webpack需要开发者找到入口，并需要清楚对于不同的资源应该使用什么Loader做何种解析和加工

- 对于知识背景来说
  gulp更像后端开发者的思路，需要对于整个流程了如指掌 webpack更倾向于前端开发者的思路

## 2. 与webpack类似的工具还有哪些？谈谈你为什么最终选择（或放弃）使用webpack？

同样是基于入口的打包工具还有以下几个主流的：

- webpack
- [rollup](https://link.zhihu.com/?target=https%3A//rollupjs.org/)
- [parcel](https://link.zhihu.com/?target=https%3A//parceljs.org/)

**从应用场景上来看：**

- webpack适用于大型复杂的前端站点构建
- rollup适用于基础库的打包，如vue、react
- parcel适用于简单的实验性项目，他可以满足低门槛的快速看到效果

> 由于parcel在打包过程中给出的调试信息十分有限，所以一旦打包出错难以调试，所以不建议复杂的项目使用parcel

## 3. 有哪些常见的Loader？他们是解决什么问题的？

2. **`file-loader`**：把文件输出到一个文件夹中，在代码中通过**相对 URL** 去引用输出的文件 (处理图片和字体)
3. **`url-loader`**：和 file-loader 类似，区别是用户可以**设置一个阈值在文件很小的情况下以 base64 的方式把文件内容注入到代码中**去
3. **`babel-loader`**：把 ES6 转换成 ES5
4. **`ts-loader`**: 将 TypeScript 转换成 JavaScript
5. **`awesome-typescript-loader`**:将 TypeScript 转换成 JavaScript，性能优于 ts-loader
6. **`source-map-loader`**：加载额外的 Source Map 文件，以方便断点调试
7. **`style-loader`**：**把 CSS 代码注入到 JavaScript 中**，通过 DOM 操作去加载 CSS。
8. **`css-loader`**：**加载 CSS**，支持**模块化、压缩、文件导入**等特性
9. **`sass-loader`**:将SCSS/SASS代码转换成CSS
10. **`postcss-loader`**:扩展 CSS 语法，使用下一代 CSS，可以配合 autoprefixer 插件自动补齐 CSS3 前缀
11. **`image-loader`**：加载并且压缩图片文件
12. `svg-inline-loader`:将压缩后的 SVG 内容注入代码中
13. `json-loader`:加载 JSON 文件（默认包含）
14. `raw-loader`:加载文件原始内容（utf-8）
15. `eslint-loader`：通过 ESLint 检查 JavaScript 代码
16. `tslint-loader`: 通过 TSLint检查 TypeScript 代码
17. `mocha-loader`: 加载 Mocha 测试用例的代码
18. `coverjs-loader`:计算测试的覆盖率
19. `vue-loader`:加载 Vue.js 单文件组件
20. `handlebars-loader`: 将 Handlebars 模版编译成函数并返回
21. **`i18n-loader`**:国际化
22. **`cache-loader`**:可以在一些性能开销较大的 Loader 之前添加，目的是将结果缓存到磁盘里

## 4.有哪些常见的Plugin？他们是解决什么问题的？

2. `ignore-plugin` :忽略部分文件
3. **`html-webpack-plugin`**:简化 HTML 文件创建 (依赖于 html-loader)
4. **`web-webpack-plugin`**:可方便地为单页应用输出 HTML，比 html-webpack-plugin 好用
5. **`commons-chunk-plugin`**：提取公共代码
5. **`clean-webpack-plugin`**: 目录清理
6. **`terser-webpack-plugin`**:支持压缩 ES6 (Webpack4)
7. `uglifyjs-webpack-plugin`：通过`UglifyES`压缩`ES6`代码
8. **`webpack-parallel-uglify-plugin`**: **多进程执行代码压缩，提升构建速度**
9. **`mini-css-extract-plugin`**:分离样式文件，CSS 提取为独立文件，支持按需加载 (替代extract-text-webpack-plugin)
10. `serviceworker-webpack-plugin`:为网页应用增加离线缓存功能
11. `define-plugin`：定义环境变量 (Webpack4 之后指定 mode 会自动配置)
12. `ModuleConcatenationPlugin`: 开启 Scope Hoisting
13. **`webpack-bundle-analyzer`**: 可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)
14. `speed-measure-webpack-plugin`:  可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)
15. `webpack-dashboard`：可以更友好的展示相关打包信息。
16. **`webpack-merge`**：提取公共配置，减少重复配置代码
17. `size-plugin`：监控资源体积变化，尽早发现问题
18. `HotModuleReplacementPlugin`：模块热替换

## 5.Loader和Plugin的不同？

**不同的作用**

- **Loader**直译为"加载器"。Webpack将一切文件视为模块，但是webpack原生是只能解析js文件，如果想将其他文件也打包的话，就会用到`loader`。 所以**Loader的作用是让webpack拥有了加载和解析*非JavaScript文件*的能力 **。(本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作)
- **Plugin**直译为"插件"。Plugin可以**扩展webpack的功能**，让webpack具有更多的灵活性。 **在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果**。

**不同的用法**

- **Loader**在**`module.rules`**中配置，也就是说他作为模块的解析规则而存在。 **类型为数组，每一项都是一个`Object` **，里面描述了对于什么类型的文件（`test`），使用什么加载(`loader`)和使用的参数（`options`）
- **Plugin**在**`plugins`**中单独配置。 **类型为数组，每一项是一个`plugin`的实例 **，参数都通过构造函数传入。

## 6.webpack的构建流程是什么?从读取配置到输出文件这个过程尽量说全

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. `初始化参数`：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
2. `开始编译`：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
3. `确定入口`：根据配置中的 entry 找出所有的入口文件；
4. `编译模块`：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
5. `完成模块编译`：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
6. `输出资源`：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
7. `输出完成`：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

简单说

- 初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler
- 编译：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理
- 输出：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中

## **source map是什么？生产环境怎么用？**

`source map` 是**将编译、打包、压缩后的代码映射回源代码的过程 **。打包压缩后的代码不具备良好的可读性，想要调试源码就需要 soucre map。

map文件只要不打开开发者工具，浏览器是不会加载的。

`hidden-source-map`：借助**第三方错误监控平台 Sentry 使用**

`nosources-source-map`：只会显示具体行数以及查看源代码的错误栈。安全性比 sourcemap 高

`sourcemap`：通过 nginx 设置将 .map 文件只对白名单开放(公司内网)

注意：避免在生产中使用 `inline-` 和 `eval-`，因为它们会增加 bundle 体积大小，并降低整体性能。。

## **模块打包原理知道吗？**

Webpack 实际上为每个模块创造了一个可以导出和导入的环境，本质上并没有修改 代码的执行逻辑，代码执行顺序与模块加载顺序也完全一致。

## **文件监听原理**

在发现源码发生变化时，自动重新构建出新的输出文件。

Webpack开启监听模式，有两种方式：

- 启动 webpack 命令时，带上 --watch 参数
- 在配置 webpack.config.js 中设置 watch:true

缺点：每次需要手动刷新浏览器

原理：轮询判断文件的最后编辑时间是否变化，如果某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来，等 `aggregateTimeout` 后再执行。

```javascript
module.export = {    
    // 默认false,也就是不开启    
    watch: true,    
    // 只有开启监听模式时，watchOptions才有意义    
    watchOptions: {        
        // 默认为空，不监听的文件或者文件夹，支持正则匹配        
        ignored: /node_modules/,        
        // 监听到变化发生后会等300ms再去执行，默认300ms        
        aggregateTimeout:300,        
        // 判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次
        poll:1000   
    }}
```

## **如何对bundle体积进行监控和分析？**

`VSCode` 中有一个插件 `Import Cost` 可以帮助我们对引入模块的大小进行实时监测，还可以使用 `webpack-bundle-analyzer` 生成 `bundle` 的模块组成图，显示所占体积。

`bundlesize` 工具包可以进行自动化资源体积监控。

## **文件指纹是什么？怎么用？**

文件指纹是**打包后输出的文件名的后缀 **。

**`Hash`**：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改

**`Chunkhash`**：和 Webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash

**`Contenthash`**：**根据文件内容来定义 hash**，文件内容不变，则 contenthash 不变

```javascript
// JS的文件指纹设置,设置 output 的 filename，用 chunkhash:
module.exports = {    
    entry: {        
        app: './scr/app.js',        
        search: './src/search.js'    
    },    
    output: {        
        filename: '[name][chunkhash:8].js',        
        path:__dirname + '/dist'    
    }}

// CSS的文件指纹设置.设置 MiniCssExtractPlugin 的 filename，使用 contenthash
module.exports = {    
    entry: {        
        app: './scr/app.js',        
        search: './src/search.js'    
    },    
    output: {        
        filename: '[name][chunkhash:8].js',        
        path:__dirname + '/dist'    
    },    
    plugins:[        
        new MiniCssExtractPlugin({            
            filename: `[name][contenthash:8].css`        
        })    
    ]}
```

**图片的文件指纹设置**

**设置file-loader的name，使用hash**。

占位符名称及含义

- ext     资源后缀名
- name    文件名称
- path    文件的相对路径
- folder  文件所在的文件夹
- contenthash   文件的内容hash，默认是md5生成
- hash         文件内容的hash，默认是md5生成
- emoji        一个随机的指代文件内容的emoj

```javascript
const path = require('path');
module.exports = {    
    entry: './src/index.js',    
    output: {        
        filename:'bundle.js',        
        path:path.resolve(__dirname, 'dist')    
    },    
    module:{        
        rules:[{            
            test:/\.(png|svg|jpg|gif)$/,            
            use:[{                
                loader:'file-loader',                
                options:{                    
                    name:'img/[name][hash:8].[ext]'                
                }            
            }]        
        }]    
    }}
```

## **在实际工程中，配置文件上百行乃是常事，如何保证各个loader按照预想方式工作**

可以**使用 `enforce` 强制执行 `loader` 的作用顺序 **，`pre` 代表在所有正常 loader 之前执行，`post` 是所有 loader 之后执行。(inline 官方不推荐使用)

## 7.是否写过Loader和Plugin？描述一下编写loader或plugin的思路？

Loader像一个"翻译官"把读到的源文件内容转义成新的文件内容，并且每个Loader通过链式操作，将源文件一步步翻译成想要的样子。

编写Loader时要遵循单一原则，每个Loader只做一种"转义"工作。 每个Loader的拿到的是源文件内容（`source`），可以通过返回值的方式将处理后的内容输出，也可以调用`this.callback()`方法，将内容返回给webpack。 还可以通过 `this.async()`生成一个`callback`函数，再用这个callback将处理后的内容输出出去。 此外`webpack`还为开发者准备了开发loader的工具函数集——`loader-utils`。

相对于Loader而言，Plugin的编写就灵活了许多。 webpack在运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

## 8.webpack的热更新是如何做到的？说明其原理？

webpack的热更新又称热替换（Hot Module Replacement），缩写为HMR。 这个机制可以做到**不用刷新浏览器而将新变更的模块替换掉旧的模块**。

![img](https://pic3.zhimg.com/80/v2-40ff7f2e518e4b4695777d5160a3406e_720w.jpg)

首先要知道server端和client端都做了处理工作

1. 第一步，在 webpack 的 watch 模式下，文件系统中某一个文件发生修改，webpack 监听到文件变化，根据配置文件对模块重新编译打包，并将打包后的代码通过简单的 JavaScript 对象保存在内存中。
2. 第二步是 webpack-dev-server 和 webpack 之间的接口交互，而在这一步，主要是 dev-server 的中间件 webpack-dev-middleware 和 webpack 之间的交互，webpack-dev-middleware 调用 webpack 暴露的 API对代码变化进行监控，并且告诉 webpack，将代码打包到内存中。
3. 第三步是 webpack-dev-server 对文件变化的一个监控，这一步不同于第一步，并不是监控代码变化重新打包。当我们在配置文件中配置了devServer.watchContentBase 为 true 的时候，Server 会监听这些配置文件夹中静态文件的变化，变化后会通知浏览器端对应用进行 live reload。注意，这儿是浏览器刷新，和 HMR 是两个概念。
4. 第四步也是 webpack-dev-server 代码的工作，该步骤主要是通过 sockjs（webpack-dev-server 的依赖）在浏览器端和服务端之间建立一个 websocket 长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端，同时也包括第三步中 Server 监听静态文件变化的信息。浏览器端根据这些 socket 消息进行不同的操作。当然服务端传递的最主要信息还是新模块的 hash 值，后面的步骤根据这一 hash 值来进行模块热替换。
5. webpack-dev-server/client 端并不能够请求更新的代码，也不会执行热更模块操作，而把这些工作又交回给了 webpack，webpack/hot/dev-server 的工作就是根据 webpack-dev-server/client 传给它的信息以及 dev-server 的配置决定是刷新浏览器呢还是进行模块热更新。当然如果仅仅是刷新浏览器，也就没有后面那些步骤了。
6. HotModuleReplacement.runtime 是客户端 HMR 的中枢，它接收到上一步传递给他的新模块的 hash 值，它通过 JsonpMainTemplate.runtime 向 server 端发送 Ajax 请求，服务端返回一个 json，该 json 包含了所有要更新的模块的 hash 值，获取到更新列表后，该模块再次通过 jsonp 请求，获取到最新的模块代码。这就是上图中 7、8、9 步骤。
7. 而第 10 步是决定 HMR 成功与否的关键步骤，在该步骤中，HotModulePlugin 将会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用。
8. 最后一步，当 HMR 失败后，回退到 live reload 操作，也就是进行浏览器刷新来获取最新打包代码。

( HMR的核心就是客户端从服务端拉去更新后的文件，准确的说是 chunk diff (chunk 需要更新的部分)，实际上 WDS 与浏览器之间维护了一个 `Websocket`，当本地资源发生变化时，WDS 会向浏览器推送更新，并带上构建时的 hash，让客户端与上一次资源进行对比。客户端对比出差异后会向 WDS 发起 `Ajax` 请求来获取更改内容(文件列表、hash)，这样客户端就可以再借助这些信息继续向 WDS 发起 `jsonp` 请求获取该chunk的增量更新。

后续的部分(拿到增量更新之后如何处理？哪些状态该保留？哪些又需要更新？)由 `HotModulePlugin` 来完成，提供了相关 API 以供开发者针对自身场景进行处理，像`react-hot-loader` 和 `vue-loader` 都是借助这些 API 实现 HMR。)

## 9.如何利用webpack来优化前端性能？（提高性能和体验）

用webpack优化前端性能是指**优化webpack的输出结果，让打包的最终结果在浏览器运行快速高效**。

- ***压缩代码***。**删除多余的代码、注释、简化代码的写法等**等方式。可以利用webpack的`UglifyJsPlugin`和`ParallelUglifyPlugin`来压缩JS文件， 利用`cssnano`（css-loader?minimize）来压缩css
- **利用CDN加速 **。在构建过程中，**将引用的静态资源路径修改为CDN上对应的路径**。可以利用webpack对于`output`参数和各loader的`publicPath`参数来修改资源路径
- **删除死代码（Tree Shaking） **。**将代码中永远不会走到的片段删除掉**。可以通过在启动webpack时追加参数**`--optimize-minimize`**来实现
- **提取公共代码 **。

## 10.如何提高webpack的构建速度？

1. 多入口情况下，使用`CommonsChunkPlugin`来提取公共代码
2. 通过`externals`配置来提取常用库
3. 利用`DllPlugin`和`DllReferencePlugin`预编译资源模块 通过`DllPlugin`来对那些我们引用但是绝对不会修改的npm包来进行预编译，再通过`DllReferencePlugin`将预编译的模块加载进来。
4. 使用`Happypack` 实现多线程加速编译
5. 使用`webpack-uglify-parallel`来提升`uglifyPlugin`的压缩速度。 原理上`webpack-uglify-parallel`采用了多核并行压缩来提升压缩速度
6. 使用`Tree-shaking`和`Scope Hoisting`来剔除多余代码

使用`高版本`的 Webpack 和 Node.js

`多进程/多实例构建`：HappyPack(不维护了)、thread-loader

**压缩代码：**

- 多进程并行压缩
  - webpack-paralle-uglify-plugin
  - uglifyjs-webpack-plugin 开启 parallel 参数 (不支持ES6)
  - terser-webpack-plugin 开启 parallel 参数
- 通过 mini-css-extract-plugin 提取 Chunk 中的 CSS 代码到单独文件，通过 css-loader 的 minimize 选项开启 cssnano 压缩 CSS。

**图片压缩：**

- 使用基于 Node 库的 imagemin (很多定制选项、可以处理多种图片格式)
- 配置 image-webpack-loader

`缩小打包作用域`：

- exclude/include (确定 loader 规则范围)
- resolve.modules 指明第三方模块的绝对路径 (减少不必要的查找)
- resolve.mainFields 只采用 main 字段作为入口文件描述字段 (减少搜索步骤，需要考虑到所有运行时依赖的第三方模块的入口文件描述字段)
- resolve.extensions 尽可能减少后缀尝试的可能性
- noParse 对完全不需要解析的库进行忽略 (不去解析但仍会打包到 bundle 中，注意被忽略掉的文件里不应该包含 import、require、define 等模块化语句)
- IgnorePlugin (完全排除模块)
- 合理使用alias

`提取页面公共资源`：

- 基础包分离：
  - 使用 html-webpack-externals-plugin，将基础包通过 CDN 引入，不打入 bundle 中
  - 使用 SplitChunksPlugin 进行(公共脚本、基础包、页面公共文件)分离(Webpack4内置) ，替代了 CommonsChunkPlugin 插件

`DLL`：

- 使用 DllPlugin 进行分包，使用 DllReferencePlugin(索引链接) 对 manifest.json 引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间。
- HashedModuleIdsPlugin 可以解决模块数字id问题

`充分利用缓存提升二次构建速度`：

- babel-loader 开启缓存
- terser-webpack-plugin 开启缓存
- 使用 cache-loader 或者 hard-source-webpack-plugin

```
Tree shaking
```

- 打包过程中检测工程中没有引用过的模块并进行标记，在资源压缩时将它们从最终的bundle中去掉(只能对ES6 Modlue生效) 开发中尽可能使用ES6 Module的模块，提高tree shaking效率
- 禁用 babel-loader 的模块依赖解析，否则 Webpack 接收到的就都是转换过的 CommonJS 形式的模块，无法进行 tree-shaking
- 使用 PurifyCSS(不在维护) 或者 uncss 去除无用 CSS 代码
  - purgecss-webpack-plugin 和 mini-css-extract-plugin配合使用(建议)

```
Scope hoisting
```

- 构建后的代码会存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。Scope hoisting 将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突
- 必须是ES6的语法，因为有很多第三方库仍采用 CommonJS 语法，为了充分发挥 Scope hoisting 的作用，需要配置 mainFields 对第三方模块优先采用 jsnext:main 中指向的ES6模块化语法

```
动态Polyfill
```

- 建议采用 polyfill-service 只给用户返回需要的polyfill，社区维护。 (部分国内奇葩浏览器UA可能无法识别，但可以降级返回所需全部polyfill)

## 11.怎么配置单页应用？怎么配置多页应用？

单页应用可以理解为webpack的标准模式，直接**在`entry`中指定单页应用的入口即可 **，这里不再赘述

多页应用的话，可以使用webpack的 `AutoWebPlugin`来完成简单自动化的构建，但是前提是项目的目录结构必须遵守他预设的规范。 多页应用中要注意的是：

- 每个页面都有公共的代码，可以将这些代码抽离出来，避免重复的加载。比如，每个页面都引用了同一套css样式表
- 随着业务的不断扩展，页面可能会不断的追加，所以一定要让入口的配置足够灵活，避免每次添加新页面还需要修改构建配置

## 12.npm打包时需要注意哪些？如何利用webpack来更好的构建？

NPM模块需要注意以下问题：

1. 要支持CommonJS模块化规范，所以要求打包后的最后结果也遵守该规则。
2. Npm模块使用者的环境是不确定的，很有可能并不支持ES6，所以打包的最后结果应该是采用ES5编写的。并且如果ES5是经过转换的，请最好连同SourceMap一同上传。
3. Npm包大小应该是尽量小（有些仓库会限制包大小）
4. 发布的模块不能将依赖的模块也一同打包，应该让用户选择性的去自行安装。这样可以避免模块应用者再次打包时出现底层模块被重复打包的情况。
5. UI组件类的模块应该将依赖的其它资源文件，例如`.css`文件也需要包含在发布的模块里

基于以上需要注意的问题，我们可以对于webpack配置做以下扩展和优化：

1. CommonJS模块化规范的解决方案： 设置`output.libraryTarget='commonjs2'`使输出的代码符合CommonJS2 模块化规范，以供给其它模块导入使用
2. 输出ES5代码的解决方案：使用`babel-loader`把 ES6 代码转换成 ES5 的代码。再通过开启`devtool: 'source-map'`输出SourceMap以发布调试。
3. Npm包大小尽量小的解决方案：Babel 在把 ES6 代码转换成 ES5 代码时会注入一些辅助函数，最终导致每个输出的文件中都包含这段辅助函数的代码，造成了代码的冗余。解决方法是修改`.babelrc`文件，为其加入`transform-runtime`插件
4. 不能将依赖模块打包到NPM模块中的解决方案：使用`externals`配置项来告诉webpack哪些模块不需要打包。
5. **对于依赖的资源文件打包 **的解决方案：通过`css-loader`和`extract-text-webpack-plugin`来实现，配置如下：

```javascript
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  module: {
    rules: [
      {
        // 增加对 CSS 文件的支持
        test: /\.css/,
        // 提取出 Chunk 中的 CSS 代码到单独的文件中
        use: ExtractTextPlugin.extract({
          use: ['css-loader']
        }),
      },
    ]
  },
  plugins: [
    new ExtractTextPlugin({
      // 输出的 CSS 文件名称
      filename: 'index.css',
    }),
  ],
};
```

## 13.如何在vue项目中实现按需加载？

**Vue UI组件库的按需加载** 为了快速开发前端项目，经常会引入现成的UI组件库如ElementUI、iView等，但是他们的体积和他们所提供的功能一样，是很庞大的。 而通常情况下，我们仅仅需要少量的几个组件就足够了，但是我们却将庞大的组件库打包到我们的源码中，造成了不必要的开销。

不过很多组件库已经提供了现成的解决方案，如Element出品的`babel-plugin-component`和AntDesign出品的`babel-plugin-import` 安装以上插件后，在`.babelrc`配置中或`babel-loader`的参数中进行设置，即可实现组件按需加载了。

```javascript
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```

**单页应用的按需加载** 现在很多前端项目都是通过单页应用的方式开发的，但是随着业务的不断扩展，会面临一个严峻的问题——首次加载的代码量会越来越多，影响用户的体验。

通过`import(*)`语句来控制加载时机，webpack内置了对于`import(*)`的解析，会将`import(*)`中引入的模块作为一个新的入口在生成一个chunk。 当代码执行到`import(*)`语句时，会去加载Chunk对应生成的文件。`import()`会返回一个Promise对象，所以为了让浏览器支持，需要事先注入`Promise polyfill`