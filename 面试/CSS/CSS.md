# CSS

## 两种盒模型

盒模型分为标准盒模型和怪异盒模型(IE模型)

```javascript
box-sizing：content-box   //标准盒模型
box-sizing：border-box    //怪异盒模型
```

### **content-box**

默认值，标准盒子模型。 [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 与 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 只包括内容的宽和高， 不包括边框（border），内边距（padding），外边距（margin）。注意: 内边距、边框和外边距都在这个盒子的外部。 比如说，`.box {width: 350px; border: 10px solid black;}` 在浏览器中的渲染的实际宽度将是 370px。

### **border-box**

[`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 和 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 属性包括内容，内边距和边框，但不包括外边距。这是当文档处于 Quirks模式 时Internet Explorer使用的[盒模型](https://developer.mozilla.org/en-US/docs/CSS/Box_model)。注意，填充和边框将在盒子内 , 例如, `.box {width: 350px; border: 10px solid black;}` 导致在浏览器中呈现的宽度为350px的盒子。内容框不能为负，并且被分配到0，使得不可能使用border-box使元素消失。

## 如何居中？

### 水平居中

**内联元素**，**宽度默认就是内容的宽度**，只需要给父级添加text-align

```css
.wrapper{text-align: center;}
```

**块级元素**，将它的margin-left和margin-right设置为auto，并且块级元素一定要设置宽度，否则元素默认为100%宽度，不需要居中。

```css
.inner{
    display: block;
    width: 150px;
    margin: 0 auto;
}
// 一定要设置宽度，不然就不需要局中了
```

两个以上的水平局中，可以将其设置为`display:inline-block`，在设置父级text-align

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

宽高确定情况下，实用 **position absolute + 负margin**

宽高不确定的情况下，实用**position absolute + transform**

**垂直水平局中**

子元素宽高确定的情况下，使用**position absolute + 负margin**

子元素宽高不确定的，使用**position absolute + transform**

```css
.inner{
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
.rigth{
    height: 100%;
    margin-left: 100px; /*大于等于#left的宽度*/
}
```

### 左列自适应,右列定宽

**float+overflow**

```html
<div class="wrapper">
        <div class="rigth"></div>
        <div class="left"></div>
</div>
```

```css
.left {
    overflow: hidden;
    /*触发bfc*/
    height: 100%;
}
.rigth {
    margin-left: 10px;
    /*margin需要定义在#right中*/
    float: right;
    width: 100px;
    height: 100%;
}
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
    margin-left: 10px;
    float: left; /*浮动*/
    width: 100px;
    height: 100%;
}
.main{
    float: left; /*浮动*/
    width: 100px;
    height: 100%;
    margin-left: 20px;
}
.rigth {
    margin-left: 230px;  /*等于#left和#center的宽度之和加上间隔,多出来的就是#right和#center的间隔*/
    height: 100%;
    background-color: blue;
}

```

### 间列自适应宽度，旁边两侧固定宽度

双飞翼布局

实现步骤

- 三个部分都设定为左浮动，然后设置center的宽度为100%，此时，left和right部分会跳到下一行；
- 通过设置margin-left为负值让left和right部分回到与center部分同一行；
- center部分增加一个内层div，并设margin: 0 200px；

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
    /* //确保中间内容可以显示出来，两倍left宽+right宽 */
    min-width: 600px; 
}

.left {
    float: left; /*浮动*/
    width: 200px;
    margin-left: -100%;
    
    height: 400px;
    
}
.right {
    float: left; /*浮动*/
    width: 200px;
    margin-left: -200px;
    
    height: 400px;
}

.main {
    float: left; /*浮动*/
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

## flex 怎么用，常用属性有哪些？

flex 的核心的概念就是 **容器** 和 **轴**。

### 父容器

**justify-content 项目在主轴上的对齐方式**

- flex-start | flex-end | center | space-between | space-around
- space-between 子容器沿主轴均匀分布，位于首尾两端的子容器与父容器相切。
- space-around  子容器沿主轴均匀分布，位于首尾两端的子容器到父容器的距离是子容器间距的一半。

**align-items** **定义项目在侧轴上如何对齐**

- flex-start | flex-end | center | baseline | stretch;
- baseline: 项目的第一行文字的基线对齐。
- stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

### 子容器

**align-self 单个项目对齐方式**

- align-self: auto | flex-start | flex-end | center | baseline | stretch;

**flex：前面三个属性的简写 是flex-grow  flex-shrink flex-basis的简写**

- flex-grow 放大比例 根据所设置的比例分配盒子所剩余的空间
- flex-shrink 缩小比例 设置元素的收缩比例   多出盒子的部分，按照比例的大小砍掉相应的大小，即比例越大，被砍的越大，默认值是1
- flex-basis  伸缩基准值 项目占据主轴的空间。 该属性设置元素的宽度或高度，当然width也可以用来设置元素宽度，如果元素上同时出现了width 和flex-basis那么flex-basis会覆盖width的值

  flex: 0 1 auto； 默认主轴是row,那么不会去放大比例，如果所有的子元素宽度和大于父元素宽度时，就会按照比例的大小去砍掉相应的大小。

### 轴

**flex-direction 决定主轴的方向 即项目的排列方向**

row | row-reverse | column | column-reverse

## BFC 是什么？

BFC全称是Block Formatting Context，即**块格式化上下文 **.

BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

**BFC布局规则**

1. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

2. 内部的Box会在垂直方向，一个接一个地放置。

3. Box垂直方向的距离由margin决定。**属于同一个BFC的两个相邻Box的margin会发生重叠 **

   ```html
   <!--属于同一个BFC的两个相邻的Box会发生margin重叠，所以我们可以设置，两个不同的BFC，也就是我们可以让把第二个p用div包起来，然后激活它使其成为一个BFC -->
   <style>
       *{
           margin: 0;
           padding: 0;
       }
       p {
           color: #f55;
           background: yellow;
           width: 200px;
           line-height: 100px;
           text-align:center;
           margin: 30px;
       }
       div{
           overflow: hidden;
       }
   </style>
   <body>
       <p>1看看我的 margin是多少</p>
       
    	<!--<p>2看看我的 margin是多少</p>  -->
       <div>
           <p>2看看我的 margin是多少</p>
       </div>
   </body>
   ```

4. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

   ```html
   <style>
       *{
           margin: 0;
           padding: 0;
       }
       body {
           width: 100%;
           position: relative;
       }
    
       .left {
           width: 100px;
           height: 150px;
           float: left; /*浮动*/
           background: rgb(139, 214, 78);
           text-align: center;
           line-height: 150px;
           font-size: 20px;
       }
       .right {
           overflow: hidden; /* 添加这行使BFC的区域不会与float box重叠 */
           height: 300px;
           background: rgb(170, 54, 236);
           text-align: center;
           line-height: 300px;
           font-size: 40px;
       }
   </style>
   <body>
       <div class="left">LEFT</div>
       <div class="right">RIGHT</div>
   </body>
   ```

5. BFC的区域不会与float box重叠。

6. 计算BFC的高度时，浮动元素也参与计算

   ```html
   <!--当我们不给父节点设置高度，子节点设置浮动的时候，会发生高度塌陷，这个时候我们就要清除浮动。-->
   <style>
       .par {
           border: 5px solid rgb(91, 243, 30);
           width: 300px;
           overflow: hidden; /*清除浮动，高度值浮动元素也参与计算*/
       }
       
       .child {
           border: 5px solid rgb(233, 250, 84);
           width:100px;
           height: 100px;
           float: left;
       }
   </style>
   <body>
       <div class="par">
           <div class="child"></div>
           <div class="child"></div>
       </div>
   </body>
   ```

**下列方式会创建块格式化上下文**：

2. float属性不为none
3. position为absolute或fixed
4. display为`inline-block, table-cell, table-caption, flex, inline-flex`
5. overflow不为visible

需要背的条件👇

1. 浮动元素（元素的 float 不是 none）
2. 绝对定位元素（元素的 position 为 absolute 或 fixed）
3. 行内块元素
4. overflow 值不为 visible 的块元素
5. 弹性元素（display为 flex 或 inline-flex元素的直接子元素）

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
// 定义动画的时间  duration 
// 动画的名称
// 动画的贝塞尔曲线
// animation-fill-mode 属性规定动画在播放之前或之后，其动画效果是否可见。 
// forwards  当动画完成后，保持最后一个属性值	
```

## 清除浮动

第一种父级元素添加伪元素

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

第二种给父级元素添加 `overflow:hidden` 或者 `auto` 样式

第三种给所有浮动标签后面添加一个空的 div,给 div 添加 clear:both 属性

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
   页面被加载时，link会同时被加载，而**@import引用的css会等到页面被加载完再加载 **。
3. 兼容性区别：
   import只在IE5以上才能识别，而link是html标签，无兼容问题。
4. dom可操作性区别：
   可以通过JS 操作 DOM ，插入link标签来改变样式；由于 DOM 方法是基于文档的，无法使用@import的方式插入样式
5. 权重区别：
   如果已经存在相同样式，@import引入的这个样式将被该 CSS 文件本身的样式层叠掉，表现出link方式的样式权重高于@import的权重这样的直观效果。
   （aSuncat：简而言之，link和@import，谁写在后面，谁的样式就被应用，后面的样式覆盖前面的样式。）

## 伪类和伪元素

伪类：:active :focus :hover :link :visited :first-child 

伪元素：:before :after :first-letter :first-line

## svg和canvas的区别

1. canvas画图基于像素点，是位图，如果进行放大或缩小会失真 ；

   svg基于图形，用html标签描绘形状，放大缩小不会失真 

2. canvas需要在js中绘制 ；svg在html绘制 

3. canvas支持颜色比svg多 

4. canvas无法对已经绘制的图像进行修改、操作 ；svg可以获取到标签进行操作