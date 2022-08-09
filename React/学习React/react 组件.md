## 定义组件

react 组件分为 **函数组件** 与 **class组件**。**<u>组件名称必须以大写字母开头，因为React 会将以小写字母开头的组件视为原生 DOM 标签。</u>**

### 一、函数组件

定义组件最简单的方式就是编写 JavaScript 函数：

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

该函数是一个有效的 React 组件，因为它接收唯一带有数据的 “**props**”（代表属性）对象与并返回一个 React 元素。这类组件被称为“函数组件”，因为它本质上就是 JavaScript 函数。

### 二、class 组件

还可以使用 [ES6 的 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) 来定义组件：

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
const element = <Welcome name="Sara" />;
```

> 当 React 元素为用户自定义组件时，它会将 JSX 所接收的**属性（attributes）**以及**子组件（children）**转换为单个对象传递给组件，这个对象被称之为 “**props**”。

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
const element = <Welcome name="Sara" />;
root.render(element);
```

## 组件的`Props`(只读性)

组件绝不能修改自身的 `props`，这通常会被叫做“**自上而下**”或是“**单向**”的数据流。

```jsx
function sum(a, b) {
  return a + b;
}
```

这样的函数被称为[“纯函数”](https://en.wikipedia.org/wiki/Pure_function)，因为该函数不会尝试更改入参，且多次调用下相同的入参始终返回相同的结果。

相反，下面这个函数则不是纯函数，因为它更改了自己的入参：

```jsx
function withdraw(account, amount) {
  account.total -= amount;
}
```

**所有 React 组件都必须像纯函数一样保护它们的 props 不被更改**

## 组件的`State`

* `state` 赋值

  构造函数是唯一可以给 `this.state` 赋值的地方

*  `state`修改

  使用 `setState()`

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Clock />);
```

### `State` 的更新可能是异步的

> 为什么`state`的更新可能是异步的？主要出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用

所以基于 `this.props` 和 `this.state` 可能会异步更新，那你就不要依赖他们的值来更新下一个状态。

例如，此代码可能会无法更新计数器：

```jsx
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

> 要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。
>
> **这个函数用上一个 `state` 作为第一个参数，将此次更新被应用时的 `props` 做为第二个参数**：

```jsx
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

#### **解决方法**

1. `setState()` 接收一个函数而不是一个对象

## 阻止组件渲染

在某些情况下，可能需要隐藏组件。若要完成此操作，你可以让 `render` 方法直接返回 `null`，而不进行任何渲染。

下面的示例中，`<WarningBanner />` 会根据 prop 中 `warn` 的值来进行条件渲染。如果 `warn` 的值是 `false`，那么组件则不会渲染:

```jsx
function WarningBanner(props) {
  if (!props.warn) {    return null;  }
  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render(<Page />);
```

在组件的 `render` 方法中返回 `null` 并不会影响组件的生命周期。例如，上面这个示例中，`componentDidUpdate` 依然会被调用。

## react 组件生命周期