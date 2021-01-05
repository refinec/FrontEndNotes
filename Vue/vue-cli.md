# vue-cli

## Babel

Babel 可以通过 `babel.config.js` 进行配置。

> Vue CLI 使用了 Babel 7 中的新配置格式 `babel.config.js`。和 `.babelrc` 或 `package.json` 中的 `babel` 字段不同，这个配置文件不会使用基于文件位置的方案，而是会一致地运用到项目根目录以下的所有文件，包括 `node_modules` 内部的依赖。我们推荐在 Vue CLI 项目中始终使用 `babel.config.js` 取代其它格式。

所有的 Vue CLI 应用都使用 `@vue/babel-preset-app`，它包含了 `babel-preset-env`、JSX 支持以及为最小化包体积优化过的配置。通过[它的文档](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app)可以查阅到更多细节和 preset 选项。查阅 [Polyfill](https://cli.vuejs.org/zh/guide/browser-compatibility.html#polyfill) 

## ESLint

ESLint 可以通过 `.eslintrc` 或 `package.json` 中的 `eslintConfig` 字段来配置。

更多细节查阅 [@vue/cli-plugin-eslint](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint)

## TypeScript

TypeScript 可以通过 `tsconfig.json` 来配置。

更多细节可查阅 [@vue/cli-plugin-typescript](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-typescript)

## 单元测试

### Jest

更多细节可查阅 [@vue/cli-plugin-unit-jest](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-jest)。

### Mocha (配合 `mocha-webpack`)

更多细节可查阅 [@vue/cli-plugin-unit-mocha](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-mocha)。

## E2E 测试

###  Cypress

更多细节可查阅 [@vue/cli-plugin-e2e-cypress](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-cypress)。

###  Nightwatch

更多细节可查阅 [@vue/cli-plugin-e2e-nightwatch](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-nightwatch)。