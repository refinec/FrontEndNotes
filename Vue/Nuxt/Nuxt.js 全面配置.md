## 总体配置

```
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

