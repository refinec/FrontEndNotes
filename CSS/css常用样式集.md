# css常用样式集

## 滚动条修改

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

## 修改input的placeholder

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

## 去掉*input[type="number"]*的默认样式

```css
// &::-webkit-outer-spin-button,
// &::-webkit-inner-spin-button {
    //   -webkit-appearance: none !important;
    // }

// input[type="number"] {
    //   -moz-appearance: textfield;
    // }
```

## 修改checkbox默认样式

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

