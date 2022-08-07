## 逐帧动画(loading)

```html
<div class="loader">Loading…</div>
```

```css
/**
 * Frame-by-frame animations
 */

@keyframes loader {
	to { background-position: -800px 0; }
}

.loader {
	width: 100px;
    height: 100px;
	text-indent: 999px;
    overflow: hidden; /* 隐藏文本 */
	background: url(http://dabblet.com/img/loader.png) 0 0;
	animation: loader 1s infinite steps(8); /* loading由8张图片组成，所以steps步长为8 */
}
```

loading图片：

![img](../../../assets/css/loader.png)

### 效果

![](../../../assets/css/frame-by-frame.gif)