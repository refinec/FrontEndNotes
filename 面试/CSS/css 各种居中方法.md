# css 各种居中方法

## 水平居中

###  1.block 子元素定宽 + margin

```html
<style>
    .parent {
        height: 200px;
    }
    .child {
        width: 150px;
        margin: 0 auto; 
    }
</style>
<div class="parent">
    <div class="child">children 子元素</div>
</div>
```

### 2.inline-block/inline 子元素 + text-align

inline 类型的元素设置高宽是无效的,因此也没有定不定宽的说法

```html
<style>
    .parent2 {
        height: 200px;
        text-align: center;
    }
    .child2 {
        /* display: inline; */
        display: inline-block;
    }
</style>
<div class="parent2">
    <div class="child2">children 子元素</div>
</div>
```

## 水平垂直居中

### *3.flex/gird + 子元素不定宽高

```html
<style>
    .parent5 {
        /* display: grid; */
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .child5 {

    }
</style>
<div class="parent5">
    <div class="child5">children 子元素</div>
</div>
```

### *4.flex/gird + 子元素不定宽高 + margin

```html
<style>
    .parent6 {
        /* display: grid; */
        display: flex;
    }
    .child6 {
        margin: auto;
    }
</style>
<div class="parent6">
    <div class="child6">children 子元素</div>
</div>
```

### *5. gird + 子元素不定宽高(居中属性设置在子元素上)

```html
<style>
    .parent7-2 {
        display: grid;
    }
    .child7-2 {
        justify-self: center;
        align-self: center;
    }
</style>
<div class="parent7-2">
    <div class="child7-2">children 子元素</div>
</div>
```

### 6.absolute + 子元素定宽高 + margin

```html
<style>
    .parent9 {
        position: relative;
    }
    .child9 {
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        top: 0;
        width: 150px;
        height: 100px;
        margin: auto;
        font-size: 32px;
    }
</style>
<div class="parent9">
    <div class="child9">children 子元素</div>
</div>
```

### 7.absolute + 子元素不定宽高 + margin

fit-content 这属性具有兼容问题

```html
<style>
    .parent10 {
        position: relative;
    }
    .child10 {
        margin: auto;
        position: absolute;
        width: fit-content;
        height: fit-content;
        left: 0;
        right: 0;
        bottom: 0;
        top: 0;
    }
</style>
<div class="parent10">
    <div class="child10">children 子元素</div>
</div>
```

### *8.absolute + 子元素不定宽高 + transform

```html
<style>
    .parent11 {
        position: relative;
        height:200px
    }
    .child11 {
        /*margin: auto;*/
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
    }
</style>
<div class="parent11">
    <div class="child11">children 子元素</div>
</div>
```

### *9.table-cell + 子元素不定宽高 + text-align + vertical-align

```html
<style>
    .parent12 {
        /* width 设置百分比会失效,如果宽度不设置就由子元素的内容宽度决定 */
        width: 1000px;
        display: table-cell;
        text-align: center;
        vertical-align: middle;
    }
    .child12 {

    }
</style>
<div class="parent12">
    <div class="child12">children 子元素</div>
</div>
```

### *10.inline/inline-block 子元素不定宽高 + vertical-align

```html
<style>
    .parent13 {
        text-align: center;
    }
    .parent13::before {
        content: "";
        line-height: 200px;
        font-size: 0;
    }
    .child13 {
        /* display: inline-block; */
        display: inline;
        vertical-align: middle
    }
</style>
<div class="parent13">
    <div class="child13">children 子元素</div>
</div>
```

### *11.writing-mode + inline/inline-block 子元素不定宽高 + text-align

这个方法受默认排版影响,也是有兼容问题的

```html
<style>
    .parent15 {
        writing-mode: vertical-lr;
        text-align: center;
    }
    .child15 {
        font-size: 32px;
        width: 100%;
        writing-mode: horizontal-tb;
        /* display: inline; */
        display: inline-block;
    }
</style>
<div class="parent15">
    <div class="child15">children 子元素</div>
</div>
```

### *12.block 子元素定宽 + 父元素高度由子元素决定 + padding/margin

```html
<style>
    .parent16 {
        
    }
    .child16 {
        font-size: 32px;
        width: 150px;
        /* margin: auto; */
        /* padding: 50px 0; */
        margin: 50px auto;
    }
</style>
<div class="parent16">
    <div class="child16">children 子元素</div>
</div>
```

### *13.子元素不定宽高 + 父元素高度由子元素决定 + line-height

```html
<style>
    .parent17 {
        text-align: center;
    }
    .child17 {
        font-size: 32px;
        /* display: inline-block; */
        display: inline;
        line-height: 200px;
    }
</style>
<div class="parent17">
    <div class="child17">children 子元素</div>
</div>
```

### *14.正方形十字居中

```html
<style>
    .parent18 {
        border: 1px solid #000;
        background-color: rgb(214, 120, 52);
        width: 20%;
        position: relative;
    }
    .parent18::before {
        content: "";
        display: block;
        width: 0;
        height: 0;
        padding: 50% 0;
    }

    .child18::before, .child18::after {
        content: "";
        display: block;
        position: absolute;
        background-color: chartreuse;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
    }
    .child18::before {
        width: 50%;
        height: 5%;
    }
    .child18::after {
        height: 50%;
        width: 5%;
    }
</style>
<div class="parent18">
    <div class="child18"></div>
</div>
```

