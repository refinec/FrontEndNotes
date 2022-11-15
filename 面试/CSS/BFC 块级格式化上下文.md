# BFC 块级格式化上下文

> **BFC是一个独立的布局环境，其中的元素布局是不受外界的影响**，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

## BFC的规则

1. `BFC`就是页面中的一个隔离的**独立容器**，**容器内外的元素互不影响**

2. `BFC`就是一个块级元素，块级元素会在垂直方向一个接一个的排列

3. 垂直方向的距离由margin决定。**属于同一个BFC的两个相邻Box的margin会发生重叠 **

   ```html
   <!--属于同一个BFC的两个相邻的Box会发生margin重叠，所以我们可以设置，两个不同的BFC，也就是我们可以让把第二个p用div包起来，然后激活它使其成为一个BFC -->
   <style>
       p {
           margin: 30px;
           background: yellow;
           width: 200px;
           line-height: 100px;
           text-align:center;
       }
       div{
           overflow: hidden; /* 激活为BFC */
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


4. 计算`BFC`的高度时，浮动元素也参与计算

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

5. BFC的区域不会与`float` 元素重叠

## BFC 触发条件

触发`BFC`条件的`CSS`属性：

1. `overflow: hidden`
2. `display: inline-block | flex | table | table-cell | flow-root`
3. `position: absolute | fixed`
4. `float` 不为 `none`

## BFC解决的问题

### 1. 两个元素 垂直`margin`外边距重叠问题

1. 如果是父子元素外边距重叠，在父元素上添加`overflow: hidden`或`display: flow-root`等即可
2. 如果是相邻元素外边距重叠，在另一个元素上包裹一个`div`即可

### 2. 使用`float`脱离文档流，高度塌陷问题

在浮动元素的相邻元素上添加触发BFC的css属性即可