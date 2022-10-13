## 总体配置

```js

const isDev = process.env.NODE_ENV !== 'production'

module.exports = {
    // rootDir: '', // 设置 Nuxt 应用的根目录
    srcDir: 'src',  // 用于配置应用的源码目录路径(默认值为rootDir), 即pages、components等目录应在该目录下
    target: 'server', // 渲染目标 server 或者 static
    // 定义服务器访问主机和端口,https,socket...
    server: {
        host: process.env.HOST || '0.0.0.0',
        port: process.env.PORT || (isDev ? 3000 : 8080),
        // https: {
        //     key: fs.readFileSync(path.resolve(__dirname, 'server.key')),
        //     cert: fs.readFileSync(path.resolve(__dirname, 'server.crt'))
        // },
        // socket: '/tmp/nuxt.socket',
        // timing: false,
    },
    // 可以使用 $config 从客户端和服务器访问此对象的值。
    publicRuntimeConfig: {
        nodeEnv: process.env.NODE_ENV,
    },
    // 可以使用 $config 从服务器访问此对象的值。
    privateRuntimeConfig: {
        axios: {
            baseURL: process.env.API_ENDPOINT_LOCAL || process.env.API_ENDPOINT,
        },
    },
    // 定义在客户端和服务端共享的环境变量
    env: {
        baseUrl: process.env.BASE_URL || 'http://localhost:3000'
    },
    // 配置应用的 meta 信息
    head: {
        titleTemplate: '%s - Nuxt.js',
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width, initial-scale=1' },
            { hid: 'description', name: 'description', content: 'Meta description' }
        ]
    },
    // 配置全局的 CSS 文件、模块、库。 (每个页面都会被引入)
    css: [
        // 直接加载一个 Node.js 模块。（在这里它是一个 Sass 文件）
        'bulma',
        // 项目里要用的 CSS 文件
        '@/assets/css/main.css',
        // 项目里要使用的 SCSS 文件
        '@/assets/css/main.scss',
        'ant-design-vue/dist/antd.less',
    ],
    // 自动扫描并导入组件
    components: false,
    // 加载进度条配置
    loading: false,
    // loading: {},
    // loadingIndicator: {
    //     name: 'circle',
    //     color: '#3B8070',
    //     background: 'white'
    // },
    build: {
        // Nuxt.js 允许您将dist文件上传到 CDN 来获得最快渲染性能，只需将publicPath设置为 CDN 即可。
        publicPath: 'https://cdn.nuxtjs.org',


        // 打包分析，Nuxt 使用 webpack-bundle-analyzer
        // 可通过 nuxt build --analyze 或 nuxt build -a 命令来启用该分析器进行编译构建
        analyze: true,

        // 为 JS 和 Vue 文件设定自定义的 babel 配置
        babel: {
            babelrc: false,
            cacheDirectory: undefined,
            presets: ['@nuxt/babel-preset-app']
        },

        // 启用缓存
        cache: false,

        // 在生成的 HTML 中的 <link rel="stylesheet"> 和 <script> 标记上配置 crossorigin 属性。
        crossorigin: "",

        // true 为开发模式(development)， false 为生产模式(production)。
        cssSourceMap: true,

        // 是否允许 vue-devtools 调试
        devtools: false,

        // 
        extend(config, { isClient, loaders: { vue } }) {
            // 为 客户端打包 进行扩展配置
            if (isClient) {
                config.devtool = 'eval-source-map'
                vue.transformAssetUrls.video = ['src', 'poster']
            }
        },

        // 使用 Vue 服务器端渲染指南启用常见 CSS 提取
        extractCSS: false,

        // 自定义打包文件名, 以下是默认值
        filenames: {
            app: ({ isDev }) => isDev ? '[name].js' : '[contenthash].js',
            chunk: ({ isDev }) => isDev ? '[name].js' : '[contenthash].js',
            css: ({ isDev }) => isDev ? '[name].css' : '[contenthash].css',
            img: ({ isDev }) => isDev ? '[path][name].[ext]' : 'img/[contenthash:7].[ext]',
            font: ({ isDev }) => isDev ? '[path][name].[ext]' : 'fonts/[contenthash:7].[ext]',
            video: ({ isDev }) => isDev ? '[path][name].[ext]' : 'videos/[contenthash:7].[ext]'
        },

        // 以下是默认值
        optimization: {
            minimize: true,
            minimizer: [
                // terser-webpack-plugin
                // optimize-css-assets-webpack-plugin
            ],
            splitChunks: {
                chunks: 'all',
                automaticNameDelimiter: '.',
                name: undefined,
                cacheGroups: {}
            }
        },

        // 以下是默认值
        terser: {
            parallel: true,
            cache: false,
            sourceMap: false,
            extractComments: {
                filename: 'LICENSES'
            },
            terserOptions: {
                output: {
                    comments: /^\**!|@preserve|@license|@cc_on/
                }
            }
        },

        parallel: false,

        // 配置 Webpack 插件
        plugins: [
            new webpack.DefinePlugin({
                'process.VERSION': version
            })
        ],

        // 以下默认值
        postcss: {
            plugins: {
                'postcss-import': {},
                'postcss-url': {},
                'postcss-preset-env': {},
                'cssnano': { preset: 'default' } // disabled in dev mode
            }
        },

        // 代码分模块
        splitChunks: {
            layouts: false,
            pages: true,
            commons: true
        },

        // 为服务器端渲染创建特殊的 webpack 包。
        // true 为通用模式，false 为spa模式
        ssr: true,

        // 注入一些公共CSS样式
        modules: ['@nuxtjs/style-resources'],
        styleResources: {
            scss: './assets/variables.scss',
        },

        // 自定义模板, 这些模板将基于 Nuxt 配置进行渲染。
        // templates: {
        //     src: '~/modules/support/plugin.js', // `src` 可以是绝对的或相对的路径
        //     dst: 'support.js', // `dst` 是相对于项目`.nuxt`目录
        //     options: {
        //         // 选项`options`选项可以将参数提供给模板
        //         live_chat: false
        //     }
        // },

        // 使用 Babel 与特定的依赖关系进行转换
        transpile: [/ant-design-vue/],

        // 使用watch来监听文件更改。此功能特别适用用在modules中。
        watch: ['~/.nuxt/support.js'],

        // 默认情况下，生成过程不扫描符号链接中的文件。这个布尔值包含它们，因此允许在文件夹(例如“ page”文件夹)中使用符号链接
        followSymlinks: true,

        // 控制部分构建信息输出日志
        // quiet: false,

        // profile: false,

        // optimizeCSS: false


        // 自定义 webpack 加载器
        // loaders: {}

        // 友好错误提示
        // friendlyErrors: true

        // devMiddleware: {},
        // hardSource: false,
        // hotMiddleware: {},
    },
    // watchers 配置
    watchers: {
        webpack: {
            ignored: /node_modules/,
            // aggregateTimeout: 300, // 默认值
            // poll: 1000 // 默认值
        }
    },
    // 核心功能扩展
    modules: [
        // Using package name
        '@nuxtjs/axios',
        '@nuxtjs/style-resources',
        '@nuxtjs/device',

        // Relative to your project srcDir
        '~/modules/awesome.js',

        // Providing options
        ['@nuxtjs/google-analytics', { ua: 'X1234567' }],

        // Inline definition
        function () { }
    ],
    // 个性化配置 Nuxt.js 应用的路由
    router: {
        middleware: []
    },
    proxy: {
        '/api/v2/talent-broker/': {
            target: 'https://opcenter.shiyanlou.com',
            pathRewrite: { '^/api/v2/talent-broker/': '/api/v1/talent-broker/' },
            headers: {
                Connection: 'keep-alive',
            },
            onProxyReq,
        },
        '/api/v2/apifox/': {
            target: 'http://127.0.0.1:4523/m1/400770-0-default',
            pathRewrite: { '^/api/v2/apifox/': '/api/v2/' },
            headers: {
                Connection: 'keep-alive',
            },
            onProxyReq,
        },
        '/api/v2/opcenter/': {
            target: 'https://opcenter.shiyanlou.com',
            pathRewrite: { '^/api/v2/opcenter/': '/api/v1/' },
            headers: {
                Connection: 'keep-alive',
            },
            onProxyReq,
        },
        '/api/v2': {
            target: process.env.PROXY_API || 'https://staging.shiyanlou.com',
            headers: {
                Connection: 'keep-alive',
            },
            onProxyReq,
        },
    },

    buildDir: '.nuxt', // 定义.nuxt(默认)目录

    mode: 'universal',  // nuxt模式

    // 用于配置那些需要在 根vue.js应用 实例化之前需要运行的 Javascript 插件
    plugins: ['~/plugins/ant-ui.js'],

    // 设置 node_modules 目录的路径
    // modulesDir: ['node_modules'],

    // 设置页面切换过渡效果的默认属性值
    transition: {
        name: 'page',
        mode: 'out-in'
    },

    // 设置布局切换过渡效果的默认属性值
    layoutTransition: {
        name: 'layout',
        mode: 'out-in'
    },

    // 配置 Nuxt.js 应用生成静态站点的具体方式
    // generate: {
    //     dir: 'dist', // nuxt generate 生成的目录名称
    //     devtools: false, // 是否允许 vue-devtools 调试
    //     fallback: '200.html', // 在将生成的站点部署到静态主机时，可以使用此文件。它将回退到模式：mode:'spa'。
    //     interval: 0, // 两个渲染周期之间的间隔，以避免使用来自 Web 应用程序的 API 调用互相干扰
    //     routes: [],
    //     subFolders: true,
    //     concurrency: 500,
    // },

    // Vue.config 配置
    vue: {
        config: {
            productionTip: true,
            devtools: false
        }
    },

    // 定义主 HTML 模板中使用的全局 ID 以及主 Vue 实例名称和其他选项。
    globalName: 'myCustomName',

    // 配置 Nuxt.js 应用是开发模式还是生产模式,但dev 属性的值会被 nuxt 命令 覆盖
    // dev: true,

    // cli: {
    //     bannerColor: 'yellow'
    // },

    // 定义 Nuxt.js 应用程序的自定义目录
    // dir: {
    //     assets: 'assets',
    //     layouts: 'layouts',
    //     middleware: 'middleware',
    //     pages: 'pages',
    //     static: 'static',
    //     store: 'store'
    // },

    // 扩展插件
    // extendPlugins(plugins) {
    //     const pluginIndex = plugins.findIndex(
    //         ({ src }) => src === '~/plugins/shouldBeFirst.js'
    //     )
    //     const shouldBeFirstPlugin = plugins[pluginIndex]

    //     plugins.splice(pluginIndex, 1)
    //     plugins.unshift(shouldBeFirstPlugin)

    //     return plugins
    // },

    // 打包版本：'client' | 'server' or true | false(默认不开启)
    //modern: false,

    // hooks 是Nuxt 事件的监听器，这些事件通常在 Nuxt 模块中使用，但也可以在 nuxt.config.js 中使用。
    hooks: {
        build: {
            done(builder) {
                const extraFilePath = path.join(
                    builder.nuxt.options.buildDir,
                    'extra-file'
                )
                fs.writeFileSync(extraFilePath, 'Something extra')
            }
        }
    },

    // 忽略匹配在ignore内指定的ignore模式的所有文件。
    ignore: ['**/*.test.*', '**/*.spec.*'],


    // 服务器端渲染中间件, 类似node express的路由中间件
    serverMiddleware: [
        // Will register redirect-ssl npm package
        'redirect-ssl',

        // Will register file from project api directory to handle /api/* requires
        { path: '/api', handler: '~/api/index.js' },

        // We can create custom instances too
        { path: '/static2', handler: serveStatic(__dirname + '/static2') }
    ],

    // 遥测nuxt特性使用数据
    // telemetry: false,

    // 允许您监听自定义文件来重新启动服务器
    //watch: ['~/custom/*.js'],

}
```

### 环境变量配置

可以配置在客户端和服务端共享的环境变量

```js
module.exports = {
  env: {
    baseUrl: process.env.BASE_URL || 'http://localhost:3000'
  }
}
```

通过以下两种方式来使用 baseUrl 变量

1. 通过 `process.env.baseUrl`
2. 通过 `context.baseUrl`，请参考[context api](https://zh.nuxtjs.org/api/#上下文对象)

### 提供全局 scss 变量

- 方法一：

  ```
  npm i -S @nuxtjs/style-resources
  npm i -D sass-loader node-sass
  ```

  修改 nuxt.config.js
  ```js
  export default {
    modules: [
      '@nuxtjs/style-resources',
    ],
    styleResources: {
      scss: '~/assets/scss/variable.scss'
    }
  }
  ```

- 方法二：

  ```
  npm i -D nuxt-sass-resources-loader sass-loader node-sass
  ```

  修改 nuxt.config.js

  ```js
  export default {
    modules: [
      ['nuxt-sass-resources-loader', ['~/assets/scss/variable.scss']]
    ],
    styleResources: {
      scss: '~/assets/scss/variable.scss'
    }
  }
  ```

### 按需引入 element-ui

```
npm i -D babel-plugin-component
// or
yarn add -D babel-plugin-component
```

修改 nuxt.config.js

```js
module.exports = {
  plugins: ['@/plugins/element-ui'],
  build: {
    babel: {
      plugins: [
        [
          'component',
          { libraryName: 'element-ui', styleLibraryName: 'theme-chalk' }
        ]
      ]
    }
  },
}
```

修改 `plugins/element-ui.js`

```js
import Vue from 'vue'
import {
  Button, Loading, Notification, Message, MessageBox
} from 'element-ui'

import lang from 'element-ui/lib/locale/lang/zh-CN'
import locale from 'element-ui/lib/locale'

// configure language
locale.use(lang)

// set
Vue.use(Loading.directive)
Vue.prototype.$loading = Loading.service
Vue.prototype.$msgbox = MessageBox
Vue.prototype.$alert = MessageBox.alert
Vue.prototype.$confirm = MessageBox.confirm
Vue.prototype.$prompt = MessageBox.prompt
Vue.prototype.$notify = Notification
Vue.prototype.$message = Message

// import components
Vue.use(Button);
// or
// Vue.component(Button.name, Button)
```

### Brotli 压缩

`npm i shrink-ray-current`

若安装失败,请先用管理员身份安装以下全局依赖

```
npm install --global windows-build-tools
或
yarn global add windows-build-tools
```

修改 nuxt.config.js

```js
export default {
  render: {
    http2: {
      push: true
    },
    compressor: shrinkRay()
  }
}
```

### 打包分析

package.json 中添加 analyze 命令

```js
"analyze": "nuxt build --analyze"
```

修改 `nuxt.config.js`

```js
export default {
  build: {
    analyza: {
      analyzeMode: 'static'
    }
  }
}
```

### 配置 hard-source-webpack-plugin

`npm i -D hard-source-webpack-plugin`

修改 nuxt.config.js

```js
module.exports = {
  build: {
    extractCSS: true,
    extend(config, ctx) {
      if (ctx.isDev) {
        config.plugins.push(
          new HardSourceWebpackPlugin({
            cacheDirectory: '.cache/hard-source/[confighash]'
          })
        )
      }
    }
  }
}
```

### 去除多余 css

```
npm i --D glob-all purgecss-webpack-plugin
```

若安装失败,请先用管理员身份安装以下全局依赖

```
npm install --global windows-build-tools
或
yarn global add windows-build-tools
```

修改 nuxt.config.js

```js
const PurgecssPlugin = require('purgecss-webpack-plugin')
const glob = require('glob-all')
const path = require('path')
const resolve = dir => path.resolve(__dirname, dir);

module.exports = {
  build: {
    extractCSS: true,
    extend(config, ctx) {
      if (!ctx.isDev) {
        config.plugins.push(
          new PurgecssPlugin({
            paths: glob.sync([
              resolve('./pages/**/*.vue'),
              resolve('./layouts/**/*.vue'),
              resolve('./components/**/*.vue')
            ]),
            extractors: [
              {
               extractor: class Extractor {
                  static extract(content) {
                    const validSection = content.replace(
                      /<style([\s\S]*?)<\/style>+/gim,
                      ""
                    );
                    return validSection.match(/[A-Za-z0-9-_:/]+/g) || [];
                  }
                },
                extensions: ['vue']
              }
            ],
            whitelist: ['html', 'body', 'nuxt-progress']
          })
        )
      }
    }
  }
}
```

