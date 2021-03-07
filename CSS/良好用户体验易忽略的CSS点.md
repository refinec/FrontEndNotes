## 1. 0内容展示

对于 0 结果页面，分清楚：

- **数据为空**：其中又可能包括了用户无权限、搜索无结果、筛选无结果、页面无数据
- **异常状态**：其中又可能包括了网络异常、服务器异常、加载失败等待

不同的情况可能对应不同的 0 结果页面，附带不同的操作引导。



## 2.处理动态内容 - 保护边界 使用 `min/max-width` 或 `min/max-height` 对容器的高宽限度进行合理的控制

```css
.btn {
    min-width: 88px;
    padding: 0 16px; /*使用padding保护边界*/
}
```

![](E:\MarkDown\FrontEndNotes\assets\css\2065661232-f02ca79a2516d6bc_articlex.png)

## 3. 图片相关

### 给图片同时设置高宽

> 给 `<img>` 标签同时写上高宽，可以在图片未加载之前提前占住位置，避免图片从未加载状态到渲染完成状态高宽变化引起的重排问题 以及 避免因为图片尺寸错误带来的布局问题

```css
img {
    width: 150px;
    height: 100px;
}
```

**借助 `object-fit`**解决限制高宽出现的问题(如图片拉伸)：

* `object-fit: cover`使图片内容在保持其宽高比的同时填充元素的整个内容框

* `object-position`控制图片在其内容框中的位置，类似于 `background-position`。默认是 `object-position: 50% 50%`居中显示

### 考虑屏幕 `dpr `-- 响应式图片

> 不同的屏幕有不同的`dpr`，这种时候需要考虑利用多倍图去适配不同 `dpr` 的屏幕。

**使用<img> 标签提供的属性 `srcset `让进行该骚操作,可以给不同 `dpr` 的屏幕，提供最适合的图片：**

* **旧写法**

  ```html
  <img src='photo@1x.png'
     srcset='photo@1x.png 1x,
             photo@2x.png 2x,
             photo@3x.png 3x' 
  />
  ```

* **新写法**

   使用 w  宽度描述符并配合 `sizes` 一起使用

  ```html
  <img 
          src = "photo.png" 
          sizes = “(min-width: 600px) 600px, 300px" 
          srcset = “photo@1x.png 300w,
                         photo@2x.png 600w,
                         photo@3x.png 1200w,
  >
  ```

### 图片丢失

**处理的方式:**

1. 利用图片加载失败，触发 `<img>` 元素的 `onerror` 事件，给加载失败的 `<img>` 元素新增一个样式类

2. 利用新增的样式类，配合 `<img>` 元素的伪元素，展示默认兜底图的同时，还能一起展示 `<img>` 元素的 `alt` 信息

   ```html
   <img src="test.png" alt="图片描述" onerror="this.classList.add('error');">
   ```

   ```css
   img.error {
       position: relative;
       display: inline-block;
   }
   
   img.error::before { /*伪元素 before ，加载默认错误兜底图*/
       content: "";
       /** 定位代码 **/
       background: url(error-default.png);
   }
   
   img.error::after { /*利用伪元素 after，展示图片的 alt 信息*/
       content: attr(alt);
       /** 定位代码 **/
   }
   ```

   ![](../assets/css/792503157-0906bbb70e71c5f8_articlex.png)