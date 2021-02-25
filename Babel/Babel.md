# Babel

> babel.config.js(babel7)或者.babelrc(<7)文件，.babelrc.js` ，或者放到`package.json，是用来设置转码的规则和插件。

`npm install -D @babel/core @babel/cli`

`@babel/cli`是babel的命令行工具，主要提供`babel`命令。

`babel` 的核心功能包含在 `@babel/core` 模块中。没有它，在 `babel` 的世界里注定寸步难行。

```json
# vue项目
{
 // 此项指明，转码的规则
 "presets": [
 // env项是借助插件babel-preset-env，下面这个配置说的是babel对es6,es7,es8进行转码，并且设置amd,commonjs这样的模块化文件，不进行转码
 ["@babel/preset-env", { "modules": false }],
 // 下面这个是不同阶段出现的es语法，包含不同的转码插件
 "stage-2"
 ],
 // 下面这个选项是引用插件来处理代码的转换，transform-runtime用来处理全局函数和优化babel编译
 "plugins": ["@babel/plugin-transform-runtime","@babel/plugin-transform-arrow-functions","syntax-dynamic-import",{
      "corejs": 3 // 指定 runtime-corejs 的版本
 }], // "syntax-dynamic-import"可不需要
 // 下面指的是在生成的文件中，不产生注释
 "comments": false,
 // 下面这段是在特定的环境中所执行的转码规则，当环境变量是下面的test就会覆盖上面的设置
 "env": {
 // test 是提前设置的环境变量，如果没有设置BABEL_ENV则使用NODE_ENV，如果都没有设置默认就是development
 "test": {
  "presets": ["env", "stage-2"],
  // instanbul是一个用来测试转码后代码的工具
  "plugins": ["istanbul"]
 }
 }
}
```

```json
# react项目
{
 "presets": [
  ["@babel/preset-env", { "modules": false }],
  "stage-2",
  "react"
 ],
 "plugins": ["transform-runtime","@babel/plugin-transform-arrow-functions"],
 "comments": false,
 "env": {
  "test": {
   "presets": ["env", "stage-2"],
   "plugins": [ "istanbul" ]
  }
 }
}

```



## 语法转译器

> 主要对 JavaScript 最新的语法糖进行编译，并不负责转译新增的 API 和全局对象。
> 如 Promise,Iterator,Generator,Set,Maps,Proxy,Symbol 等全局对象，以及一些定义在全局对象的方法（比如 includes/Object.assign 等）并不能被编译。

常用到的转译器包有，babel-preset-env、babel-preset-es2015、babel-preset-es2016、babel-preset-es2017、babel-preset-latest等。

在实际开发中可以只选用babel-preset-env来代替余下的，但是还需要配上javascirpt的制作规范一起使用，同时也是官方推荐

另外最新的提案包含在 stage-* 中，stage里面包含了当年最新规范的草案，每年更新。

```json
{
  // preset就是一组插件的集合
 "presets": ["@babel/preset-env", {
   "modules": false,
     "useBuiltIns": "usage",
        "corejs": 3
  }],
  "stage-2" // npm install -D babel-preset-stage-2 
}

```

关键配置选项 useBuiltIns，它的取值共有三个：

- false：即不处理 api，不会自动对每个文件的api进行转换，也不会去引入polyfill。
- entry：对全部文件进行api转换，并且需要在入口文件中手动引入`import from @babel/polyfill`。
- usage：对api的转换采用按需加载，即用到哪个方法就自动引入对应的转换代码，不会全量的引入polyfill，无需在页面入口手动引入`import from @babel/polyfill`。

其中`false`是默认值，一般用的比较少，而`@babel/polyfill`在Babel 7.4之后分成了两个库core-js和regenerator-runtime，所以需要引入这两个库来代替，一般情况下使用`usage`是最合适的，在构建中会自动扫描代码中使用的新api，并引入对应的转换代码，而不是全量引入，从而可以减少资源包的大小，但是需要注意的是：

- 配置`usage`可以按需引入转换代码，但是@babel/polyfill依然需要安装。但是引入方式需要修改成`core-js`和`regenerator-runtime`。
- 配置`usage`可以按需引入转换代码，但是对于`node_modules`文件夹下的代码，默认是不会转换的（使用vue cli创建的项目，babel-loader默认不会转换这部分代码），所以类似ant-design，element-ui这些使用了新的api的库，在`node_modules`里是不会被转换的。

对于此，可以再用`entry`方式来全量引入，确保api转换不会出问题，当然可也以修改`vue.config.js`配置文件来增加对`node_modules`下的库转换，代码如下：

```javascript
module.exports = {
    ...
	transpileDependencies: ['ant-design-vue'],
}
```

另外，对于`@babel/preset-env`的配置项中的`corejs`需要和安装的corejs版本一致。

* `npm install -D @babel/preset-env`

  `@babel/preset-env`所包含的插件将支持所有最新的JS特性(es2015 es2016等不包含 stage 阶段)，将其转换成ES5代码。允许我们使用最新的js语法，比如 let，const，箭头函数等等。

  但是babel只转换新的js语法，如箭头函数等，但不转换新的API，这时候需要借助`@babel/polyfill`，把es的新特性都装进来。

  ```json
  {
    "presets": [
      ["@babel/preset-env",  {
        "modules": false //默认会将模块转换为commonjs, 还有其他配置参数："amd" | "umd" | "systemjs" | "commonjs",
          "useBuiltIns": "usage",
          "corejs": 3
      }]
    ]
  }
  ```

## API 和全局对象转译器

> 负责转译新增的 API 和全局对象，保证在浏览器的兼容性。如Promise,Iterator,Generator,Set,Maps,Proxy,Symbol 等全局对象，以及一些定义在全局对象的方法（比如 includes/Object.assign 等）

### babel polyfill

> 垫片，可以转译所有 ES6 API 和全局对象。用来垫平不同浏览器或者不同环境下的差异，让新的内置函数、实例方法等在低版本浏览器中也可以使用。

`@babel/polyfill` 模块包括 `core-js` 和一个自定义的 `regenerator runtime` 模块，可以模拟完整的 ES2015+ 环境（不包含第4阶段前的提议）。

babel v7.4版之后，官方不推荐再使用@babel/polyfill了，需要直接安装`core-js` 和 `regenerator-runtime`去替代之前的`@babel/polyfill`。

> 虽然@babel/polyfill还在进行版本升级，但其使用的core-js包为2.x.x版本，而core-js这个包本身已经发布到了3.x.x版本了，@babel/polyfill以后也不会使用3.x.x版本的包了。`core-js@2` 分支中已经不会再添加新特性，新特性都会添加到 `core-js@3`。

`npm install -S core-js regenerator-runtime`

```js
//import '@babel/polyfill'; （babel v7.4版之后，不推荐使用）
import "core-js/stable";
import "regenerator-runtime/runtime";

let func = () => { }
let arr = [1, 2, 4]
arr.includes(3)
```

**缺点：**增加包体，比如仅是使用到一种 ES6 新增 API，他也会增加所有的转移语法。所以使用`@babel/runtime`把所有语法转换会用到的辅助函数都集成在一起

1. `npm install -S @babel/runtime` // 产生辅助函数

   `npm install -D @babel/plugin-transform-runtime` // 这个插件的作用就是自动移除语法转换后内联的辅助函数。另一个作用是创建一个沙盒环境来避免对全局环境的污染。`@babel/plugin-transform-runtime`的大多数应用场景是在写第三方库时来使用，这样可以避免影响到使用者的JavaScript环境，造成全局污染。在自己独立的项目中，更多情况下推荐使用`@babel/preset-env`搭配`useBuiltIns`即可，而无需再使用`@babel/plugin-transform-runtime`，参考[issues](https://github.com/babel/babel/issues/10008)

   `npm install -S @babel/runtime-corejs3` // 是@babel/runtime的进化版，开启@babel/plugin-transform-runtime的API转换功能

   对于@babel/runtime及其进化版@babel/runtime-corejs2、@babel/runtime-corejs3，我们只需要根据自己的需要安装一个。

   * 如果你不需要对core-js做API转换，那就安装@babel/runtime并把corejs配置项设置为false即可。
   * 如果你需要用core-js2做API转换，那就安装@babel/runtime-corejs2并把corejs配置项设置为2即可。
   *  如果你需要用core-js3做API转换，那就安装@babel/runtime-corejs3并把corejs配置项设置为3即可。

   

   > 全自动的，不会污染全局 API。但是它只会对es6的语法进行转换，而不会对新api进行转换。如果需要转换新api，也可以通过使用`@babel/polyfill`来规避兼容性问题

   ```json
   {
     "presets": [
       "@babel/preset-env"
     ],
     "plugins": [
       [
         "@babel/plugin-transform-runtime",
         {
           "corejs": 3 //corejs取值是false、2和3，默认值是false
         }
       ]
     ]
   }
   
   ```

   `npm install -D babel-plugin-syntax-dynamic-import`

   这个插件主要解决动态引入模块的问题。如果.babelrc配置项中使用了"stage-2"，也可以不实用该插件，同样支持动态模块引入

   ```js
   function nDate() {
    import('moment').then(function(moment) {
     console.log(moment().format());
    }).catch(function(err) {
     console.log('Failed to load moment', err);
    });
   }
   nDate();
   ```

   

2. `npm install -D @babel/polyfill`

   修改全局 prototype 来对 API 和全局变量的垫片的，所以可以转译实例方法。

   ```json
   1.import "@babel/polyfill";
   
   2.require('@babel/polyfill');
   
   3.module.exports = { // 适合使用 webpack 构建的项目
   　　entry: ["@babel/polyfill", "./app/js"]
   };
   ```

3. `npm install -S es6-promise`

   如果仅使用到 promise 相关 API，可以仅添加此转译器

   ```js
   import _promise from 'es6-promise';
   
   const Promise = _promise.Promise;
   ```

## jsx,flow,TypeScript 等插件转译器

* `npm install -D @babel/preset-react`

  `npm install -D @babel/preset-env @babel/plugin-transform-runtime`

  使用 react 项目时，需要使用此包配合转译

  ```json
  {
    "presets": [
      ["@babel/env", {
        "modules": false
      }],
      "@babel/preset-react"
    ],
    "comments": false, //不产生注释
    "plugins": ["@babel/plugin-transform-runtime"]
  }
  ```

* `npm install --save-dev @babel/preset-typescript`

  当项目使用 TypeScript 编写时，需要使用此包配合转译

  ```json
  {
      "presets": ["@babel/preset-typescript"]
  }
  ```

## Browserslist 集成

`@babel/preset-env`会根据配置的目标环境，生成插件列表来编译。当不需要兼容所有的浏览器和环境时，通过指定目标环境，可以让编译代码保持最小。

官方推荐使用 `.browserslistrc` 文件来指定目标环境。

```json
# .browserslistrc
> 1%
not ie <= 8
```

上面的配置含义是，目标环境是市场份额大于1%的浏览器并且不考虑IE8及以下的IE浏览器。

```json
last 2 Chrome versions // 执行 `npm run babel`，你会发现箭头函数不会被编译成ES5，因为 `chrome` 的最新2个版本都能够支持箭头函数。
```

查看 [`browserslist` 的更多配置](https://github.com/browserslist/browserslist)

## webpack配置babel

`npm install  -D babel-loader`

```js
# webpack.config.js
module: {
    rules:[
    {
            test: /\.js$/,
            <!--引入的第三方模块不转码-->
            exclude: /node_modules/, 
            use: [{
                loader: 'babel-loader',
                options: {
                 <!--设置cache目录，加快转码速度，默认目录为 node_modules/.cache/babel-loader-->
                cacheDirectory: true,
                },
            }],
        },
    ]
},
<!--在开发模式下配置，通过chrome devtool/source/webpack-internal可以看到babel转码结果-->
devtool: 'cheap-module-eval-source-map',

```

**另外如果在webpack中配置eslint,还要保证eslint的检查在babel转码之前,配置如下**

```js
module: {
    rules:[{
        // 保证先于 babel-loader 执行
        enforce: 'pre',
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'eslint-loader'
    }]
}

```

```js
# .eslintrc.js 
parser: 'babel-eslint'
```

## 其他插件

- [babel-plugin-transform-remove-strict-mode 移除严格模式 ](https://github.com/genify/babel-plugin-transform-remove-strict-mode)
- [babel-plugin-lodash](https://github.com/lodash/babel-plugin-lodash)
- [babel-plugin-equire 按需加载echarts模块](https://github.com/ywwhack/babel-plugin-equire)
- [babel-plugin-component 按需加载Element UI模块](https://github.com/ElementUI/babel-plugin-component)

## 总结

- babel7的包都是以 @babel开头的，所有的模块插件啥的都是在node_modules/@babel/目录下
- babel中真正干活的是插件，插件的作用是transform
- preset（预置），就是一个插件包的集合，你也可以自己写插件包,插件包不够用，再配置plugins呗
- 配置了babel7,肯定要用新版本的babel-loader,老版本的babel-loader会找 babel-core而不是@babel/core会报错
- babel7新增了babel.config.js配置文件，是项目级别的配置，建议使用， .babelrc配置仍然可以用，至于为啥，memorepo可以了解一下
- babel7废弃了babel-preset-stage-0这样的配置，如果不懂stage-0啥的，自行学习，如果要用，需要直接引入cores-js的相关模块
- @babel-preset-env 这样的preset可简写，如上
- polyfill垫片，就是web开发中，可移植的代码补充库
- 关于babel-polyfill其实就是core-js 和 一个工具库
- core-js是实现了一些现在没支持的新功能的库，分2和3两个大版本
- babel6之后，对export default 这样的 es 模块写法不支持，需要一个 babel-plugin-add-module-exports的库支持
- 要支持装饰器的写法，需要@babel/plugin-proposal-decorators 插件支持，配置如上
- babel-plugin-import 是配置antd按需加载的一个插件，做React项目，几乎antd是必备的吧

