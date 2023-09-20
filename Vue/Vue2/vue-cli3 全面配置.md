## 总体配置

```js
// vue.config.js
const path = require('path')
const resolve = dir => path.join(__dirname, dir)
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)

module.exports = {
    publicPath: './',      // 编译后的地址，可以根据环境进行设置
    assetsDir: 'assets',   // 静态文件目录
    lintOnSave: true,      // 是否开启编译时是否不符合eslint提示
    productionSourceMap: !IS_PROD, // 生产环境的 source map
    parallel: require('os').cpus().length > 1,
    css: {
        extract: IS_PROD, // 提取css代码
        sourceMap: false,
    },
    chainWebpack: config => {
        // 代码压缩
        config.optimization.minimize(true)

        // 代码分割
        if (IS_PROD) {
            config.optimization.delete('splitChunks')
        }

        // 项目目录别名 ( src目录配置为@ )
        config.resolve.alias
            .set('@', resolve('src'))
            .set('assets', resolve('src/assets'))
        
        return config
    },
    configureWebpack: config => {
        if (IS_PROD) {
            config.optimization = {
                splitChunks: {
                    cacheGroups: {
                        libs: {
                            name: 'chunk-libs',
                            test: /[\\/]node_modules[\\/]/,
                            priority: 10,
                            chunks: 'initial'
                        },
                        elementUI: {
                            name: 'chunk-elementUI',
                            priority: 20,
                            test: /[\\/]node_modules[\\/]element-ui[\\/]/,
                            chunks: 'all'
                        }
                    }
                }
            }
        }
    }
}
```

### 配置`development`、`production`环境变量

通过在 `package.json` 里的 `scripts` 配置项中添加`--mode xxx` 来选择不同环境

* 只有以 `VUE_APP` 开头的变量会被 `webpack.DefinePlugin` 静态嵌入到客户端侧的包中，代码中可以通过 `process.env.VUE_APP_XXXX` 访问
* `NODE_ENV` 和 `BASE_URL` 是两个特殊变量，在代码中始终可用`process.env.NODE_ENV`、`process.env.BASE_URL`

```json
{
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "analyz": "vue-cli-service build --mode analyz",
    "lint": "vue-cli-service lint"
  }
}
```

1. `.env` 文件

   `serve` 默认的本地开发环境配置

   ```sh
   NODE_ENV = 'development'
   BASE_URL = './'
   VUE_APP_PUBLIC_PATH = './'
   VUE_APP_API = 'https://test.staven630.com/api'
   ```

2. `.env.production` 文件

   `build` 默认的环境配置

   ```sh
   NODE_ENV = 'production'
   BASE_URL = 'https://prod.staven630.com/'
   VUE_APP_PUBLIC_PATH = 'https://prod.oss.com/staven-blog'
   VUE_APP_API = 'https://prod.staven630.com/api'
   
   ACCESS_KEY_ID = 'xxxxxxxxxxxxx'
   ACCESS_KEY_SECRET = 'xxxxxxxxxxxxx'
   REGION = 'oss-cn-hangzhou'
   BUCKET = 'staven-prod'
   PREFIX = 'staven-blog'
   ```

3. `.env.analyz` 文件

   自定义 build 环境配置

   ```shell
   NODE_ENV = 'production'
   BASE_URL = 'https://prod.staven630.com/'
   VUE_APP_PUBLIC_PATH = 'https://prod.oss.com/staven-blog'
   VUE_APP_API = 'https://prod.staven630.com/api'
   
   ACCESS_KEY_ID = 'xxxxxxxxxxxxx'
   ACCESS_KEY_SECRET = 'xxxxxxxxxxxxx'
   REGION = 'oss-cn-hangzhou'
   BUCKET = 'staven-prod'
   PREFIX = 'staven-blog'
   
   IS_ANALYZE = true
   ```

### 基础配置

```js
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
module.exports = {
  publicPath: IS_PROD ? process.env.VUE_APP_PUBLIC_PATH : './', // 默认'/'，部署应用包时的基本 URL
  lintOnSave: true,      // 是否开启编译时是否不符合eslint提示
  // outputDir: process.env.outputDir || 'dist', // 'dist', 生产环境构建文件的目录
  // assetsDir: "assets", // 静态文件目录,相对于outputDir的静态资源(js、css、img、fonts)目录
  runtimeCompiler: true, // 是否使用包含运行时编译器的 Vue 构建版本
  productionSourceMap: !IS_PROD, // 生产环境的 source map
  parallel: require('os').cpus().length > 1,
  pwa: {}
}
```

### **项目目录别名 ( src目录配置为@ )

```js
const path = require('path')
const resolve = dir => path.join(__dirname, dir)
module.exports = {
	chainWebpack: (config) => {
    // 配置别名
    config.resolve.alias
        .set('@', resolve('src'))
        .set('assets',resolve('src/assets'))
        .set('components',resolve('src/components'))
        .set('router',resolve('src/router'))
        .set('utils',resolve('src/utils'))
        .set('static',resolve('src/static'))
        .set('store',resolve('src/store'))
        .set('views',resolve('src/views'))
	}
}
```

### **代码压缩

```js
module.exports = {
  chainWebpack: config => {
    config.optimization.minimize(true); // 代码压缩
  }
}
```

### **利用 splitChunks 单独打包第三方模块

```js
module.exports = {
  chainWebpack: config => {
    config.optimization.splitChunks({   // 代码分割
      chunks: 'all'
    })
  }
}
```

或者

```js
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)

module.exports = {
  configureWebpack: config => {
    if (IS_PROD) {
      config.optimization = {
        splitChunks: {
          cacheGroups: {
            libs: {
              name: 'chunk-libs',
              test: /[\\/]node_modules[\\/]/,
              priority: 10,
              chunks: 'initial'
            },
            elementUI: {
              name: 'chunk-elementUI',
              priority: 20,
              test: /[\\/]node_modules[\\/]element-ui[\\/]/,
              chunks: 'all'
            }
          }
        }
      }
    }
  },
  chainWebpack: config => {
    if (IS_PROD) {
      config.optimization.delete('splitChunks')
    }
    return config
  }
}
```

### **提取css代码

因为js会动态的加载出css，所以js文件包会比较大，那么需要提取css代码到文件. 这里我们只需要将css配置一下:

```js
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
module.exports = {
  css: {
      extract: IS_PROD
  }
}
```

如图，`css`从`app.js`文件中单独分离出来

![img](../../assets/vue/extract_css.jpg)

### 配置 proxy 代理解决跨域问题

在前端请求过程中，如果后台没有设置跨域请求的，可以在webpack进行配置。

```js
module.exports = {
  // overlay: { // 让浏览器 overlay 同时显示警告和错误
  //   warnings: true,
  //   errors: true
  // },
  // open: false, // 是否打开浏览器
  // host: "localhost",
  // port: "8080", // 代理断就
  // https: false,
  // hotOnly: false, // 热更新
  proxy: {
    '/api': {
      target:
      'https://www.easy-mock.com/mock/5bc75b55dc36971c160cad1b/sheets', // 目标代理接口地址
      secure: false,
      changeOrigin: true, // 开启代理，在本地创建一个虚拟服务端
      // ws: true, // 是否启用websockets
      pathRewrite: {
        '^/api': '/'
      }
    }
  }
}
```

访问

```vue
<script>
import axios from "axios";
export default {
  mounted() {
    axios.get("/api/1").then(res => {
      console.log(res);
    });
  }
};
</script>
```

### 开启gzip压缩

`npm install --save-dev compression-webpack-plugin`

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin')

const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
const productionGzipExtensions = /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i

module.exports = {
  configureWebpack: config => {
    const plugins = []
    if (IS_PROD) {
      plugins.push(
        new CompressionWebpackPlugin({
          filename: '[path].gz[query]',
          algorithm: 'gzip',
          test: productionGzipExtensions,
          threshold: 10240,
          minRatio: 0.8
        })
      )
    }
    config.plugins = [...config.plugins, ...plugins]
  }
}
```

还可以开启比 gzip 体验更好的 Zopfli 压缩详见https://webpack.js.org/plugins/compression-webpack-plugin

`npm i -D @gfx/zopfli brotli-webpack-plugin`

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin')
const zopfli = require('@gfx/zopfli')
const BrotliPlugin = require('brotli-webpack-plugin')

const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
const productionGzipExtensions = /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i

module.exports = {
  configureWebpack: config => {
    const plugins = []
    if (IS_PROD) {
      plugins.push(
        new CompressionWebpackPlugin({
          algorithm(input, compressionOptions, callback) {
            return zopfli.gzip(input, compressionOptions, callback)
          },
          compressionOptions: {
            numiterations: 15
          },
          minRatio: 0.99,
          test: productionGzipExtensions
        })
      )
      plugins.push(
        new BrotliPlugin({
          test: productionGzipExtensions,
          minRatio: 0.99
        })
      )
    }
    config.plugins = [...config.plugins, ...plugins]
  }
}
```

### 压缩图片

`npm i -D image-webpack-loader`

```js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('images')
      .use('image-webpack-loader')
      .loader('image-webpack-loader')
      .options({
        mozjpeg: { progressive: true, quality: 65 },
        optipng: { enabled: false },
        pngquant: { quality: [0.65, 0.9], speed: 4 },
        gifsicle: { interlaced: false },
        webp: { quality: 75 }
      })
  }
}
```

### 添加打包分析

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
  chainWebpack: config => {
    // 打包分析
    if (process.env.IS_ANALY) {
      config.plugin('webpack-report').use(BundleAnalyzerPlugin, [
        {
          analyzerMode: 'static'
        }
      ])
    }
  }
}
```

添加`.env.analyz` 文件

```sh
NODE_ENV = 'production'
IS_ANALYZ = true
```

package.json 的 scripts 中添加

```js
"analyz": "vue-cli-service build --mode analyz"
```

执行

`npm run analyz`

### 外部库使用CDN加载

> 防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖

```js
module.exports = {
  configureWebpack: config => {
    config.externals = {
      vue: 'Vue',
      'element-ui': 'ELEMENT',
      'vue-router': 'VueRouter',
      vuex: 'Vuex',
      axios: 'axios'
    }
  },
  chainWebpack: config => {
    const cdn = {
      // 访问https://unpkg.com/element-ui/lib/theme-chalk/index.css获取最新版本
      css: ['//unpkg.com/element-ui@2.10.1/lib/theme-chalk/index.css'],
      js: [
        '//unpkg.com/vue@2.6.10/dist/vue.min.js', // 访问https://unpkg.com/vue/dist/vue.min.js获取最新版本
        '//unpkg.com/vue-router@3.0.6/dist/vue-router.min.js',
        '//unpkg.com/vuex@3.1.1/dist/vuex.min.js',
        '//unpkg.com/axios@0.19.0/dist/axios.min.js',
        '//unpkg.com/element-ui@2.10.1/lib/index.js'
      ]
    }

    // html中添加cdn
    config.plugin('html').tap(args => {
      args[0].cdn = cdn
      return args
    })
  }
}
```

在html中添加

```html
<!-- 使用CDN的CSS文件 -->
<% for (var i in htmlWebpackPlugin.options.cdn &&
htmlWebpackPlugin.options.cdn.css) { %>
<link rel="stylesheet" href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" />
<% } %>

<!-- 使用CDN的JS文件 -->
<% for (var i in htmlWebpackPlugin.options.cdn &&
htmlWebpackPlugin.options.cdn.js) { %>
<script
  type="text/javascript"
  src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"
></script>
<% } %>
```

### 为 sass 提供全局样式，以及全局变量 (自动导入)

```text
assets
 ├─ css
    ├─ common.scss    存放公共的样式
    ├─ mixin.scss     存放混入样式
    ├─ reset.scss     存放重置样式
    └─ variable.scss  存放变量
```

```js
module.exports = {
  css: {
    loaderOptions: {
      modules: false,
    	extract: IS_PROD,
    	sourceMap: false,
      // pass options to sass-loader
      scss: {
       	// 向全局sass样式传入共享的全局变量, $src可以配置图片cdn前缀
        // 详情: https://cli.vuejs.org/guide/css.html#passing-options-to-pre-processor-loaders
        prependData: `
          @import "@scss/config.scss";
          @import "@scss/variables.scss";
          @import "@scss/mixins.scss";
          @import "@scss/utils.scss";
          $src: "${process.env.VUE_APP_OSS_SRC}";
          `
      }
    }
  },
}
```

在 scss 中引用

```css
.home {
  background: url($src+'/images/500.png');
}
```

### 为 stylus 提供全局变量

`npm i -D style-resources-loader`

```js
const path = require('path')
const resolve = dir => path.resolve(__dirname, dir)
const addStylusResource = rule => {
  rule
    .use('style-resouce')
    .loader('style-resources-loader')
    .options({
      patterns: [
        resolve('src/assets/stylus/variable.styl')
      ]
    })
}
module.exports = {
  chainWebpack: config => {
    const types = ['vue-modules', 'vue', 'normal-modules', 'normal']
    types.forEach(type =>
      addStylusResource(config.module.rule('stylus').oneOf(type))
    )
  }
}
```


### 删除 moment 语言包

> 删除 moment 除 zh-cn 中文包外的其它语言包，无需在代码中手动引入 zh-cn 语言包

```js
const webpack = require('webpack')

module.exports = {
  chainWebpack: config => {
    config
      .plugin('ignore')
      .use(new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /zh-cn$/))

    return config
  }
}
```



### 预渲染 prerender-spa-plugin

`npm i -D prerender-spa-plugin`

```js
const path = require('path')
const PrerenderSpaPlugin = require('prerender-spa-plugin')
const routesConfig = require('./router-config')
const resolve = dir => path.join(__dirname, dir)
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)

module.exports = {
  configureWebpack: config => {
    const plugins = []

    if (IS_PROD) {
      // 预加载
      plugins.push(
        new PrerenderSpaPlugin({
          staticDir: resolve('dist'),
          routes: Object.keys(routesConfig),
          postProcess(ctx) {
            ctx.route = ctx.originalRoute
            ctx.html = ctx.html.split(/>[\s]+</gim).join('><')
            ctx.html = ctx.html.replace(
              /<title>(.*?)<\/title>/gi,
              `<title>${routesConfig[ctx.route].title}</title><meta name="keywords" content="${routesConfig[ctx.route].keywords}" /><meta name="description" content="${routesConfig[ctx.route].description}" />`
            )
            if (ctx.route.endsWith('.html')) {
              ctx.outputPath = path.join(__dirname, 'dist', ctx.route)
            }
            return ctx
          },
          minify: {
            collapseBooleanAttributes: true,
            collapseWhitespace: true,
            decodeEntities: true,
            keepClosingSlash: true,
            sortAttributes: true
          },
          renderer: new PrerenderSpaPlugin.PuppeteerRenderer({
            // 需要注入一个值，这样就可以检测页面当前是否是预渲染的
            inject: {},
            headless: false,
            // 视图组件是在API请求获取所有必要数据后呈现的，因此我们在dom中存在“data view”属性后创建页面快照
            renderAfterDocumentEvent: 'render-event'
          })
        })
      )
    }

    config.plugins = [...config.plugins, ...plugins]
  }
}
```

`mounted()`中添加 `document.dispatchEvent(new Event('render-event'))`

```js
new Vue({
  router,
  store,
  render: h => h(App),
  mounted() {
    document.dispatchEvent(new Event('render-event'))
  }
}).$mount('#app')
```

**为自定义预渲染页面添加自定义 title、description、content**

- 删除 public/index.html 中关于 description、content 的 meta 标签。保留 title 标签
- 配置 router-config.js

```js
module.exports = {
  '/': {
    title: '首页',
    keywords: '首页关键词',
    description: '这是首页描述'
  },
  '/about.html': {
    title: '关于我们',
    keywords: '关于我们页面关键词',
    description: '关于我们页面关键词描述'
  }
}
```

### 添加 IE 兼容

`npm i -S @babel/polyfill`

在 main.js 中添加

```js
import '@babel/polyfill'
```

配置 `babel.config.js`

```js
const plugins = []

module.exports = {
  presets: [['@vue/app', { useBuiltIns: 'entry' }]],
  plugins: plugins
}
```

### 去除 `console.log`

正常情况下我们会在开发环境进行`console`调试，但是如果不删除，过多会出现内存泄漏的情况

**方法一：**

`npm install -D uglifyjs-webpack-plugin`

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
module.exports = {
    configureWebpack: config => {
        if (IS_PROD) {
            const plugins = [];
            plugins.push(
                new UglifyJsPlugin({
                    uglifyOptions: {
                        compress: {
                            warnings: false,
                            drop_console: true,
                            drop_debugger: false,
                            pure_funcs: ['console.log'] // 移除console
                        }
                    },
                    sourceMap: false,
                    parallel: true
                })
            );
            config.plugins = [
                ...config.plugins,
                ...plugins
            ];
        }
    }
}
```

如果使用 uglifyjs-webpack-plugin 会报错，可能存在 node_modules 中有些依赖需要 babel 转译。

而 vue-cli 的[transpileDependencies](https://cli.vuejs.org/zh/config/#transpiledependencies)配置默认为[], babel-loader 会忽略所有 node_modules 中的文件。如果你想要通过 Babel 显式转译一个依赖，可以在这个选项中列出来。配置需要转译的第三方库。

**方法二：**
 `npm i --save-dev babel-plugin-transform-remove-console`

```js
// babel.config.js

const plugins = [];
if(['production', 'prod'].includes(process.env.NODE_ENV)) {  
  plugins.push("transform-remove-console")
}

module.exports = {
  presets: [["@vue/app",{"useBuiltIns": "entry"}]],
  plugins: plugins
};
```

### 去除多余无效的 css

> 谨慎使用。可能出现各种样式丢失现象。

- 方案一：`npm i -D postcss-import @fullhuman/postcss-purgecss`

  更新 postcss.config.js

  ```js
  const autoprefixer = require('autoprefixer')
  const postcssImport = require('postcss-import')
  const purgecss = require('@fullhuman/postcss-purgecss')
  const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
  let plugins = []
  if (IS_PROD) {
    plugins.push(postcssImport)
    plugins.push(
      purgecss({
        content: [
          './layouts/**/*.vue',
          './components/**/*.vue',
          './pages/**/*.vue'
        ],
        extractors: [
          {
            extractor: class Extractor {
              static extract(content) {
                const validSection = content.replace(
                  /<style([\s\S]*?)<\/style>+/gim,
                  ''
                )
                return (
                  validSection.match(/[A-Za-z0-9-_/:]*[A-Za-z0-9-_/]+/g) || []
                )
              }
            },
            extensions: ['html', 'vue']
          }
        ],
        whitelist: ['html', 'body'],
        whitelistPatterns: [
          /el-.*/,
          /-(leave|enter|appear)(|-(to|from|active))$/,
          /^(?!cursor-move).+-move$/,
          /^router-link(|-exact)-active$/
        ],
        whitelistPatternsChildren: [/^token/, /^pre/, /^code/]
      })
    )
  }
  module.exports = {
    plugins: [...plugins, autoprefixer]
  }
  ```

* 方案二: `npm i -D glob-all purgecss-webpack-plugin`	

  ```js
  const path = require('path')
  const glob = require('glob-all')
  const PurgecssPlugin = require('purgecss-webpack-plugin')
  const resolve = dir => path.join(__dirname, dir)
  const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)
  
  module.exports = {
    configureWebpack: config => {
      const plugins = []
      if (IS_PROD) {
        plugins.push(
          new PurgecssPlugin({
            paths: glob.sync([resolve('./**/*.vue')]),
            extractors: [
              {
                extractor: class Extractor {
                  static extract(content) {
                    const validSection = content.replace(
                      /<style([\s\S]*?)<\/style>+/gim,
                      ''
                    )
                    return (
                      validSection.match(/[A-Za-z0-9-_/:]*[A-Za-z0-9-_/]+/g) || []
                    )
                  }
                },
                extensions: ['html', 'vue']
              }
            ],
            whitelist: ['html', 'body'],
            whitelistPatterns: [
              /el-.*/,
              /-(leave|enter|appear)(|-(to|from|active))$/,
              /^(?!cursor-move).+-move$/,
              /^router-link(|-exact)-active$/
            ],
            whitelistPatternsChildren: [/^token/, /^pre/, /^code/]
          })
        )
      }
      config.plugins = [...config.plugins, ...plugins]
    }
  }
  ```

### 修复 HMR(热更新)失效

```js
module.exports = {
  chainWebpack: config => {
    // 修复HMR
    config.resolve.symlinks(true)
  }
}
```

### 修复 Lazy loading routes Error： Cyclic dependency [vuejs/vue-cli#1669](https://github.com/vuejs/vue-cli/issues/1669)

```js
module.exports = {
  chainWebpack: config => {
    config.plugin('html').tap(args => {
      args[0].chunksSortMode = 'none'
      return args
    })
  }
}
```

### 自动生成雪碧图

默认 src/assets/icons 中存放需要生成雪碧图的 png 文件。首次运行 npm run serve/build 会生成雪碧图，并在跟目录生成 icons.json 文件。再次运行命令时，会对比 icons 目录内文件与 icons.json 的匹配关系，确定是否需要再次执行 webpack-spritesmith 插件。

`npm i -D webpack-spritesmith`

```js
const SpritesmithPlugin = require('webpack-spritesmith')
const path = require('path')
const fs = require('fs')

let has_sprite = true

try {
  let result = fs.readFileSync(path.resolve(__dirname, './icons.json'), 'utf8')
  result = JSON.parse(result)
  const files = fs.readdirSync(path.resolve(__dirname, './src/assets/icons'))
  has_sprite =
    files && files.length
      ? files.some(item => {
          let filename = item.toLocaleLowerCase().replace(/_/g, '-')
          return !result[filename]
        })
      : false
} catch (e) {
  has_sprite = false
}

// 雪碧图样式处理模板
const SpritesmithTemplate = function(data) {
  // pc
  let icons = {}
  let tpl = `.ico { 
  display: inline-block; 
  background-image: url(${data.sprites[0].image}); 
  background-size: ${data.spritesheet.width}px ${data.spritesheet.height}px; 
}`

  data.sprites.forEach(sprite => {
    const name = '' + sprite.name.toLocaleLowerCase().replace(/_/g, '-')
    icons[`${name}.png`] = true
    tpl = `${tpl} 
.ico-${name}{
  width: ${sprite.width}px; 
  height: ${sprite.height}px; 
  background-position: ${sprite.offset_x}px ${sprite.offset_y}px;
}
`
  })

  fs.writeFile(
    path.resolve(__dirname, './icons.json'),
    JSON.stringify(icons, null, 2),
    (err, data) => {}
  )

  return tpl
}

module.exports = {
  configureWebpack: config => {
    const plugins = []
    if (has_sprite) {
      plugins.push(
        new SpritesmithPlugin({
          src: {
            cwd: path.resolve(__dirname, './src/assets/icons/'), // 图标根路径
            glob: '**/*.png' // 匹配任意 png 图标
          },
          target: {
            image: path.resolve(__dirname, './src/assets/images/sprites.png'), // 生成雪碧图目标路径与名称
            // 设置生成CSS背景及其定位的文件或方式
            css: [
              [
                path.resolve(__dirname, './src/assets/scss/sprites.scss'),
                {
                  format: 'function_based_template'
                }
              ]
            ]
          },
          customTemplates: {
            function_based_template: SpritesmithTemplate
          },
          apiOptions: {
            cssImageRef: '../images/sprites.png' // css文件中引用雪碧图的相对位置路径配置
          },
          spritesmithOptions: {
            padding: 2
          }
        })
      )
    }
    config.plugins = [...config.plugins, ...plugins]
  }
}
```

### 文件上传 ali oss

开启文件上传 ali oss，需要将 publicPath 改成 ali oss 资源 url 前缀,也就是修改 VUE_APP_PUBLIC_PATH。具体配置参见[webpack-oss](https://github.com/staven630/webpack-oss)

`npm i -D webpack-oss`

```js
const AliOssPlugin = require('webpack-oss')
const IS_PROD = ['production', 'prod'].includes(process.env.NODE_ENV)

const format = AliOssPlugin.getFormat()

module.exports = {
  publicPath: IS_PROD ? `${process.env.VUE_APP_PUBLIC_PATH}/${format}` : './', // 默认'/'，部署应用包时的基本 URL
  configureWebpack: config => {
    const plugins = []

    if (IS_PROD) {
      plugins.push(
        new AliOssPlugin({
          accessKeyId: process.env.ACCESS_KEY_ID,
          accessKeySecret: process.env.ACCESS_KEY_SECRET,
          region: process.env.REGION,
          bucket: process.env.BUCKET,
          prefix: process.env.PREFIX,
          exclude: /.*\.html$/,
          format
        })
      )
    }
    config.plugins = [...config.plugins, ...plugins]
  }
}
```



