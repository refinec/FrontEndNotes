## 字体图标使用说明

`uni-app`支持使用字体图标，使用方式与普通web项目相同，但需要注意以下几点：

1. 字体文件小于`40kb`，`uni-app`会自动将其转化为`base64`格式

2. 字体文件大于`40kb`，需要开发者自己转换，否则使用不生效

3. 字体文件的引用路径推荐以`~@`开头的绝对路径

   ```css
   @font-face {
     font-family: xxx;
     src: url("~@/static/iconfontt.ttf");
   }
   ```

   