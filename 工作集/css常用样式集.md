# css常用样式集

## 1.滚动条修改

```css
/*滚动条宽 长,滚动条整体部分，其中的属性有width,height,background,border等。*/
需滚动的元素::-webkit-scrollbar{
    width:0; /* 0为隐藏，有宽度后可设置以下属性*/
}
/*滚动条的滑轨背景颜色,可以用display:none让其不显示，也可以添加背景图片，颜色改变显示效果。*/
需滚动的元素::-webkit-scrollbar-track{
    background-color: #f5f5f5;
    -webkit-box-shadow:inset 0 0 3px rgba(0,0,0,0.1);
    border-radius:5px;
}
/* 滑块颜色 */
需滚动的元素::-webkit-scrollbar-thumb{
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 5px;
}
/*滚动条两端的按钮。可以用display:none让其不显示，也可以添加背景图片，颜色改变显示效果。*/
需滚动的元素::-webkit-scrollbar-button{
    background-color: #eee;
    display: none;
}
/* 横向滚动条和纵向滚动条相交处尖角的颜色 */
需滚动的元素::-webkit-scrollbar-corner{
    background-color: black;
}
```



隐藏`div`元素的滚动条

```css
div::-webkit-scrollbar {
    display: none;
}
```

- `div::-webkit-scrollbar` 滚动条整体部分
- `div::-webkit-scrollbar-thumb` 滚动条里面的小方块，能向上向下移动（或往左往右移动，取决于是垂直滚动条还是水平滚动条）
- `div::-webkit-scrollbar-track` 滚动条的轨道
- `div::-webkit-scrollbar-button` 滚动条的轨道的两端按钮，允许通过点击微调小方块的位置。
- `div::-webkit-scrollbar-track-piece` 内层轨道，滚动条中间部分
- `div::-webkit-scrollbar-corner` 边角，即两个滚动条的交汇处
- `div::-webkit-resizer` 两个滚动条的交汇处上用于通过拖动调整元素大小的小控件

注意此方案有兼容性问题，一般需要隐藏滚动条时我都是用一个色块通过定位盖上去，或者将子级元素调大，父级元素使用`overflow-hidden`截掉滚动条部分。暴力且直接。

## 2.修改input的placeholder

```css
::-webkit-input-placeholder { /* WebKit, Blink, Edge */
    color:    #909;
}
:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
   color:    #909;
   opacity:  1;
}
:-moz-placeholder { /* Mozilla Firefox 19+ */
   color:    #909;
   opacity:  1;
}
:-ms-input-placeholder { /* Internet Explorer 10-11 */
   color:    #909;
}
::-ms-input-placeholder { /* Microsoft Edge */
   color:    #909;
}
```

## 3.去掉*input[type="number"]*的默认样式

```css
// &::-webkit-outer-spin-button,
// &::-webkit-inner-spin-button {
    //   -webkit-appearance: none !important;
    // }

// input[type="number"] {
    //   -moz-appearance: textfield;
    // }
```

## 4.修改checkbox默认样式

```html
<input type="checkbox" id="checkbox_sel" class="selectBetting" />
<label for="checkbox_sel" class=""></label>
```

```css
.selectBetting{ 
  display: none; 
} 
.selectBetting + label { 
  background: url(../images/checkbox.png) no-repeat;
  width: 64px;
  height: 64px;
  display: inline-block; 
  float: left;
  cursor: pointer;
}  
.selectBetting:checked + label { 
  background: url(../images/checkbox.png) no-repeat;
} 
.selectBetting:checked + label::after { 
  content: '√';  
  float: left;
  font-family: "customfont";
  color: #6be9ff; 
  width: 100%; 
  text-align: center; 
  font-size: 32px;
  padding: 10px 0 0 0px;
  vertical-align: text-top; 
}
```

## 5.文本超长

### 单行文本 使用单行省略

```css
width: 200px;
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

### 多行文本的超长省略

```css
width: 200px;
overflow : hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 2; /*两行*/
-webkit-box-orient: vertical;
```

## 6. 将文本内容分为多列并添加边框

```css
p {
    columns: 3;
    column-rule: solid 2px #222; /*类似border-right作用*/
}
```

![](../assets/css/columns.jpg)

## 7. `li`项旁边的默认小圆圈称为**marker**

```css
li::marker {
    color: #ccc;
}
```

## 8.input输入框聚焦时，输入框占位符内容以动画形式移动到左上角作为标题

```html
<div class="input-box"> 
    <input class="input-control input-outline" placeholder="账号">
    <label class="input-label">账号</label>
</div>
```

1. 首先让浏览器默认的`placeholder`效果不可见 

   ```css
   .input-control:placeholder-shown::placeholder { 
       color: transparent; 
   }
   ```

2. 使用`.input-label`元素代替input 的占位符

   ```css
   .input-box{
     position: relative;
   }
   .input-label {
     position: absolute;
     left: 16px; top: 14px;
     pointer-events: none;
   }
   ```

3. 最后在输入框聚焦以及占位符不显示的时候对`<label>`元素进行重定位，效果是缩小并移动到上方

   ```css
   .input-control:not(:placeholder-shown) ~ .input-label,
   .input-control:focus ~ .input-label {
     color: #2486ff;
     transform: scale(0.75) translate(-2px, -32px);
   }
   ```

## 9. 彩色粉笔字

```html
<head>
    <link href="https://fonts.googleapis.com/css?family=Cabin+Sketch:700&display=swap" rel="stylesheet">
</head>
<body>
    <div class="text-effect">
        <span>B</span><span>e</span><span>s</span><span>t</span>
        <span>J</span><span>Q</span><span>u</span><span>e</span>		<span>r</span><span>y</span>
    </div>
</body>
```

```css
.text-effect{
	color: #ffff00;
	font-family: 'Cabin Sketch', cursive;
	font-size: 100px;
	text-align: center;
	text-transform: uppercase;
	margin: 0 auto;
	position: relative;
}
.text-effect span{
	display: inline-block;
	animation: animate 0.5s linear infinite both;
}
.text-effect span:nth-child(1),
.text-effect span:nth-child(4),
.text-effect span:nth-child(7),
.text-effect span:nth-child(10){
   color: #4cd137;
}
.text-effect span:nth-child(2),
.text-effect span:nth-child(5),
.text-effect span:nth-child(8){
   color: #ff0000;
}
@keyframes animate{
	0%, 50%, 100%{ transform: rotate(0deg) scale(1); }
	25%{ transform: rotate(4deg) scale(0.98); }
	75%{ transform: rotate(-4deg) scale(1.02); }
}
```

## 10. Affix固钉 sticky position

```html
<div class="static">Static</div>
<div class="sticky">Sticky</div>
```

```css
.sticky {
  position: sticky;
  top: 0;
  width: 100%;
  
  padding: 1em 0;
  background-color: #149bdf;
  background-image: linear-gradient(-45deg, rgba(255, 255, 255, 0.15) 25%, transparent 25%, transparent 50%, rgba(255, 255, 255, 0.15) 50%, rgba(255, 255, 255, 0.15) 75%, transparent 75%, transparent);
  background-size: 100px 100px;
  color: white;
  text-align: center;
}

.fixed {
  position: fixed;
}

.static {
  padding: 5em 0;
  background: #1085bc;
  color: white;
  text-align: center;
}

:root {
  height: 1000%;
  font-family: sans-serif;
}
```

```js
var sticky = document.querySelector('.sticky');

if (sticky.style.position !== 'sticky') {
  var stickyTop = sticky.offsetTop;

  document.addEventListener('scroll', function () {
    window.scrollY >= stickyTop ?
      sticky.classList.add('fixed') :
      sticky.classList.remove('fixed');
  });
}
```

## 11. Clip-path 属性添加shadow

```html
<!DOCTYPE html>
<html>
  <body>
    <div class="container">
      <div class="shape"></div>
    </div>
  </body>
  <style>
    
    .container{
      filter: drop-shadow(0px 10px 5px rgba(0,0,0,0.5));
    }
    .shape{
      -webkit-clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
      clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
      background:red;
      width:100px;
      height:100px;     
    }
  </style>
</html>
```

## 12. 修改div为空时的placeholder

```scss
div{
  min-height: 50px;
  background: #eeeeee45;
  border-radius: 2px;
  border: 2px dashed #eeeeee;
  position: relative;
  &:empty::before {
    // content: attr(data-placeholder); 
    content: "[line " attr(data-placeholder) "]"; // 拼接字符串
    color: lightgrey;
    width: fit-content;
    display: inline-block;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate3d(-50%, -50%, 0);
    font-size: 1rem;
    letter-spacing: 2px;
  }
}
```

## 13. 丝带

<img src="../assets/css/6种CSS丝带.png" style="zoom:50%;" />

```html
<div>
 <section>
  <h2><span>CSS3 丝带</span></h2>
  <div class="ribbon"><span class="ribbon1"><span>丝带效果1</span></span></div>
  <div class="ribbon"><span class="ribbon2">丝<br>带<br>效<br>果<br>2</span></div>
  <div class="ribbon"><span class="ribbon3">丝带效果3</span></div>
  <div class="ribbon"><span class="ribbon4">丝带效果4</span></div>
  <div class="ribbon"><span class="ribbon5">丝带效果5</span></div>
  <div class="ribbon"><div class="wrap"><span class="ribbon6">丝带效果6</span></div></div>
 </section>
</div>
```

1. **丝带1**

   ```css
   .ribbon1 {
       position: absolute;
       top: -6px;
       right: 10px;
   }
   .ribbon1:after {
       position: absolute;
       content: "";
       display: block;
       width: 0;
       height: 0;
       border-left: 53px solid transparent;
       border-right: 53px solid transparent;
       border-top: 10px solid #F8463F;
   }
   .ribbon1 span {
       position: relative;
       display: inline-block;
       text-align: center;
       background: #F8463F;
       font-size: 14px;
       line-height: 1;
       padding: 12px 8px 10px;
       border-top-right-radius: 8px;
       width: 90px;
   }
   .ribbon1 span:before, .ribbon1 span:after {
       position: absolute;
       content: "";
       display: block;
   }
   .ribbon1 span:before {
       background: #F8463F;
       height: 6px;
       width: 6px;
       left: -6px;
       top: 0;
   }
   .ribbon1 span:after {
       background: #C02031;
       height: 6px;
       width: 8px;
       border-radius: 8px 8px 0 0;
       left: -8px;
       top: 0;
   }
   ```

2. **丝带2**

   ```css
   .ribbon2 {
     display: inline-block;
     width: 60px;
     padding: 10px 0;
     background: #F47530;
     top: -6px;
     left: 25px;
     position: absolute;
     text-align: center;
     border-top-left-radius: 3px;
   }
   .ribbon2:before {
      height: 0;
      width: 0;
      border-bottom: 6px solid #8D5A20;
      border-right: 6px solid transparent;
      right: -6px;
      top: 0;
   }
   .ribbon2:before, .ribbon2:after {
       content: "";
       position: absolute;
   }
   .ribbon2:after {
       height: 0;
       width: 0;
       border-left: 30px solid #F47530;
       border-right: 30px solid #F47530;
       border-bottom: 30px solid transparent;
       bottom: -30px;
       left: 0;
   }
   ```

3. **丝带3**

   ```css
   .ribbon3 {
     display: inline-block;
     position: absolute;
     width: 150px;
     height: 50px;
     line-height: 50px;
     padding-left: 15px;
     background: #59324C;
     left: -8px;
     top: 20px
   }
   .ribbon3:before, .ribbon3:after {
       content: "";
       position: absolute;
   }
   .ribbon3:before {
     height: 0;
     width: 0;
     border-bottom: 8px solid black;
     border-left: 8px solid transparent;
     top: -8px;
     left: 0;
   }
   .ribbon3:after {
     height: 0;
     width: 0;
     border-top: 25px solid transparent;
     border-bottom: 25px solid transparent;
     border-left: 15px solid #59324C;
     right: -15px;
   }
   ```

4. **丝带4**

   ```css
   .ribbon4 {
       position: absolute;
       top: 15px;
       padding: 8px 10px;
       background: #00B3ED;
       box-shadow: -1px 2px 4px rgba(0,0,0,0.5);
     }
   .ribbon4:before, .ribbon4:after {
       position: absolute;
       content: "";
       display: block;
   }
   .ribbon4:before {
       width: 7px;
       height: 100%;
       padding: 0 0 7px;
       top: 0;
       left: -7px;
       background: inherit;
       border-radius: 5px 0 0 5px;
   }
   .ribbon4:after {
       width: 5px;
       height: 5px;
       background: rgba(0,0,0,0.35);
       bottom: -5px;
       left: -5px;
       border-radius: 5px 0 0 5px;
    }
   ```

5. **丝带5**

   ```css
   .ribbon5 {
     display: inline-block;
     width: calc(100% + 20px);
     height: 50px;
     line-height: 50px;
     text-align: center;
     margin-left: -10px;
     margin-right: -10px;
     background: #EDBA19;
     position: relative;
     top: 20px;
   }
   .ribbon5:before {
     content: "";
     position: absolute;
     height: 0;
     width: 0;
     border-top: 10px solid #cd8d11;
     border-left: 10px solid transparent;
     bottom: -10px;
     left: 0;
   }
   .ribbon5:after {
     content: "";
     position: absolute;
     height: 0;
     width: 0;
     border-top: 10px solid #cd8d11;
     border-right: 10px solid transparent;
     right: 0;
     bottom: -10px;
   }
   ```

6. **丝带6**

   ```css
   .wrap {
     width: 100%;
     height: 188px;
     position: absolute;
     top: -8px;
     left: 8px;
     overflow: hidden;
   }
   .wrap:before {
       content: "";
       display: block;
       border-radius: 8px 8px 0px 0px;
       width: 40px;
       height: 8px;
       position: absolute;
       right: 100px;
       background: #4D6530;
   }
   .wrap:after {
       content: "";
       display: block;
       border-radius: 0px 8px 8px 0px;
       width: 8px;
       height: 40px;
       position: absolute;
       right: 0px;
       top: 100px;
       background: #4D6530;
   }
   .ribbon6 {
     display: inline-block;
     text-align: center;
     width: 200px;
     height: 40px;
     line-height: 40px;
     position: absolute;
     top: 30px;
     right: -50px;
     z-index: 2;
     overflow: hidden;
     transform: rotate(45deg);
     -ms-transform: rotate(45deg);
     -moz-transform: rotate(45deg);
     -webkit-transform: rotate(45deg);
     -o-transform: rotate(45deg);
     border: 1px dashed;
     box-shadow:0 0 0 3px #57DD43,  0px 21px 5px -18px rgba(0,0,0,0.6);
    background: #57DD43;
   }
   ```


## 14. 三角形角标

元素宽高设置为0，通过`border`属性来设置，让其它三个方向的`border`颜色为透明或者和背景色保持一致，剩余一条`border`的颜色设置为需要的颜色。

```css
div {
    width: 0;
    height: 0;
    border: 5px solid transparent;
    border-top-color: red;
}
```

