## 变量(@、@{})

变量以**@**开头

```less
.arrays {
    @ary: 1,
        2,
        3;
    @ary2: 1 2 3;
    ary: `@{ary}.join(', ')`;
    ary: `@{ary2}.join(', ')`;
}
```

## 混入

* **带参数（及参数缺省值）的混入**

  ```less
  .border-radius(@radius: 3 px) {
      border-radius: @radius; 
      -moz-border-radius: @radius; 
      -webkit-border-radius: @radius;
  }
  .button {
      .border-radius(6 px);
  }
  .button2 {
      .border-radius();
  }
  
  // 混入生成的 CSS 代码
  .button {
      border-radius: 6px;
      -moz-border-radius: 6px;
      -webkit-border-radius: 6px;
  }
  .button2 {
      border-radius: 3px;
      -moz-border-radius: 3px;
      -webkit-border-radius: 3px;
  }
  ```

* **利用不同的参数个数来匹配不同的混入**

  ```less
  .mixin (@a) {
      color: @a;
      width: 10px;
  }
  .mixin (@a, @b) {
      color: fade(@a, @b);
  }
  .header {
      .mixin(red);
  }
  .footer {
      .mixin(blue, 50%);
  }
  
  // 不同参数个数调用后生成的 CSS 代码：
  .header {
      color: #ff0000;
      width: 10px;
  }
  .footer {
      color: rgba(0, 0, 255, 0.5);
  }
  ```

* **用常量参数来控制混入的模式匹配**

  定义常量参数就不同了，这时候不仅参数的个数要对应的上，而且常量参数的值和调用时的值也要一样才会匹配的上

  ```less
  .mixin (dark, @color) {
      color: darken(@color, 10%);
  }
  .mixin (light, @color) {
      color: lighten(@color, 10%);
  }
  .mixin (@zzz, @color) {
      display: block;
      weight: @zzz;**
  }
  .header {
      .mixin(dark, red);
  }
  .footer {
      .mixin(light, blue);
  }
  .body {
      .mixin(none, blue);
  }
  
  //常量参数生成的 CSS 代码：
  .header {
      color: #cc0000;
      display: block;
      weight: dark;
  }
  .footer {
      color: #3333ff;
      display: block;
      weight: light;
  }
  .body {
      display: block;
      weight: none;
  }
  ```

* **无参和常量参数的模式匹配**

  ```less
  .border-radius (@radius: 3px) {
      border-radius: @radius;
      -moz-border-radius: @radius;
      -webkit-border-radius: @radius;
  }
  .border-radius (7px) {
      border-radius: 7px;
      -moz-border-radius: 7px;
  }
  .border-radius () {
      border-radius: 4px;
      -moz-border-radius: 4px;
      -webkit-border-radius: 4px;
  }
  .button {
      .border-radius(6px);
  }
  .button2 {
      .border-radius(7px);
  }
  .button3 {
      .border-radius();
  }
  ```

  无参的混入是能够匹配任何调用，而常量参数非常严格，必须保证参数的值 （7px）和调用的值 （7px）一致才会匹配。

  加入了无参混入后生成的 CSS 代码：

  ```less
  .button {
      border-radius: 6px;
      -moz-border-radius: 6px;
      -webkit-border-radius: 6px;
      border-radius: 4px;
      -moz-border-radius: 4px;
      -webkit-border-radius: 4px;
  }
  .button2 {
      border-radius: 7px;
      -moz-border-radius: 7px;
      -webkit-border-radius: 7px;
      border-radius: 7px;
      -moz-border-radius: 7px;
      border-radius: 4px;
      -moz-border-radius: 4px;
      -webkit-border-radius: 4px;
  }
  .button3 {
      border-radius: 3px;
      -moz-border-radius: 3px;
      -webkit-border-radius: 3px;
      border-radius: 4px;
      -moz-border-radius: 4px;
      -webkit-border-radius: 4px;
  }
  ```

## 条件表达式(when、and、，(or)、not)

> 实现的方式就是利用了 when这个关键词

* **利用条件表达式来控制模式匹配**

  ```less
  .w(@value){
      // 检查传入的变量值是否大于0
      & when (@value > 0) {
          // 如果大于0，则生成width样式
          width: unit(@value, px);
      }
  }
  ```

  ```less
  .mixin (@a) when (@a >=10) {
      background-color: black;
  }
  .mixin (@a) when (@a < 10) {
      background-color: white;
  }
  .class1 {
      .mixin(12)
  }
  .class2 {
      .mixin(6)
  }
  
  // 条件表达式生成的 CSS 代码
  .class1 {
      background-color: black;
  }
  .class2 {
      background-color: white;
  }
  ```

* **类型检查函数来辅助条件表达式**

  例如 iscolor、isnumber、isstring、iskeyword、isurl等等

* **条件表达式中支持的类型检查函数**

  ```less
  .mixin (@a) when (iscolor(@a)) {
      background-color: black;
  }
  .mixin (@a) when (isnumber(@a)) {
      background-color: white;
  }
  .class1 {
      .mixin(red)
  }
  .class2 {
      .mixin(6)
  }
  // 类型检查匹配后生成的 CSS 代码
  .class1 {
      background-color: black;
  }
  .class2 {
      background-color: white;
  }
  ```

* **条件表达式同样支持 AND 和 OR 以及 NOT 来组合条件表达式，但是OR 在 LESS 中并不是用 or 关键字，而是用 ， 来表示 or 的逻辑关系**

  ```less
  .smaller (@a, @b) when (@a > @b) {
      background-color: black;
  }
  .math (@a) when (@a > 10) and (@a < 20) {
      background-color: red;
  }
  .math (@a) when (@a < 10)，(@a > 20) {
      background-color: blue;
  }
  .math (@a) when not (@a =10) {
      background-color: yellow;
  }
  .math (@a) when (@a =10) {
      background-color: green;
  }
  .testSmall {
      .smaller(30, 10)
  }
  .testMath1 {
      .math(15)
  }
  .testMath2 {
      .math(7)
  }
  .testMath3 {
      .math(10)
  }
  // AND，OR，NOT 条件表达式生成的 CSS 代码
  .testSmall {
      background-color: black;
  }
  .testMath1 {
      background-color: red;
      background-color: yellow;
  }
  .testMath2 {
      background-color: blue;
      background-color: yellow;
  }
  .testMath3 {
      background-color: green;
  }
  ```

## JavaScript 赋值 

```less
.eval {
    js: `1 + 1`;
    js: `(1 + 1==2 ? true : false)`;
    js: `"hello".toUpperCase() + '!'`;
    title: `process.title`;    //process.title是JS的运行环境
}
.scope {
    @foo: 42;
    var: `this.foo.toJS()`;
}
.escape-interpol {
    @world: "world";
    width:~`"hello"+" "+@{world}`;
}
.arrays {
    @ary: 1,
        2,
        3;
    @ary2: 1 2 3;
    ary: `@{ary}.join(', ')`;
    ary: `@{ary2}.join(', ')`;
}
// JavaScript 赋值生成的 CSS 代码
.eval {
    js: 2;
    js: true;
    js: "HELLO!";
    title: "/Users/Admin/Downloads/LESS/Less.app/Contents/Resources/engines/bin/node";
}
.scope {
    var: 42;
}
.escape-interpol {
    width: hello world;
}
.arrays {
    ary: "1, 2, 3";
    ary: "1, 2, 3";
}
```

## 颜色

```less
lighten(@color, 10%); /* 返回的颜色在@color基础上变亮10% */
darken(@color, 10%);  /* 返回的颜色在@color基础上变暗10%*/
saturate(@color, 10%);   /* 返回的颜色在@color基础上饱和度增加10% */
desaturate(@color, 10%); /* 返回的颜色在@color基础上饱和度降低10%*/
spin(@color, 10);  /* 返回的颜色在@color基础上色调增加10 */
spin(@color, -10); /* 返回的颜色在@color基础上色调减少10 */
mix(@color1, @color2); /* 返回的颜色是@color1和@color2两者的混合色 */
```

## 函数

```less
.remMixin() {
@functions: ~`(function() {
  var clientWidth = '375px';
  function convert(size) {
    return typeof size === 'string' ? 
      +size.replace('px', '') : size;
  }
  this.rem = function(size) {
    return convert(size) / convert(clientWidth) * 10 + 'rem';
  }
})()`;
}

.remMixin();
```

```less
.vwMixin() {
    @functions: ~`(function() {
        var clientWidth = '1920px';
        function convert(size) {
            return typeof size === 'string' ? 
                +size.replace('px', '') : size;
        }
        this.vw = function(size) {
            return convert(size) / convert(clientWidth) * 100 + 'vw';
        }
    })()`;
}
.vhMixin() {
    @functions: ~`(function() {
        var clientWidth = '1080px';
        function convert(size) {
            return typeof size === 'string' ? 
                +size.replace('px', '') : size;
        }
        this.vh = function(size) {
            return convert(size) / convert(clientWidth) * 100 + 'vh';
        }
    })()`;
}
.vwMixin();
.vhMixin();
// 使用方式示例
// .el-function {
//     width: ~`vw("300px")`;
//     height: ~`vh(150)`;
// }
```



