- 每一个 `.wxs` 文件和 `<wxs>` 标签都是一个单独的模块。

- 一个模块要想对外暴露其内部的**私有变量**与**函数**，只能通过 `module.exports` 实现。再使用 `require` 函数在`.wxs`模块中引用其他 `wxs` 文件模块。

  引用的时候，要注意如下几点：

  - 只能引用 `.wxs` 文件模块，且必须使用**相对路径**。

  - `wxs` 模块均为单例，`wxs` 模块在第一次被引用时，会自动初始化为单例对象。多个页面，多个地方，多次引用，使用的都是**同一个 `wxs` 模块对象**。

  - 如果一个 `wxs` 模块在定义之后，一直没有被引用，则该模块不会被解析与运行。

  ```js
  // /pages/tools.wxs
  
  var foo = "'hello world' from tools.wxs";
  var bar = function (d) {
    return d;
  }
  module.exports = {
    FOO: foo,
    bar: bar,
  };
  module.exports.msg = "some msg";
  ```

  ```js
  // /pages/logic.wxs
  
  var tools = require("./tools.wxs");
  console.log(tools.FOO);
  console.log(tools.bar("logic.wxs"));
  console.log(tools.msg);
  ```

  ```html
  <!-- page/index/index.wxml -->
  
  <wxs src="./../tools.wxs" module="tools" />
  <view> {{tools.msg}} </view>
  <view> {{tools.bar(tools.FOO)}} </view>
  ```

> 注意📢：
>
> 1. `<wxs>` 模块只能在定义模块的 WXML 文件中被访问到。使用 `<include>` 或 `<import>` 时，`<wxs>` 模块不会被引入到对应的 WXML 文件中。
> 2. 在`<template>` 内只能使用 定义该 `<template>` 的 WXML 文件中 定义的 `<wxs>` 模块。



