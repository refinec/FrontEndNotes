# Sass

## 安装使用

**安装sass 依赖包 :**

`npm install sass-loader node-sass -D`

使用scss时候在所在的**style**样式标签上添加**lang=”scss”**即可应用对应的语法，否则报错

1. **有了两套语法规则**：

   一套依旧是用缩进作为分隔符来区分代码块的；

   一套规则和CSS一 样采用了大括号｛｝作为分隔符。

2. **有两种后缀名文件**：

   一种后缀名为sass，不使用大括号和分号；

   一种后缀名为scss，使用大括号和分号。

3. **sass的导入**：

   编译时会将@import的scss文件合并进来只生成一个CSS文件。但是在sass文件中导入css文件如@import 'reset.css'，那效果跟普通CSS导入样式文件一样，导入的css文件不会 合并到编译后的文件中，而是以@import方式存在。 

   所有的sass导入文件都可以忽略后缀名.scss。一般以_开头，如_mixin.scss。这种文件在导入的时候可以不写下划线，可写成@import "mixin"。

4. **注释:**

   SASS共有两种注释风格。

   - 标准的CSS注释 `/* comment */`，会保留到编译后的文件。

   - 单行注释 `// comment`，只保留在SASS源文件中，编译后被省略。

   - 在/*后面加一个感叹号，表示这是"重要注释"。即使是压缩模式编译，也会保留这行注释，通常可以**用于声明版权信息**。

     **/*! 
      重要注释！
      */**

5. **SASS提供四个编译风格的选项，默认为nested**

   生产环境当中，一般使用最后一个选项：**sass --style compressed test.sass test.css**

   - **nested**：嵌套缩进的css代码
   - **expanded**：展开的多行css代码
   - **compact**：简洁格式的css代码
   - **compressed**：压缩后的css代码

可以让SASS监听某个文件或目录，一旦源文件有变动，就自动生成编译后的版本。

```shell
// watch a file
sass --watch input.scss:output.css

// watch a directory
sass --watch app/sass:public/stylesheets
```

## 变量($、#{})

1. **Sass变量符为$开头**

   ```scss
   $blue : #1875e7;
   div {
       color : $blue;
   }
   ```

   **如果变量需要镶嵌在字符串之中，就必须需要写在`#{}`之中 **。

   ```scss
   $side : left;
   .rounded {
       border-#{$side}-radius: 5px;
   }
   ```

## 计算功能

```scss
body {
    margin: (14px/2);
    top: 50px + 100px;
    right: $var * 10%;
}
```

## 选择器嵌套(&)

```scss
div h1 {
    color: red;
}
//可以写成：
div {
    h1 {
        color: red;
    }
}
```

* **属性也可以嵌套，比如**`border-color`**属性，可以写成：**

  *注意，**border**后面必须加上冒号*

  ```scss
  p {
      border: {
          color: red;
      }
  }
  ```

* **在嵌套的代码块内，可以使用`&`引用父元素。比如`a:hover`伪类，可以写成：**

  ```scss
  a {
      &: hover {
          color: #ffb3ff;
      }
  }
  ```

## 代码的重用

### 继承(@extend)

> SASS允许一个选择器，继承另一个选择器。比如，现有class1：

```scss
.class1 {
    border: 1px solid #ddd;
 }
```

**class2要继承class1，就要使用`@extend`命令：**

```scss
.class2 {
    @extend .class1;
    font-size:120%;
}
```

### Mixin(代码重用@mixin、@include)

Mixin有点像C语言的宏（macro），**是可以重用的代码块**。

1. 使用**@mixin**命令，定义一个代码块。

   ```scss
   @mixin left {
        float: left;
        margin-left: 10px;
    }
   ```

2. 使用**@include**命令，调用这个**mixin**

   ```scss
   div {
       @include left;
   }
   ```

* mixin的强大之处，在于可以**指定参数和缺省值 **。

  ```scss
  @mixin left($value: 10px) {
      float: left;
      margin-right: $value;
  }
  
  // 使用的时候，根据需加入参数：
  div {
      @include left(20px);
  }
  ```

* **下面是一个mixin的实例，用来生成浏览器前缀**

  ```scss
  @mixin rounded($vert, $horz, $radius: 10px) {
           border-#{$vert}-#{$horz}-radius: $radius;
           -moz-border-radius-#{$vert}#{$horz}: $radius;
           -webkit-border-#{$vert}-#{$horz}-radius: $radius;
  }
  // 使用的时候，可以像下面这样调用：
  #navbar li { @include rounded(top, left); }
  #footer { @include rounded(top, left, 5px); }
  
  ```

## 条件语句(@if、@else)

```scss
p {
    @if 1 + 1 == 2 { border: 1px solid; }
    @if 5 < 3 { border: 2px dotted; }
}
// 或
@if lightness($color) > 30% {
    background-color: #000;
} @else {
    background-color: #fff;
}

```

## 循环语句(for、while、each)

* **for循环**

  ```scss
  @for $i from 1 to 10 {
      .border-#{$i} {
          border: #{$i}px solid blue;
      }
  }
  @for $i from 1 through 10 {
      .border-#{$i} {
          border: #{$i}px solid blue;
      }
  }
  ```

  这两个的区别是关键字 through 表示包括 最后一个数，而 to 则不包括 最后一个数

* **while循环**

  ```scss
  $i: 6;
  @while $i > 0 {
      .item-#{$i} { width: 2em * $i; }
      $i: $i - 2;
  }
  ```

* **each命令，作用与for类似**

  ```scss
  @each $member in a, b, c, d {
      .#{$member} {
          background-image: url("/image/#{$member}.jpg");
      }
  }
  ```

## 自定义函数

>  SASS允许用户编写自己的函数

```scss
@function double($n) {
    @return $n * 2;
}
#sidebar {
    width: double(5px);
}
```

## 颜色函数

>  SASS提供了一些内置的颜色函数，以便生成系列颜色

```scss
lighten(#cc3, 10%) // #d6d65c
darken(#cc3, 10%) // #a3a329
grayscale(#cc3) // #808080
complement(#cc3) // #33c
lighten($color, 10%); /* 返回的颜色在$color基础上变亮10% */
darken($color, 10%);  /* 返回的颜色在$color基础上变暗10% */
saturate($color, 10%);   /* 返回的颜色在$color基础上饱和度增加10% */
desaturate($color, 10%); /* 返回的颜色在$color基础上饱和度减少10% */
grayscale($color);  /* 返回$color的灰度色*/
complement($color); /* 返回$color的补色 */
invert($color);     /* 返回$color的反相色 */
mix($color1, $color2, 50%); /* $color1 和 $color2 的 50% 混合色*/

$color: #0982C1;
h1 {
    background: $color;
    border: 3px solid darken($color, 50%);/*边框颜色在$color的基础上变暗50%*/
}
```

