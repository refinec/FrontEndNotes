### 总体配置

```js
// vite.config.ts

import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import path from 'path'
import vueSetupExtend from 'vite-plugin-vue-setup-extend'
const nodeResolve = (dir) => path.resolve(__dirname, '.', dir)

export default ({ mode }: ConfigEnv): UserConfig => {
    const plugins = [
        vueSetupExtend() // SFC 支持 name 属性
    ]
    const resolve = {
        // 路径别名
        alias: { 
            '@': nodeResolve('src'),
            '~': nodeResolve('public'),
        },
    }

    return {
        plugins,
        resolve,
    }
}
```

### 定义全局变量

通常情况下，在页面中会用到一些常量，而这些常量可能后期又会发生变更，那么这类常量就不能在代码中使用硬编码处理。

最基础的做法是在构建时生成全局常量，使用时引用全局常量。

方法一：使用 **`define`** 定义全局变量

```ts
// vite.config.ts

export default defineConfig({
  define: {
    INITIAL_COUNT: 10,
  },
})
```

在代码中使用：

```js
const count = ref(INITIAL_COUNT);
```

方法二：[使用 env 文件定义环境变量](https://cn.vitejs.dev/guide/env-and-mode.html)

> Vite 使用[dotenv](https://github.com/motdotla/dotenv)可以加载自定义的环境变量。

以下开发的 dev、stage、prod 三种环境为例。将新建四种 env 文件

```sh
.env         # 存放不同环境共用的环境变量，定义的变量将被各环境共享
.env.dev     # 开发环境
.env.stage   # 测试环境
.env.prod    # 正式环境
```

- `.env`

  ```sh
  VITE_TOKEN_NAME = 'token'
  ```

- `.env.dev`

  ```sh
  NODE_ENV = development
  VITE_BASE_URL = http://wwww.demo.com/api
  VITE_BASE_PATH = /
  ```

- `.env.stage`

  ```sh
  NODE_ENV = production
  VITE_BASE_URL = http://wwww.stage.com/api
  VITE_BASE_PATH = /stage
  ```

- `.env.prod`

  ```sh
  NODE_ENV = production
  
  VITE_BASE_URL = http://wwww.prod.com/api
  
  VITE_BASE_PATH = /prod
  ```

vite 的`--mode` 选项，会读取指定的值匹配的环境变量，如运行 `vite --mode dev` 时，.env 和.env.dev 两个环境变量文件将被加载

package.json

```json
"scripts": {
  "dev": "vite --mode dev",
  "stage": "vue-tsc --noEmit && vite build --mode stage",
  "prod": "vue-tsc --noEmit && vite build --mode prod",
},
```

vite.config.ts

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [vue()]

  return {
    plugins,
  }
}
```

在 `vite.config.ts` 中可以使用 `process.env` 上挂载的变量。

项目中，Vite 会在一个特殊的 **`import.meta.env`** 对象上暴露环境变量。默认以"**VITE_**"开头的变量，将被挂载在 `import.meta.env` 对象上。

在 `src/env.d.ts` 中添加以下信息,可实现环境变量代码自动提示：

```ts
interface ImportMetaEnv {
  VITE_BASE_URL: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

###  配置路径别名 alias

```ts
// vite.config.ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import path from 'path'

const nodeResolve = (dir) => path.resolve(__dirname, '.', dir)

export default ({ mode }: ConfigEnv): UserConfig => {
  const resolve = {
    alias: {
      '@': nodeResolve('src'),
      '~': nodeResolve('public'),
    },
  }

  return {
    resolve,
  }
}
```

使用 typescript 开发，如果出现找不到模块“path”或其相应的类型声明。则需要安装@types/node：`npx pnpm i -D @types/node`

修改 tsconfig.json 配置：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "importHelpers": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

这样就可以使用'@'来替代相对路径对组件进行引用：

```vue
<script lang="ts">import HelloWorld from '@/components/HelloWorld.vue'</script>
```

也可以使用以下方式配置 alias：

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import path from 'path'

const nodeResolve = (dir) => path.resolve(__dirname, '.', dir)

export default ({ mode }: ConfigEnv): UserConfig => {
  return {
    resolve: {
      alias: [
        {
          find: /@\//,
          replacement: `${nodeResolve('src')}/`,
        },
        {
          find: /@comps\//,
          replacement: `${nodeResolve('src/components')}/`,
        },
      ],
    },
  }
}
```

### 手动分包(第三方包和业务代码)

```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      // 1. 对象的形式，一个属性名就是一个打包结果
      manualChunks: {
        "xxx": ["lodash", "vue"]
      },
      // 2. 函数形式
      manualChunks(id) { // id 表示模块ID，没有返回则表示所有内容都放到index-xxxxx.js的打包结果中
        // 第三方库都打包到vendor-xxxx.js文件中
        if (id.includes('node_modules')) {
          return 'vendor';
        }
      }
    }
  }
})
```



### SFC 支持 name 属性

 [vite-plugin-vue-setup-extend](https://github.com/anncwb/vite-plugin-vue-setup-extend)支持`<script setup>`新增 name 属性

`npx pnpm i -D vite-plugin-vue-setup-extend`

```ts
// vite.config.ts
import vueSetupExtend from 'vite-plugin-vue-setup-extend'

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [vueSetupExtend()]

  return {
    plugins,
  }
}
```

### 提供全局 less、scss 变量

#### 全局 less 变量

* 直接提供变量

  ```ts
  export default ({ mode }: ConfigEnv): UserConfig => {
    const css = {
      preprocessorOptions: {
        less: {
          additionalData: `@injectedColor: red;`,
          javascriptEnabled: true,
        },
      },
    }
    return {
      css,
    }
  }
  ```

* 通过导入 less 文件提供变量

  ```ts
  export default ({ mode }: ConfigEnv): UserConfig => {
    const css = {
      preprocessorOptions: {
        less: {
          additionalData: '@import "@/assets/less/variables.less";',
          javascriptEnabled: true,
        },
      },
    }
  
    return {
      css,
    }
  }
  ```

#### 全局 scss 变量

- 直接提供变量

  ```ts
  export default ({ mode }: ConfigEnv): UserConfig => {
    const css = {
      preprocessorOptions: {
        less: {
          additionalData: `$injectedColor: orange;`,
          javascriptEnabled: true,
        },
      },
    }
    return {
      css,
    }
  }
  ```

- 通过导入 scss 文件提供变量

  ```ts
  export default ({ mode }: ConfigEnv): UserConfig => {
    const css = {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/assets/scss/variables.scss";`,
          javascriptEnabled: true,
        },
      },
    }
  
    return {
      css,
    }
  }
  ```

### 按需加载 ElementPlus、Ant Design Vue

[unplugin-vue-components](https://github.com/antfu/unplugin-vue-components)

#### 按需引入 ElementPlus

> [unplugin-element-plus](https://github.com/element-plus/unplugin-element-plus)为 Element Plus 按需引入样式。

`npx pnpm i -D unplugin-vue-components unplugin-element-plus`

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'

import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import ElementPlus from 'unplugin-element-plus/vite'

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [
    Components({
      resolvers: [ElementPlusResolver()],
    }),
    ElementPlus({}),
  ]

  return {
    plugins,
  }
}
```

#### 按需引入 Ant Design Vue

`npx pnpm i -D unplugin-vue-components`

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'

import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [
    Components({
      resolvers: [AntDesignVueResolver()],
    }),
  ]
  return {
    plugins,
  }
}
```

### 打包分析

[rollup-plugin-visualizer](https://github.com/btd/rollup-plugin-visualizer)

`npx pnpm i -D rollup-plugin-visualizer`

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import { visualizer } from 'rollup-plugin-visualizer'

export default ({ mode }: ConfigEnv): UserConfig => {
  const IS_PROD = ['prod', 'production'].includes(mode)

  let plugins = []

  if (IS_PROD) {
    plugins = [...plugins, visualizer()]
  }

  return {
    plugins,
  }
}
```

### ESlint 错误显示在浏览器中

 [vite-plugin-eslint](https://github.com/gxmari007/vite-plugin-eslint)

`npx pnpm i -D vite-plugin-eslint`

```ts
// vite.config.ts

import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import eslintPlugin from 'vite-plugin-eslint'

export default ({ mode }: ConfigEnv): UserConfig => {
  const IS_PROD = ['prod', 'production'].includes(mode)

  let plugins = []

  if (!IS_PROD) {
    plugins = [
      ...plugins,
      eslintPlugin({
        cache: false,
        include: ['src/**/*.vue', 'src/**/*.ts', 'src/**/*.tsx'],
      }),
    ]
  }

  return {
    plugins,
  }
}
```

### 提供 externals

[vite-plugin-externals](https://github.com/crcong/vite-plugin-externals):为 Vite 提供 commonjs 外部支持

`npx pnpm i -D vite-plugin-externals`

```ts
// vite.config.ts

import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import { viteExternalsPlugin } from 'vite-plugin-externals'

export default ({ mode }: ConfigEnv): UserConfig => {
  const IS_PROD = ['prod', 'production'].includes(mode)

  let plugins = []

  if (IS_PROD) {
    plugins = [
      ...plugins,
      viteExternalsPlugin({
        vue: 'Vue',
        react: 'React',
        'react-dom': 'ReactDOM',
        // value support chain, tranform to window['React']['lazy']
        lazy: ['React', 'lazy'],
      }),
    ]
  }

  return {
    plugins,
  }
}

```

### 提供全局 less、scss 变量



### 配置代理 Proxy

Vitejs 的开发服务器选项https://cn.vitejs.dev/config/#server-host

```ts
export default ({ mode }: ConfigEnv): UserConfig => {
  const server = {
    host: '0.0.0.0',
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://192.168.1.163:8081/',
        changeOrigin: true,
        rewrite: (url) => url.replace(/^\/api/, ''),
      },
    },
  }

  return {
    server,
  }
}
```

- target

  实际的后端 api 地址。如请求/api/getUserInfo 会转发到http://192.168.1.163:8081/api/getUserInfo。

- changeOrigin

  是否改写 origin。设为 true，则请求 header 中 origin 将会与 target 配置项的域名一致。

- rewrite

  重写转发的请求链接。

### 使用 TSX/JSX

 [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx)通过 HMR 提供 Vue 3 JSX 和 TSX 支持。

`npx pnpm i @vitejs/plugin-vue-jsx`

```ts
// vite.config.ts
import vueJsx from '@vitejs/plugin-vue-jsx'

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [vueJsx()]

  return {
    plugins,
  }
}
```

修改 tsconfig.json 配置，使.tsx 中支持 JSX

```json
{
  "compilerOptions": {
    "jsx": "preserve" // .tsx中支持JSX
  }
}
```

### 生成雪碧图

[vite-plugin-svg-icons](https://github.com/anncwb/vite-plugin-svg-icons)

`npx pnpm i -D vite-plugin-svg-icons`

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import viteSvgIcons from 'vite-plugin-svg-icons'

import path from 'path'

const nodeResolve = (dir) => path.resolve(__dirname, dir)

export default ({ mode }: ConfigEnv): UserConfig => {
  let plugins = [
    viteSvgIcons({
      // 指定需要缓存的图标文件夹
      iconDirs: [nodeResolve('icons')],
      // 指定symbolId格式
      symbolId: 'icon-[dir]-[name]',
      // 是否压缩
      svgoOptions: true,
    }),
  ]

  return {
    plugins,
  }
}
```

使用方式见https://github.com/anncwb/vite-plugin-svg-icons/blob/main/README.zh_CN.md

###  CDN 加载类库

[vite-plugin-cdn-import](https://github.com/MMF-FE/vite-plugin-cdn-import)

`npx pnpm i -D vite-plugin-cdn-import`

```ts
import { UserConfig, ConfigEnv, loadEnv } from 'vite'
import importToCDN from 'vite-plugin-cdn-import'

export default ({ mode }: ConfigEnv): UserConfig => {
  const IS_PROD = ['prod', 'production'].includes(mode)

  let plugins = []

  if (IS_PROD) {
    plugins = [
      ...plugins,
      importToCDN({
        modules: [
          {
            name: 'cesium',
            var: 'Cesium',
            path: `https://cesium.com/downloads/cesiumjs/releases/1.88/Build/Cesium/Cesium.js`,
          },
          {
            name: 'widgets',
            path: `https://cesium.com/downloads/cesiumjs/releases/1.88/Build/Cesium/Widgets/widgets.css`,
          },
        ],
      }),
    ]
  }

  return {
    plugins,
  }
}
```

### esbuild error

```
Error: spawn D:\github\vite-config\node_modules\esbuild\esbuild.exe ENOENT
    at Process.ChildProcess._handle.onexit (node:internal/child_process:282:19)
    at onErrorNT (node:internal/child_process:480:16)
    at processTicksAndRejections (node:internal/process/task_queues:83:21)
Emitted 'error' event on ChildProcess instance at:
    at Process.ChildProcess._handle.onexit (node:internal/child_process:288:12)
    at onErrorNT (node:internal/child_process:480:16)
    at processTicksAndRejections (node:internal/process/task_queues:83:21) {
```

```
node ./node_modules/esbuild/install.js
```