# vue-cli

Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统。有三个组件：

* **CLI**：`@vue/cli` 全局安装的 npm 包，提供了终端里的vue命令（如：vue create 、vue serve 、vue ui 等命令）

* **CLI 服务**：`@vue/cli-service`是一个开发环境依赖。构建于 webpack 和 webpack-dev-server 之上（提供 如：`serve`、`build` 和 `inspect` 命令）

* **CLI 插件**：给Vue 项目提供可选功能的 npm 包 （如： Babel/TypeScript 转译、ESLint 集成、unit和 e2e测试 等）

## 安装

1. 全局安装过旧版本的 `vue-cli`(1.x 或 2.x)要先卸载它，否则跳过此步：

   `npm uninstall vue-cli -g //或者 yarn global remove vue-cli`

2. Vue CLI 3需要 nodeJs ≥ 8.9 (官方推荐 8.11.0+，你可以使用 nvm 或 nvm-windows在同一台电脑中管理多个 Node 版本）

3. 安装@vue/cli（Vue CLI 3的包名称由 `vue-cli` 改成了 `@vue/cli`, vue -V  检查vue版本号

   ` cnpm install -g @vue/cli //yarn global add @vue/cli`

## 使用

```json
? Check the features needed for your project: (Press <space> to select, <a> to toggle all, <i> to invert selection)
>( ) Babel //转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。
( ) TypeScript// TypeScript是一个JavaScript（后缀.js）的超集（后缀.ts）包含并扩展了 JavaScript 的语法，需要被编译输出为 JavaScript在浏览器运行，目前较少人再用
( ) Progressive Web App (PWA) Support// 渐进式Web应用程序
( ) Router // vue-router（vue路由）
( ) Vuex // vuex（vue的状态管理模式）
( ) CSS Pre-processors // CSS 预处理器（如：less、sass）
( ) Linter / Formatter // 代码风格检查和格式化（如：ESlint）
( ) Unit Testing // 单元测试（unit tests）
( ) E2E Testing // e2e（end to end） 测试
```

#### 单元测试 ：

```json
? Pick a unit testing solution: (Use arrow keys)
> Mocha + Chai //mocha灵活,只提供简单的测试结构，如果需要其他功能需要添加其他库/插件完成。必须在全局环境中安装
Jest //安装配置简单，容易上手。内置Istanbul，可以查看到测试覆盖率，相较于Mocha:配置简洁、测试代码简洁、易于和babel集成、内置丰富的expect
```

