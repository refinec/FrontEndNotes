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

