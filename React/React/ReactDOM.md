# ReactDOM

> `react-dom`是`react`的`DOM`包，使用react开发 WEB应用时必须引入，但是开发`React Native` 就不需要引入。

**引入：**`import * as ReactDOM from 'react-dom';`



`react-dom` 包导出了如下这些方法：

`createPortal()`

`flushSync()`

## `createPortal(child, container)`

> 创建 portal。[Portal](https://zh-hans.reactjs.org/docs/portals.html) 提供了一种将子节点渲染到已 DOM 节点中的方式，该节点存在于 DOM 组件的层次结构之外。

实际应用是通常把`Modal`弹窗渲染到与根元素同级的兄弟元素上。

```jsx
import ReactDOM from "react-dom";

// 获得根元素的兄弟元素
const rootSibling = document.getElementById("rootSibling");

const Modal = (props) => {
    return ReactDOM.createPortal(
        <div>
            {props.children}
        </div>,
        rootSibling
    )
}

export default Modal;
```

## `flushSync(callback)`

> 强制 React 同步刷新提供的回调函数中的任何更新。这确保了 DOM 会被立即 更新。

```react
flushSync(() => {
  setCount(count + 1);
});
// 此时，DOM 已更新。
```

**注意📢：**

- `flushSync` 会对性能产生很大影响。尽量少用。

- `flushSync` 可能会迫使悬而未决的 `Suspense` 边界显示其 `fallback` 的状态。

- `flushSync` 也可以运行待定副作用，并在返回之前同步应用它们所包含的任何更新。

  当需要刷新内部的更新时，`flushSync` 也可以在回调外部刷新更新。例如，如果有来自点击的未决更新。React 可能会在刷新回调之前刷新这些更新。



# ReactDOMClient

**引入：**`import * as ReactDOM from 'react-dom/client';`



# ReactDOMServer

**引入：**`import * as ReactDOMServer from 'react-dom/server';`





# DOM复合属性列表

> 在 React 中，所有的 DOM 特性和属性（包括事件处理）都应该是小驼峰命名的方式。例如，与 HTML 中的 `tabindex` 属性对应的 React 的属性是 `tabIndex`。

## 1.`className`

> `className` 属性用于指定 CSS 的 class，此特性适用于所有常规 DOM 节点和 SVG 元素

如果你在 React 中使用 Web Components（这是一种不常见的使用方式），请使用 `class` 属性代替

## 2.`dangerouslySetInnerHTML`

> `dangerouslySetInnerHTML` 是 React 为浏览器 DOM 提供 `innerHTML` 的替换方案。

通常来讲，使用代码直接设置 HTML 存在风险，因为很容易无意中使用户暴露于[跨站脚本（XSS）](https://en.wikipedia.org/wiki/Cross-site_scripting)的攻击。因此，你可以直接在 React 中设置 HTML，但当你想设置 `dangerouslySetInnerHTML` 时，需要向其传递包含 key 为 `__html` 的对象，以此来警示你。例如：

```jsx
function createMarkup() {
  return {__html: 'First &middot; Second'};
}

function MyComponent() {
  return <div dangerouslySetInnerHTML={createMarkup()} />;
}
```

## 3.`htmlFor`

> 由于 `for` 在 JavaScript 中是保留字，所以 React 元素中使用了 `htmlFor` 来代替。

## 4. `onChange`

> `onChange` 事件与预期行为一致：每当表单字段变化时，该事件都会被触发。

## 5. `selected`

> 如果要将 `<option>` 标记为已选中状态，请在 `select` 的 `value` 中引用该选项的值

## 6.`style`

> `style` 接受一个采用小驼峰命名属性的 JavaScript 对象

**注意**：样式不会自动补齐前缀。如需支持旧版浏览器，请手动补充对应的样式属性：

```jsx
const divStyle = {
  WebkitTransition: 'all', // note the capital 'W' here
  msTransition: 'all' // 'ms' is the only lowercase vendor prefix
};

function ComponentWithTransition() {
  return <div style={divStyle}>This should work cross-browser</div>;
}
```

Style 中的 key 采用小驼峰命名是为了与 JS 访问 DOM 节点的属性保持一致（例如：`node.style.backgroundImage` ）。浏览器引擎前缀都应以大写字母开头，[除了 `ms`](https://www.andismith.com/blog/2012/02/modernizr-prefixed/)。因此，`WebkitTransition` 首字母为 ”**W**”。

React 会自动添加 ”px” 后缀到内联样式为数字的属性后。如需使用 ”px” 以外的单位，请将此值设为数字与所需单位组成的字符串。例如：

```jsx
// Result style: '10px'
<div style={{ height: 10 }}>
  Hello World!
</div>

// Result style: '10%'
<div style={{ height: '10%' }}>
  Hello World!
</div>
```

## 7.`suppressContentEditableWarning`

通常，当拥有子节点的元素被标记为 `contentEditable` 时，React 会发出一个警告，因为这不会生效。

**该属性将禁止此警告**。

尽量不要使用该属性，除非你要构建一个类似 [Draft.js](https://facebook.github.io/draft-js/) 的手动管理 `contentEditable` 属性的库。

## 8.`suppressHydrationWarning`

如果你使用 React 服务端渲染，通常会在当服务端与客户端渲染不同的内容时发出警告。但是，在一些极少数的情况下，很难甚至于不可能保证内容的一致性。例如，在服务端和客户端上，时间戳通常是不同的。

**设置 `suppressHydrationWarning` 为 `true`，React 将不会警告你属性与元素内容不一致。**

它只会对元素一级深度有效，并且打算作为应急方案。因此不要过度使用它。你可以在 [`ReactDOM.hydrateRoot()` 文档](https://zh-hans.reactjs.org/docs/react-dom-client.html#hydrateroot) 中了解更多关于 hydration 的信息。

## 自定义属性

你也可以使用自定义属性，但要注意属性名全都为小写。

```jsx
<div mycustomattribute="something" />
```