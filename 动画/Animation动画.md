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