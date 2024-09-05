# CSS

## 两种盒模型

盒模型分为标准盒模型和怪异盒模型(IE模型)

```javascript
box-sizing：content-box   //标准盒模型
box-sizing：border-box    //怪异盒模型
```

### **content-box**

默认值，标准盒子模型。 [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 与 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 只包括内容的宽和高， **不包括边框（border），内边距（padding），外边距（margin）**。注意: 内边距、边框和外边距都在这个盒子的外部。 比如说，`.box {width: 350px; border: 10px solid black;}` 在浏览器中的渲染的实际宽度将是 370px。

### **border-box**

[`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 和 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 属性包括**内容**，**内边距**和**边框**，但**不包括外边距**。这是当文档处于 Quirks模式 时Internet Explorer使用的[盒模型](https://developer.mozilla.org/en-US/docs/CSS/Box_model)。注意，填充和边框将在盒子内 , 例如, `.box {width: 350px; border: 10px solid black;}` 导致在浏览器中呈现的宽度为350px的盒子。内容框不能为负，并且被分配到0，使得不可能使用border-box使元素消失。

## 如何居中？

### 水平居中

**内联元素**，**宽度默认就是内容的宽度**，只需要给父级添加text-align

```css
.wrapper { text-align: center; }
```

**块级元素**，将它的`margin-left`和`margin-right`设置为`auto`，并且块级元素一定要设置宽度，否则元素默认为100%宽度，不需要居中。

```css
.inner {
    display: block;
    width: 150px; /* 一定要设置宽度，不然就不需要局中了 */
    margin: 0 auto;
}
```

两个以上的水平局中，可以将其设置为`display:inline-block`，在设置父级`text-align`

### 垂直居中

**内联元素**，第一种实用的是flex布局，这里居中的值得是相对于父盒子

```css
.wrapper{
    display: flex;
    align-items: center;
}
```

第二种，这里面指的居中是相对于自身而言的

```css
.inner{
    height:100px;
    line-height:100px;
}
```

**块级元素**

宽高确定情况下，实用 **`position absolute + 负margin`**

宽高不确定的情况下，实用**`position absolute + transform`**

**垂直水平局中**

子元素宽高确定的情况下，使用**`position absolute + 负margin`**

子元素宽高不确定的，使用**`position absolute + transform`**

```css
.inner {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    background: blue;
}
```

## 两列布局

### 左列定宽,右列自适应

**float+margin**

```css
.left{
    float: left;
    width: 100px;
    height: 100%;
}
.right{
    margin-left: 100px; /* 大于等于#left的宽度 */
    height: 100%;
}
```

### 左列自适应,右列定宽

**float+overflow**

```html
<div>
    <div class="right"></div>
    <div class="left"></div>
</div>
```

```css
<style>
    .left {
        overflow: hidden; /* 触发bfc */
        height: 100px;
        background: red;
    }

    .right {
        float: right;
        margin-left: 10px; /* margin需要定义在right中 */
        width: 100px;
        height: 100px;
        background: yellow;
    }
</style>
```

## 三列布局

### 两列定宽,一列自适应

使用float+margin实现

```html
<div class="wrapper">
    <div class="left"></div>
    <div class="main"></div>
    <div class="rigth"></div>
</div>
```

```css
.wrapper {
    height: 400px;
    min-width: 500px;
}

.left {
    float: left;
    margin-left: 10px;
    width: 100px;
    height: 100%;
    background: red;
}

.main {
    float: left;
    margin-left: 20px;
    width: 100px;
    height: 100%;
    background: yellow;
}

.rigth {
    /*等于 left 和 main 的宽度之和加上间隔,多出来的就是 right 和 main的间隔*/
    margin-left: 230px;
    height: 100%;
    background: blue;
}
```

### (双飞翼布局) 间列自适应宽度，旁边两侧固定宽度

实现步骤:

1. 三个部分都设定为左浮动，然后设置`main`的宽度为100%，此时，`left`和`right`部分会跳到下一行；
2. 通过设置margin-left为负值让left和right部分回到与center部分同一行；
3. center部分增加一个内层div，并设margin: 0 200px；

```html
<div class="wrapper">
    <div class="main">
        <div class="inner"></div>
    </div>
    <div class="left"></div>
    <div class="right"></div>
</div>
```

```css
.wrapper {
    /* 确保中间内容可以显示出来，两倍left宽+right宽 */
    min-width: 600px; 
}

.left {
    float: left;
    width: 200px;
    margin-left: -100%;
    height: 500px;
}
.right {
    float: left;
    width: 200px;
    margin-left: -200px;
    height: 500px;
}
.main {
    float: left;
    width: 100%; 
    height: 500px;
}
.main .inner {
    /* margin水平方向要是左右两者的宽度 */
    margin: 0 200px;    
    height: 100%;
    border: 2px solid brown;
}
```

## 选择器优先级

**css常用选择器**

通配符：*
ID选择器：#ID
类选择器：.class
元素选择器：p、a    等
后代选择器：p span、div a   等
伪类选择器：a:hover 等
属性选择器：input[type="text"]  等

**css选择器权重**

!important -> 行内样式 -> #id -> .class -> 元素和伪元素 -> * -> 继承 -> 默认

**属性继承  **

（1）所有元素可继承：visibility、cursor 

（2）块级元素可继承：text-indent、text-align 

（3）内联元素可继承：

* 字体系列属性：font、font-family、font-size、font-style、font-variant、font-weight、font-stretch、font-size-adjust 

* 除text-indent、text-align之外的文本系列属性：

  letter-spacing、word-spacing、white-space、line-height、color、text-transform、direction 

**不可继承的样式属性 **： 

1、display 

2、文本属性：vertical-align、text-decoration、text-shadow、unicode-bidi 

3、盒子模型属性：border、padding、margin、width、height 

4、背景属性：background 

5、定位属性：float、clear、position

 6、生成内容属性：content 

7、轮廓样式属性：outlien-style 

8、页面样式属性：size 

9、声音样式属性：pause-before



## CSS新特性

transition：过渡
transform：旋转、缩放、移动或者倾斜
animation：动画
gradient：渐变
shadow：阴影
border-radius：圆角

**transition**

```css
transition: property duration timing-function delay;
// css属性名称   过渡时间  过渡时间曲线  过渡延迟时间
```

**transform**

```css
transform:rotate(30deg)  旋转
transform:translate(100px,20px)  移动
transform:scale(2,1.5);  缩放
transform:skew(30deg,10deg);  扭曲
```

**animation**

```css
animation: move 1s linear forwards;
// 动画的名称
// 定义动画的时间  duration 
// 动画的贝塞尔曲线
// animation-fill-mode 属性规定动画在播放之前或之后，其动画效果是否可见。 
// forwards  当动画完成后，保持最后一个属性值	
```

## 清除浮动

1. 父级元素添加伪元素

   ```css
   .clearfix:after{
     content: "";
     display: block;
     clear: both; 
   }
   
    .clearfix{
        zoom: 1; /* IE 兼容*/
    }
   ```

2. 给父级元素添加 `overflow:hidden` 或者 `auto` 样式

3. 给所有浮动标签后面添加一个空的 div,给 div 添加 `clear:both` 属性

    ```html
    <div class="clear"> </div>
    <style>
        .clear {
            clear:both;
            margin:0;
            padding:0;
        }
    </style>
    ```

## 三种定位方案

在定位的时候，浏览器就会根据元素的盒类型和上下文对这些元素进行定位，可以说盒就是定位的基本单位。定位时，有三种定位方案，分别是**常规流，浮动、绝对定位 **。

**常规流(Normal flow)**

* 在常规流中，盒一个接着一个排列;

* 在**块级格式化上下文**里面， 它们**竖着**排列；

* 在**行内格式化上下文**里面， 它们**横着**排列;

* 当`position`为`static`或`relative`，并且`float`为`none`时会触发常规流；

* 对于**静态定位**(static positioning)，`position: static`，**盒的位置是常规流布局里的位置**；

* 对于**相对定位**(relative positioning)，`position: relative`，盒偏移位置由这些属性定义`top`，`bottom`，`left`and`right`。**即使有偏移，仍然保留原有的位置**，其它常规流不能占用这个位置。

 **浮动(Floats)**

- 盒称为浮动盒(floating boxes)；
- 它位于当前行的开头或末尾；
- 这**导致常规流环绕在它的周边**，除非设置 clear 属性；

**绝对定位(Absolute positioning)**

- 绝对定位方案，**盒从常规流中被移除**，不影响常规流的布局；
- 它的定位相对于它的包含块，相关CSS属性：`top`，`bottom`，`left`及`right`；
- 如果元素的属性`position`为`absolute`或`fixed`，它是绝对定位元素；
- 对于`position: absolute`，元素定位将相对于最近的一个`relative`、`fixed`或`absolute`的父元素，如果没有则相对于`body`；

## css hack

>  针对不同的浏览器或浏览器不同版本写相应的CSS的过程，就是CSS hack。

CSS hack书写顺序，一般是将适用范围广、被识别能力强的CSS定义在前面。

**CSS hack分类3种：**条件hack、属性级hack、选择符hack。

```css
/* 1、条件hack */
<!--[if IE]>
<style>
	.test{color:red;}
</style>
<![endif]-->

/* 2、属性hack（类内部hack）*/
.test{
	color:#090\9; /*For IE8*/
	*color:#f00; /*For IE7 and earlier*/
	_color:#ff0; /*For IE6 and earlier*/
}

/* 3、选择符hack（选择器属性前缀法）*/
* htm .test{color:#0f90;} /*For IE6 and earlier*/
* + html .test{color:#ff0;} /*For IE7*/
```

## link和@import的区别？

1. 从属关系区别：
   link属于html标签，而@import是css提供的。
2. 加载顺序区别：
   页面被加载时，link会同时被加载，而**<u>@import引用的css会等到页面被加载完再加载</u> **。
3. 兼容性区别：
   import只在IE5以上才能识别，而link是html标签，无兼容问题。
4. dom可操作性区别：
   可以通过JS 操作 DOM ，插入link标签来改变样式；由于 DOM 方法是基于文档的，无法使用@import的方式插入样式
5. 权重区别：
   如果已经存在相同样式，@import引入的这个样式将被该 CSS 文件本身的样式层叠掉，表现出link方式的样式权重高于@import的权重这样的直观效果。
   （aSuncat：简而言之，link和@import，谁写在后面，谁的样式就被应用，后面的样式覆盖前面的样式。）

## 伪类和伪元素

伪类：`:active` `:focus` `:hover` `:link` `:visited` `:first-child `

伪元素：`:before` `:after` `:first-letter` `:first-line`

## svg和canvas的区别

1. canvas画图**基于像素点**，是位图，如果进行放大或缩小**会失真** ；

   svg**基于图形**，用html标签描绘形状，放大缩小**不会失真** 

2. canvas需要**在js中绘制** ；svg**在html绘制** 

3. canvas支持颜色比svg多 

4. canvas无法对已经绘制的图像进行修改、操作 ；svg可以获取到标签进行操作