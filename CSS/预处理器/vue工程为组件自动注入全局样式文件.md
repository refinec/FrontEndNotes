# vue工程为组件自动注入全局样式文件

```sh
// Less
$ npm i -D less less-loader

// Sass/Scss
$ npm i -D node-sass sass-loader

// Stylus
$ npm i -D stylus stylus-loader
```

1. 提取出一些公共的样式，如`variables`、`mixins`、`functions`等几乎在所有业务组件中都会用到的样式：

   ```
   -- src
   ---- styles
   -------- variables.less
   -------- mixins.less
   -------- functions.less
   ```


2. 创建一个包含上面引入的入口样式文件`entry.less`，然后在各组件中导入即可

   ```less
   # entry.less
   
   @import './variables';
   @import './mixins';
   @import './functions';
   ```

   ```less
   <style lang="less">
   @import "../styles/entry";
   
   // 其他样式
   </style>
   ```

3. 在vue工程中配置自动导入

   **Less和Stylus**

   配置Less和Stylus自动导入有两种方案：

   - 使用[style-resources-loader](https://github.com/yenshih/style-resources-loader)
   - 使用[vue-cli-plugin-style-resources-loader](https://www.npmjs.com/package/vue-cli-plugin-style-resources-loader)

   推荐使用第一种，因为第二种方案只是对第一种方案的包装，且暂不支持热更新。

   安装：`npm i -D style-resources-loader`

   ```javascript
   #vue.config.js
   // vue.config.js
   const path = require('path')
   
   module.exports = {
     chainWebpack: config => {
       const types = ['vue-modules', 'vue', 'normal-modules', 'normal']
       types.forEach(type => addStyleResource(config.module.rule('less').oneOf(type)))  // A
     },
   }
   
   function addStyleResource (rule) {
     rule.use('style-resource')
       .loader('style-resources-loader')
       .options({
         patterns: [
           path.resolve(__dirname, './src/styles/entry1.less'), // B 
           path.resolve(__dirname, './src/styles/entry2.less'), // 配置多个导入
         ],
       })
   }
   ```

   如果工程使用的是`Stylus`，则将A行替换为

   ```js
   types.forEach(type => addStyleResource(config.module.rule('stylus').oneOf(type)))
   ```

   将B行替换为

   ```js
   path.resolve(__dirname, './src/styles/entry.styl')
   ```

   **Sass/Scss**

   > Sass/Scss配置自动导入也可以使用上面的方案，但是使用其原生的方案更加便捷

   ```javascript
   # vue.config.js
   module.exports = {
     css: {
       loaderOptions: {
         sass: {
           prependData: `  // sass-loader@8.0.0之前，要将prependData替换为data
               @import "@/styles/entry1.scss"; // 配置多个导入
               @import "@/styles/entry2.scss";
           `
         }
       }
     }
   }
   ```

   