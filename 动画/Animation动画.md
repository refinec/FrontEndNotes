## [CSS animation](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation)

> CSS **animation** 属性是 [`animation-name`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-name)，[`animation-duration`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-duration), [`animation-timing-function`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-timing-function)，[`animation-delay`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-delay)，[`animation-iteration-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-iteration-count)，[`animation-direction`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-direction)，[`animation-fill-mode`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-fill-mode) 和 [`animation-play-state`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-play-state) 属性的一个简写属性形式。
>
> `animation: name duration timing-function delay iteration-count direction fill-mode play-state;`
>
> `animation: myAnim 1s linear 1s infinite alternate both running;`

* `animation-name` 动画帧名称

  指定一个或多个 [`@keyframes`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@keyframes) at-rule 的名称。多个动画名称用`,`分开

* `animation-duration` 动画完成一个周期所需的时间

  用秒（`s`）或毫秒（`ms`）指定。值必须是正数或零，单位是必需的。

* `animation-timing-function` 动画函数

  `cubic-bezier`函数曲线定制网址：http://cubic-bezier.com

  ```css
  /* 关键字值 */
  animation-timing-function: ease; /*等同于 cubic-bezier(0.25, 0.1, 0.25, 1.0)，即默认值，表示动画在中间加速，在结束时减速*/
  animation-timing-function: ease-in; /*等同于 cubic-bezier(0.42, 0, 1.0, 1.0)，表示动画一开始较慢，随着动画属性的变化逐渐加速，直至完成*/
  animation-timing-function: ease-out; /*等同于 cubic-bezier(0, 0, 0.58, 1.0)，表示动画一开始较快，随着动画的进行逐渐减速*/
  animation-timing-function: ease-in-out; /*等同于 cubic-bezier(0.42, 0, 0.58, 1.0)，表示动画属性一开始缓慢变化，随后加速变化，最后再次减速变化*/
  animation-timing-function: linear; /*等同于 cubic-bezier(0.0, 0.0, 1.0, 1.0)，表示动画以匀速运动*/
  animation-timing-function: step-start;
  animation-timing-function: step-end;
  
  /* 函数值 */
  animation-timing-function: cubic-bezier(0.1, 0.7, 1, 0.1);/*自定义的三次贝塞尔曲线，其中 p1 和 p3 的值必须在 0 到 1 的范围内*/
  
  /* Steps 函数关键字 */
  /*按照 n 个定格在过渡中显示动画迭代，每个定格等长时间显示。例如，如果 n 为 5，则有 5 个步骤。动画是否在 0%、20%、40%、60% 和 80% 处或 20%、40%、60%、80% 和 100% 处暂停，或者在动画的 0% 和 100% 之间设置 5 个定格，又或是在包括 0% 和 100% 的情况下设置 5 个定格（在 0%、25%、50%、75% 和 100% 处）取决于使用以下跳跃项之一*/
  animation-timing-function: steps(4, jump-start);/*jump-start:表示一个左连续函数，因此第一个跳跃发生在动画开始时*/
  animation-timing-function: steps(10, jump-end);/*jump-end:表示一个右连续函数，因此最后一个跳跃发生在动画结束时*/
  animation-timing-function: steps(20, jump-none);/*两端都没有跳跃。相反，在 0% 和 100% 标记处分别停留，每个停留点的持续时间为总动画时间的 1/n*/
  animation-timing-function: steps(5, jump-both);/*在 0% 和 100% 标记处停留，有效地在动画迭代过程中添加一个步骤*/
  animation-timing-function: steps(6, start);/*等同于 jump-start*/
  animation-timing-function: steps(8, end);/*等同于 jump-end*/
  
  /* 多个动画 */
  animation-timing-function: ease, step-start, cubic-bezier(0.1, 0.7, 1, 0.1);
  
  /* 全局值 */
  animation-timing-function: inherit;
  animation-timing-function: initial;
  animation-timing-function: revert;
  animation-timing-function: revert-layer;
  animation-timing-function: unset;
  ```

* `animation-delay` 动画延迟开始时间

  用秒（`s`）或毫秒（`ms`）指定。

* `animation-iteration-count` 动画播放次数

  ```css
  /* 关键字值 */
  animation-iteration-count: infinite;
  
  /* 数字值 */
  animation-iteration-count: 3;
  animation-iteration-count: 2.4;
  
  /* 多个值 */
  animation-iteration-count: 2, 0, infinite;
  
  /* 全局值 */
  animation-iteration-count: inherit;
  animation-iteration-count: initial;
  animation-iteration-count: revert;
  animation-iteration-count: revert-layer;
  animation-iteration-count: unset;
  ```

* `animation-direction` 动画播放方向

  > 设置动画是应正向播放、反向播放还是在正向和反向之间交替播放。

  ```css
  /* 单个动画 */
  animation-direction: normal;/*动画在每个循环中正向播放。换言之，每次动画循环时，动画将重置为起始状态并重新开始。这是默认值*/
  animation-direction: reverse;/*动画在每个循环中反向播放。换句话说，每次动画循环时，动画将重置为结束状态并重新开始。动画步骤将反向执行，并且时间函数也将被反转。例如，ease-in 时间函数变为 ease-out*/
  animation-direction: alternate;/*动画在每个循环中正反交替播放，第一次迭代是正向播放。确定循环是奇数还是偶数的计数从 1 开始*/
  animation-direction: alternate-reverse;/*动画在每个循环中正反交替播放，第一次迭代是反向播放。确定循环是奇数还是偶数的计数从 1 开始*/
  
  /* 多个动画 */
  animation-direction: normal, reverse;
  animation-direction: alternate, reverse, normal;
  
  /* 全局值 */
  animation-direction: inherit;
  animation-direction: initial;
  animation-direction: revert;
  animation-direction: revert-layer;
  animation-direction: unset;
  ```

* `animation-fill-mode` 设置 CSS 动画在执行之前和之后如何将样式应用于其目标

  ```css
  /* Single animation */
  animation-fill-mode: none;/*当动画未执行时，动画将不会将任何样式应用于目标，而是已经赋予给该元素的 CSS 规则来显示该元素*/
  animation-fill-mode: forwards;/*目标将保留由执行期间遇到的最后一个关键帧计算值*/
  animation-fill-mode: backwards;/*动画将在应用于目标时立即应用第一个关键帧中定义的值，并在animation-delay期间保留此值*/
  animation-fill-mode: both;/*动画将遵循forwards和backwards的规则，从而在两个方向上扩展动画属性*/
  
  /* Multiple animations */
  animation-fill-mode: none, backwards;
  animation-fill-mode: both, forwards, none;
  ```

* `animation-play-state` 动画播放状态：运行或暂停

  ```css
  /* 单个动画 */
  animation-play-state: running;
  animation-play-state: paused;
  
  /* 多个动画 */
  animation-play-state: paused, running, running;
  
  /* 全局值 */
  animation-play-state: inherit;
  animation-play-state: initial;
  animation-play-state: revert;
  animation-play-state: revert-layer;
  animation-play-state: unset;
  
  ```

### 保留兼容性厂商前缀

```css
@-webkit-keyframes myAnim {}
@keyframes myAnim {}

div {
	-webkit-animation: myAnim 1s;
	animation: myAnim 1s;
}
```

## [Web Animations API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Animations_API)

> 包含[`Animation`](https://developer.mozilla.org/zh-CN/docs/Web/API/Animation)、[`KeyframeEffect`](https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect)、[`AnimationTimeline`](https://developer.mozilla.org/zh-CN/docs/Web/API/AnimationTimeline)、[`DocumentTimeline`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentTimeline)、、

### [`Element.animate()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/animate)

> 元素的**`animate()`** 方法创建一个新的 [`Animation`](https://developer.mozilla.org/zh-CN/docs/Web/API/Animation) 对象实例，将它应用于元素，然后运行动画。

`Animation`对应的属性和方法👉🏻：https://developer.mozilla.org/zh-CN/docs/Web/API/Animation

语法格式：`animate(keyframes, options)`

* `keyframes`

  关键帧对象数组，**或**一个关键帧对象（其属性为可迭代值的数组）。参见[关键帧格式](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Animations_API/Keyframe_Formats)

  ```json
  [
    {
      // from
      opacity: 0,
      color: "#fff",
    },
    { 
      opacity: 0.1, 
      offset: 0.7, // 关键帧的偏移.相当于播放到70%时应用该帧
    },
    {
      // to
      opacity: 1,
      color: "#000",
    },
  ]
  ```

  * `offset` 的值必须是在 **[0.0, 1.0]** 这个区间内，且须升序排列。并非所有的关键帧都需要设置 `offset`。没有指定 `offset` 的关键帧将与相邻的关键帧均匀间隔。

  * 可以通过提供`easing`过渡来给指定关键帧之间应用过渡效果，如下所示：

    ```js
    element.animate(
      [
        { opacity: 1, easing: "ease-out" },
        { opacity: 0.1, easing: "ease-in" },
        { opacity: 0 },
      ],
      2000,
    );
    ```

  * `composite`：[`KeyframeEffect.composite`](https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect/composite) 操作用于将此关键帧中指定的值与基础值组合在一起。如果正在使用在效果上指定的复合操作，则不会出现这种情况。
  * 属性名使用[驼峰式命名法](https://developer.mozilla.org/zh-CN/docs/Glossary/Camel_case)指定。但有两个特色的 css 属性：
    - [`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float), 必须写成 `cssFloat` ，因为"float" 是 JavaScript 的保留关键字
    - [`offset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/offset), 必须写成 `cssOffset` ，因为"offset" 表示如下的关键帧 offset

* `options`

  1. 一个表示动画持续时间的整数（以毫秒为单位）。如2000

  2. 或一个包含以下一个或多个元素的对象

     * `delay`: 延迟开始时间(`毫秒`)

     * `direction`: 动画方向（`normal`、`reverse`、`alternate`、`alternate-reverse`）

     * `duration`：完成一次动画周期所需的时间(`毫秒`)

       如果此值为 0，则动画将不会运行。

     * `easeing`：动画函数（`linear` 、 `ease-in` 、 `step-end` 、`cubic-bezier(0.42, 0, 0.58, 1)`等）

     * `endDelay`：动画结束后延迟的毫秒数。主要用于根据另一个动画的结束时间进行动画排序时。默认为 0

     * `fill`：设置 CSS 动画在执行之前和之后如何将样式应用于其目标。(`backwards`、`forwards`、`both`、`none`)

     * `iterationStart`：描述动画应在迭代的哪个点开始。

       例如，0.5 表示在第一次迭代的一半开始，当设置此值时，具有 2 次迭代的动画将在第三次迭代的一半结束。默认为 0.0

     * `iterations`：动画重复的次数。默认为 `1` ，也可以取值为 `Infinity` 。

     * `composite`：用于定义动画效果如何与目标元素的现有样式或其他动画效果进行合成。（在处理多个动画或动画与现有样式冲突时非常有用）

       1. **`replace`**（默认值）：动画效果会完全覆盖目标元素的现有样式或其他动画效果。这意味着动画的最终效果将完全取代之前的任何状态。（适用于需要完全控制动画效果，且不希望保留任何现有状态的场景）
       2. **`add`**：动画效果会追加到目标元素的现有样式或其他动画效果之后。这允许动画效果与现有的样式或动画效果叠加，但不会完全覆盖它们。（适用于需要在现有动画或样式基础上添加额外动画效果的场景。例如，一个元素已经在移动，你希望它同时旋转）
       3. **`accumulate`**：动画效果会与目标元素的现有样式或其他动画效果进行累加。这与 `add` 类似，但更强调效果的累积，尤其是在多次迭代或多个动画同时作用时。（适用于需要累积动画效果的场景，例如多次迭代动画时，希望每次迭代的效果都累积起来）

       假设一个元素的初始样式为：

       ```css
       div {
         transform: translateX(50px) rotate(45deg);
       }
       
       @keyframes adjust {
         to {
           transform: translateX(100px);
         }
       }
       ```

       - **`replace`**：动画会将 `transform` 的值从 `translateX(50px) rotate(45deg)` 替换为 `translateX(100px)`。
       - **`add`**：动画会将 `transform` 的值追加为 `translateX(50px) rotate(45deg) translateX(100px)`。
       - **`accumulate`**：动画会将 `transform` 的值累加为 `translateX(150px) rotate(45deg)`。

     * `iterationComposite`：确定此动画中值如何从一次迭代构建到下一次迭代。可以是设置为 `accumulate` 或 `replace` （见上文）。默认为 `replace` 

     * `pseudoElement`：伪元素选择器的 `string` ，例如 `"::before"` 。如果存在，则效果应用于 `target` 的所选伪元素，而不是应用于 `target` 本身。

### [KeyframeEffect](https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect)

> 用`KeyframeEffect` 接口创建可动画化的属性和值集合，称为关键帧。然后可以使用 `Animation()` 构造函数来播放这些关键帧。

语法格式：https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect/KeyframeEffect

* `new KeyframeEffect(target, keyframes)`
* `new KeyframeEffect(target, keyframes, options)`
* `new KeyframeEffect(sourceKeyFrames)`：`sourceKeyFrames`是一个您想要克隆的`KeyframeEffect`对象。

 构造函数返回一个新的 `KeyframeEffect` 对象实例，并且还允许您克隆现有的关键帧效果对象实例。

```js
const emoji = document.querySelector("div"); // element to animate

const rollingKeyframes = new KeyframeEffect(
  emoji,
  [
    { transform: "translateX(0) rotate(0)" }, // keyframe
    { transform: "translateX(200px) rotate(1.3turn)" }, // keyframe
  ],
  {
    // keyframe options
    duration: 2000,
    direction: "alternate",
    easing: "ease-in-out",
    iterations: "Infinity",
  },
);

const rollingAnimation = new Animation(rollingKeyframes, document.timeline);

// play rofl animation
rollingAnimation.play();
// rollingAnimation.pause();
// rollingAnimation.playbackRate = 2; // 加快播放速度
// rollingAnimation.currentTime = 1000; // 设置当前时间
// rollingAnimation.reverse(); // 反向播放
// rollingAnimation.cancel(); // 取消动画
```

**第二个参数为什么使用 `document.timeline`**

> document.timeline` 是 `DocumentTimeline` 的一个实例，表示文档的默认时间轴。它是每个文档独有的，并且在文档的整个生命周期内持续存在。默认情况下，如果没有显式指定时间轴，`Animation` 对象会自动使用 `document.timeline

- **默认时间轴**：`document.timeline` 是文档的默认时间轴，表示自文档加载以来的时间。使用它可以让动画与文档的加载时间同步，而不需要额外定义时间轴。
- **简单易用**：对于大多数动画场景，使用默认时间轴已经足够。它简化了代码，避免了创建自定义时间轴的复杂性。
- **性能优化**：默认时间轴是浏览器优化过的，使用它可以避免不必要的性能开销

## 其他应用

### 一、创建与滚动条进度绑定的动画[ScrollTimeline](https://developer.mozilla.org/en-US/docs/Web/API/ScrollTimeline)

动画的进度随着滚动条的移动而变化:

```js
const scrollTimeline = new ScrollTimeline({
  source: document.scrollingElement, // 指定滚动源，通常是文档的滚动元素
  axis: 'block',                
});
```

```js
const box = document.querySelector(".box");
const output = document.querySelector(".output");

const timeline = new ScrollTimeline({
  source: document.documentElement, // 指定滚动源，通常是文档的滚动元素
  axis: "block", // 滚动方向，可以是 'block'、'y' 或 'inline'、'x'
});

box.animate(
  {
    rotate: ["0deg", "720deg"],
    left: ["0%", "100%"],
  },
  {
    timeline,
  },
);
```

