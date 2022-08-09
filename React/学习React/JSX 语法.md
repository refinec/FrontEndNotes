## JSX 语法

> `const element = <h1>Hello, world!</h1>;`
>
> 上面这个JSX 标签语法既不是**字符串**也不是 **HTML**，是一个 **JavaScript** 的语法扩展。

Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。

```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
// 上面的代码完全等效于下面代码
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

`React.createElement()` 会预先执行一些检查，以帮助你编写无错代码，但实际上它创建了一个这样的对象：

```jsx
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

这些对象被称为 “React 元素”。它们描述了你希望在屏幕上看到的内容。React 通过读取这些对象，然后使用它们来构建 DOM 以及保持随时更新。

1. JSX中**嵌入表达式**

   ```jsx
   const name = 'Josh Perez';
   const element = <h1>Hello, {name}</h1>;
   ```

   你可以在大括号内放置任何有效的 [JavaScript 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)。例如，`2 + 2`，`user.firstName` 或 `formatName(user)` 都是有效的 JavaScript 表达式

2. JSX 中**指定属性**

   ```jsx
   const element = <a href="https://www.reactjs.org"> link </a>;
   const element = <img src={user.avatarUrl}></img>;
   ```

   > 因为 JSX 语法上更接近 JavaScript 而不是 HTML，所以 React DOM 使用 `camelCase`（小驼峰命名）来定义属性的名称，而不使用 HTML 属性名称的命名约定。

​			例如，JSX 里的 `class` 变为 [`className`](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)， `tabindex` 变为 [`tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)。

3. JSX **防止注入攻击**

   ```jsx
   const title = response.potentiallyMaliciousInput;
   // 直接使用是安全的：
   const element = <h1>{title}</h1>;
   ```

   你可以安全地在 JSX 当中**插入用户输入内容**，这是因为`React DOM` 在渲染所有输入内容之前，默认会进行[转义](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 [XSS（cross-site-scripting, 跨站脚本）](https://en.wikipedia.org/wiki/Cross-site_scripting)攻击。

   