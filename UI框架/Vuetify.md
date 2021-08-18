# 一、显示辅助
| 设备                                    | 代码   | 类型                 | 像素范围           |
| :-------------------------------------- | :----- | :------------------- | :----------------- |
| Extra small (超小号)                    | **xs** | 小型号到大型号的手机 | < 600px            |
| Small (小号)                            | **sm** | 小型号到中型号的平板 | 600px > < 960px    |
| Medium (中号)                           | **md** | 大型号平板到手提电脑 | 960px > < 1264px*  |
| Large (大号)                            | **lg** | 桌面端               | 1264px > < 1904px* |
| Extra large (超大号)                    | **xl** | 4K 和超宽屏幕        | > 1904px*          |
| *桌面端上浏览器滚动条的宽度为 \* -16px* |        |                      |                    |

## `display`属性样式

>  当为显示辅助类设置一个特定断点时，它将应用于所有屏幕从指定值开始的宽度。 例如， `d-lg-flex` 将适用于 `lg` 和 `xl` 的屏幕尺寸。

* **使用**

  ```text
  .d-{value} 默认用于 xs，因为被推断为.d-${value}-xs
  .d-{breakpoint}-{value} 用于 sm, md, lg 和 xl
  ```

  **value**值包括：

  1. **`none`**
  2. **`block`**
  3. **`flex`**
  4. **`inline`**
  5. **`inline-block`**
  6. **`inline-flex`**
  7. **`table`**
  8. **`table-cell`**
  9. **`table-row`**

## 可见性(用于不同的分辨率)

> 根据当前 **viewport** 的宽度上限有条件的显示元素。 断点实用类始终**自下而上**应用。 这意味着如果你有 `.d-none`, 它将应用于所有断点。 然而， `.d-md-none` 将仅应用于 `md` 及以上。



| 屏幕大小             | 类                              |
| -------------------- | ------------------------------- |
| **全部隐藏**         | **`.d-none`**                   |
| 仅在 `xs` 大小时隐藏 | `.d-none .d-sm-flex`            |
| 仅在 `sm` 大小时隐藏 | `.d-sm-none .d-md-flex`         |
| 仅在 `md `大小时隐藏 | `.d-md-none .d-lg-flex`         |
| 仅在` lg `大小时隐藏 | `.d-lg-none .d-xl-flex`         |
| 仅在` xl `大小时隐藏 | `.d-xl-none`                    |
| **全部可见**         | **`.d-flex`**                   |
| 仅在` xs` 大小时可见 | `.d-flex .d-sm-none`            |
| 仅在` sm `大小时可见 | `.d-none .d-sm-flex .d-md-none` |
| 仅在 `md` 大小时可见 | `.d-none .d-md-flex .d-lg-none` |
| 仅在 `lg `大小时可见 | `.d-none .d-lg-flex .d-xl-none` |
| 仅在 `xl` 大小时可见 | `.d-none .d-xl-flex`            |

还可以使用横向显示辅助类基于当前 **viewport** 宽度上限来显示元素。 这些类可以使用以下格式 **`hidden-{breakpoint}-{condition}`** 使用。

基于以下 条件 应用类:

- `only` - 只在 `xs` 至 `xl` 断点隐藏元素
- `and down` - 在指定的断点和以下隐藏元素, 从 `sm` 到 `lg` 断点
- `and down` - 在指定的断点和以上隐藏元素, 从 `sm` 到 `lg` 断点

此外, 可以使用 `only` 条件确定目标 **媒体类型** 。 目前支持 `hidden-screen-only` 和 `hidden-print-only` 。

## 打印显示

在打印时更改显示属性。

- `.d-print-none`
- `.d-print-inline`
- `.d-print-inline-block`
- `.d-print-block`
- `.d-print-table`
- `.d-print-table-row`
- `.d-print-table-cell`
- `.d-print-flex`
- `.d-print-inline-flex`

## 无障碍(屏幕阅读器)

使用 `d-sr` 工具类在 *除* 屏幕阅读器外所有的设备上隐藏内容。

- `d-sr-only` 视觉隐藏元素但仍会通知 **屏幕阅读器** 。
- `d-sr-focusable` 视觉隐藏一个元素直到它获取焦点。 这在实现 *跳过链接* 时非常有用

# 二、elevation 海拔（即元素z-index属性）

> 使用 **`elevation-{n}`**类设置元素的海拔，总共有25个高度， 其中 `n` 是0-24之间与所需海拔对应的整数

```html
    <v-card :elevation="12"></v-card>
```

**在鼠标悬停时改变高度：**

```vue
<template>
  <div class="text--primary">
    <!-- Using the elevation prop -->
    <v-hover>
      <template v-slot:default="{ hover }">
        <v-card
          :elevation="hover ? 24 : 6"
          class="mx-auto pa-6"
        >
          Prop based elevation
        </v-card>
      </template>
    </v-hover>

    <div class="my-6"></div>

    <!-- Using a dynamic class -->
    <v-hover>
      <template v-slot:default="{ hover }">
        <div
          :class="`elevation-${hover ? 24 : 6}`"
          class="mx-auto pa-6 transition-swing"
        >
          Class based elevation
        </div>
      </template>
    </v-hover>
  </div>
</template>
```

# 三、flex弹性布局

- **`.d-flex`**
- **`.d-inline-flex`**
- **`.d-sm-flex`**
- **`.d-sm-inline-flex`**
- **`.d-md-flex`**
- **`.d-md-inline-flex`**
- **`.d-lg-flex`**
- **`.d-lg-inline-flex`**
- **`.d-xl-flex`**
- **.d-xl-inline-flex**

## 1.排列方向

* `flex-direction: row`使用为`flex-row`、`flex-row-reverse`
* `flex-direction: column`使用为`flex-column` 和 `flex-column-reverse`

### 基于断点响应变化

- **.flex-row**
- **.flex-row-reverse**
- **.flex-column**
- **.flex-column-reverse**
- **.flex-sm-row**
- **.flex-sm-row-reverse**
- **.flex-sm-column**
- **.flex-sm-column-reverse**
- **.flex-md-row**
- **.flex-md-row-reverse**
- **.flex-md-column**
- **.flex-md-column-reverse**
- **.flex-lg-row**
- **.flex-lg-row-reverse**
- **.flex-lg-column**
- **.flex-lg-column-reverse**
- **.flex-xl-row**
- **.flex-xl-row-reverse**
- **.flex-xl-column**
- **.flex-xl-column-reverse**

## 2.对齐

### `justify-content` 设置

- **.justify-start**
- **.justify-end**
- **.justify-center**
- **.justify-space-between**
- **.justify-space-around**
- **.justify-sm-start**
- **.justify-sm-end**
- **.justify-sm-center**
- **.justify-sm-space-between**
- **.justify-sm-space-around**
- **.justify-md-start**
- **.justify-md-end**
- **.justify-md-center**
- **.justify-md-space-between**
- **.justify-md-space-around**
- **.justify-lg-start**
- **.justify-lg-end**
- **.justify-lg-center**
- **.justify-lg-space-between**
- **.justify-lg-space-around**
- **.justify-xl-start**
- **.justify-xl-end**
- **.justify-xl-center**
- **.justify-xl-space-between**
- **.justify-xl-space-around**

### `align-items` 设置

- **.align-start**
- **.align-end**
- **.align-center**
- **.align-baseline**
- **.align-stretch**
- **.align-sm-start**
- **.align-sm-end**
- **.align-sm-center**
- **.align-sm-baseline**
- **.align-sm-stretch**
- **.align-md-start**
- **.align-md-end**
- **.align-md-center**
- **.align-md-baseline**
- **.align-md-stretch**
- **.align-lg-start**
- **.align-lg-end**
- **.align-lg-center**
- **.align-lg-baseline**
- **.align-lg-stretch**
- **.align-xl-start**
- **.align-xl-end**
- **.align-xl-center**
- **.align-xl-baseline**
- **.align-xl-stretch**
- **.align-start**
- **.align-end**
- **.align-center**
- **.align-baseline**
- **.align-stretch**
- **.align-sm-start**
- **.align-sm-end**
- **.align-sm-center**
- **.align-sm-baseline**
- **.align-sm-stretch**
- **.align-md-start**
- **.align-md-end**
- **.align-md-center**
- **.align-md-baseline**
- **.align-md-stretch**
- **.align-lg-start**
- **.align-lg-end**
- **.align-lg-center**
- **.align-lg-baseline**
- **.align-lg-stretch**
- **.align-xl-start**
- **.align-xl-end**
- **.align-xl-center**
- **.align-xl-baseline**
- **.align-xl-stretch**

### `align-self`子项自身设置

- **.align-self-start**
- **.align-self-end**
- **.align-self-center**
- **.align-self-baseline**
- **.align-self-auto**
- **.align-self-stretch**
- **.align-self-sm-start**
- **.align-self-sm-end**
- **.align-self-sm-center**
- **.align-self-sm-baseline**
- **.align-self-sm-auto**
- **.align-self-sm-stretch**
- **.align-self-md-start**
- **.align-self-md-end**
- **.align-self-md-center**
- **.align-self-md-baseline**
- **.align-self-md-auto**
- **.align-self-md-stretch**
- **.align-self-lg-start**
- **.align-self-lg-end**
- **.align-self-lg-center**
- **.align-self-lg-baseline**
- **.align-self-lg-auto**
- **.align-self-lg-stretch**
- **.align-self-xl-start**
- **.align-self-xl-end**
- **.align-self-xl-center**
- **.align-self-xl-baseline**
- **.align-self-xl-auto**
- **.align-self-xl-stretch**

### `align-content` 设置

> 内容对齐

- **align-content-start**
- **align-content-end**
- **align-content-center**
- **align-content-space-between**
- **align-content-space-around**
- **align-content-stretch**
- **align-sm-content-start**
- **align-sm-content-end**
- **align-sm-content-center**
- **align-sm-content-space-between**
- **align-sm-content-space-around**
- **align-sm-content-stretch**
- **align-md-content-start**
- **align-md-content-end**
- **align-md-content-center**
- **align-md-content-space-between**
- **align-md-content-space-around**
- **align-md-content-stretch**
- **align-lg-content-start**
- **align-lg-content-end**
- **align-lg-content-center**
- **align-lg-content-space-between**
- **align-lg-content-space-around**
- **align-lg-content-stretch**
- **align-xl-content-start**
- **align-xl-content-end**
- **align-xl-content-center**
- **align-xl-content-space-between**
- **align-xl-content-spacearound**
- **align-xl-content-stretch**



## 3.flex堆叠

- **.flex-nowrap**：即`flex-wrap:nowrap`
- **.flex-wrap**
- **.flex-wrap-reverse**

这些辅助类同样可以基于断点以 `flex-{breakpoint}-{condition}` 的格式创建更多的弹性变量. 以下组合可用：

- **.flex-sm-nowrap**
- **.flex-sm-wrap**
- **.flex-sm-wrap-reverse**
- **.flex-md-nowrap**
- **.flex-md-wrap**
- **.flex-md-wrap-reverse**
- **.flex-lg-nowrap**
- **.flex-lg-wrap**
- **.flex-lg-wrap-reverse**
- **.flex-xl-nowrap**
- **.flex-xl-wrap**
- **.flex-xl-wrap-reverse**

## 4.order排序

- **.order-first**
- **.order-0**
- **.order-1**
- **.order-2**
- **.order-3**
- **.order-4**
- **.order-5**
- **.order-6**
- **.order-7**
- **.order-8**
- **.order-9**
- **.order-10**
- **.order-11**
- **.order-12**
- **.order-last**
- **.order-sm-first**
- **.order-sm-0**
- **.order-sm-1**
- **.order-sm-2**
- **.order-sm-3**
- **.order-sm-4**
- **.order-sm-5**
- **.order-sm-6**
- **.order-sm-7**
- **.order-sm-8**
- **.order-sm-9**
- **.order-sm-10**
- **.order-sm-11**
- **.order-sm-12**
- **.order-sm-last**
- **.order-md-first**
- **.order-md-0**
- **.order-md-1**
- **.order-md-2**
- **.order-md-3**
- **.order-md-4**
- **.order-md-5**
- **.order-md-6**
- **.order-md-7**
- **.order-md-8**
- **.order-md-9**
- **.order-md-10**
- **.order-md-11**
- **.order-md-12**
- **.order-md-last**
- **.order-lg-first**
- **.order-lg-0**
- **.order-lg-1**
- **.order-lg-2**
- **.order-lg-3**
- **.order-lg-4**
- **.order-lg-5**
- **.order-lg-6**
- **.order-lg-7**
- **.order-lg-8**
- **.order-lg-9**
- **.order-lg-10**
- **.order-lg-11**
- **.order-lg-12**
- **.order-lg-last**
- **.order-lg-first**
- **.order-xl-0**
- **.order-xl-1**
- **.order-xl-2**
- **.order-xl-3**
- **.order-xl-4**
- **.order-xl-5**
- **.order-xl-6**
- **.order-xl-7**
- **.order-xl-8**
- **.order-xl-9**
- **.order-xl-10**
- **.order-xl-11**
- **.order-xl-12**
- **.order-xl-last**

## 5.grow放大和shrink收缩

- **flex-grow-0**
- **flex-grow-1**
- **flex-shrink-0**
- **flex-shrink-1**

`grow` 将允许元素增长以填充可用的空间, 然而 `shrink` 将允许项目收缩到它的内容所需要的空间. 但是，只有当项目必须收缩以适合其容器时才会发生这种情况，例如容器大小调整或受到 `flex-grow-1` 的影响。 值`0`将阻止该条件的发生，而`1`则允许出现这种情况。

可以基于断点以 `flex-{breakpoint}-{condition}-{state}` 的格式创建更多的弹性变量. 以下组合可用：

- **flex-sm-grow-0**
- **flex-md-grow-0**
- **flex-lg-grow-0**
- **flex-xl-grow-0**
- **flex-sm-grow-1**
- **flex-md-grow-1**
- **flex-lg-grow-1**
- **flex-xl-grow-1**
- **flex-sm-shrink-0**
- **flex-md-shrink-0**
- **flex-lg-shrink-0**
- **flex-xl-shrink-0**
- **flex-sm-shrink-1**
- **flex-md-shrink-1**
- **flex-lg-shrink-1**
- **flex-xl-shrink-1**

# 四、浮动

- **.float-left**
- **.float-right**
- **.float-start**
- **.float-end**
- **.float-none**
- **.float-sm-left**
- **.float-sm-right**
- **.float-sm-start**
- **.float-sm-end**
- **.float-sm-none**
- **.float-md-left**
- **.float-md-right**
- **.float-md-start**
- **.float-md-end**
- **.float-md-none**
- **.float-lg-left**
- **.float-lg-right**
- **.float-lg-start**
- **.float-lg-end**
- **.float-lg-none**
- **.float-xl-left**
- **.float-xl-right**
- **.float-xl-start**
- **.float-xl-end**
- **.float-xl-none**

# 五、内外边距

通过 **`{property}{direction}-{size}`** 格式使用

* **property** 应用间距类型
  - `m` - 应用 `margin`
  - `p` - 应用 `padding`
* **direction** 指定了该属性所应用的侧边:
  - `t` - 应用 `margin-top` 和 `padding-top` 的间距
  - `b` - 应用 `margin-bottom` 和 `padding-bottom` 的间距
  - `l` - 应用 `margin-left` 和 `padding-left` 的间距
  - `r` - 应用 `margin-right` 和 `padding-right` 的间距
  - `s` - 应用 `margin-left`/`padding-left` (LTR模式) 和 `margin-right`/`padding-right`(RTL模式) 的间距
  - `e` - 应用 `margin-right`/`padding-right` (LTR模式) 和 `margin-left`/`padding-left`(RTL模式) 的间距
  - `x` - 应用 `*-left` 和 `*-right` 的间距
  - `y` - 应用 `*-top` 和 `*-bottom` 的间距
  - `a` - 在所有方向应用该间距
* **size** 以4px增量控制间距属性:
  - `0` - 通过设置为 `0` 来消除所有 `margin` 或 `padding`.
  - `1` - 设置 `margin` 或 `padding` 为 4px
  - `2` - 设置 `margin` 或 `padding` 为 8px
  - `3` - 设置 `margin` 或 `padding` 为 12px
  - `4` - 设置 `margin` 或 `padding` 为 16px
  - `5` - 设置 `margin` 或 `padding` 为 20px
  - `6` - 设置 `margin` 或 `padding` 为 24px
  - `7` - 设置 `margin` 或 `padding` 为 28px
  - `8` - 设置 `margin` 或 `padding` 为 32px
  - `9` - 设置 `margin` 或 `padding` 为 36px
  - `10` - 设置 `margin` 或 `padding` 为 40px
  - `11` - 设置 `margin` 或 `padding` 为 44px
  - `12` - 设置 `margin` 或 `padding` 为 48px
  - `13` - 设置 `margin` 或 `padding` 为 52px
  - `14` - 设置 `margin` 或 `padding` 为 56px
  - `15` - 设置 `margin` 或 `padding` 为 60px
  - `16` - 设置 `margin` 或 `padding` 为 64px
  - `n1` - 设置 `margin` 为 -4px
  - `n2` - 设置 `margin` 为 -8px
  - `n3` - 设置 `margin` 为 -12px
  - `n4` - 设置 `margin` 为 -16px
  - `n5` - 设置 `margin` 为 -20px
  - `n6` - 设置 `margin` 为 -24px
  - `n7` - 设置 `margin` 为 -28px
  - `n8` - 设置 `margin` 为 -32px
  - `n9` - 设置 `margin` 为 -36px
  - `n10` - 设置 `margin` 为 -40px
  - `n11` - 设置 `margin` 为 -44px
  - `n12` - 设置 `margin` 为 -48px
  - `n13` - 设置 `margin` 为 -52px
  - `n14` - 设置 `margin` 为 -56px
  - `n15` - 设置 `margin` 为 -60px
  - `n16` - 设置 `margin` 为 -64px
  - `auto` - 设置间距为 **auto**

### 基于断点的边距

通过 `{property}{direction}-{breakpoint}-{size}` 格式使用。 这不适用于 **xs** ,因为它是推断出来的; 例如: `ma-xs-2` 等于 `ma-2`.

### 水平居中

```markup
mx-auto
```

# 六、文本和排版

## 排版

- `.text-{value}` 用于 `xs`
- `.text-{breakpoint}-{value}` 用于 `sm`, `md`, `lg` 和 `xl`

该 value 属性的值是以下之一：

- `h1`
- `h2`
- `h3`
- `h4`
- `h5`
- `h6`
- `subtitle-1`
- `subtitle-2`
- `body-1`
- `body-2`
- `button`
- `caption`
- `overline`

## 强调 font-weight

* `font-weight-black`
* `font-weight-bold`
* `font-weight-medium`
* `font-weight-regular`
* `font-weight-light`
* `font-weight-thin`
* `font-italic`

## 文本对齐

* `text-justify`
* `text-left`
* `text-center`
* `text-right`
* `text-sm-left`
* `text-md-left`
* `text-lg-left`
* `text-xl-left`

## 装饰线

* `.text-decoration-none` 移除文本装饰线
*  `.text-decoration-overline`上划线
* `.text-decoration-underline` 下划线
* ``.text-decoration-line-through` 删除线

## 不透明度

- `text--primary` 与默认文本具有相同的不透明度
-  `text--secondary` 用于提示和辅助文本。 
-  `text--disabled` 去除强调文本
- `text-break` 断开文本以适应width宽度

## 文本换行和溢出

* `.text-no-wrap`防止文本换行
* `.text-truncate`文本溢出省略

## RTL 对齐

`text-{breakpoint}-{direction}`,

* `breakpoint` 可以是 `sm`, `md`, `lg>` 或 `xl`
* ``direction` 可以是 `left` 或 `right`. 也可以通过使用 `start` 和 `end` 对齐rtl.
