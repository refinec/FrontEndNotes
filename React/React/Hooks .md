# Hooks

> `Hook`可以让你在不编写 `class` 的情况下使用 `state` 以及其他的 React **特性**。

- [基础 Hook](https://zh-hans.reactjs.org/docs/hooks-reference.html#basic-hooks)
  - [`useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate)
  - [`useEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect)
  - [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)
- [额外的 Hook](https://zh-hans.reactjs.org/docs/hooks-reference.html#additional-hooks)
  - [`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)
  - [`useCallback`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)
  - [`useMemo`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)
  - [`useRef`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useref)
  - [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)
  - [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect)
  - [`useDebugValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedebugvalue)
  - [`useDeferredValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedeferredvalue)
  - [`useTransition`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usetransition)
  - [`useId`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useid)
- [Library Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#library-hooks)
  - [`useSyncExternalStore`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usesyncexternalstore)
  - [`useInsertionEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useinsertioneffect)

## **使用Hook的原因是什么？**

1. class 是学习 React 的一大屏障，你必须去理解 JavaScript 中 `this` 的工作方式。还不能忘记绑定事件处理器

   而Hook 解决了这一点

2. 代码复用和代码管理

   通过使用 Hook，你可以把组件内相关的副作用组织在一起（例如创建订阅及取消订阅），而不要把它们拆分到不同的生命周期函数里。

## Hook使用规则

Hook 就是 JavaScript 函数，但是使用它们会有两个额外的规则：

- 只能在**函数最外层**调用 Hook。

  不要在**循环**、**条件判断**或者**子函数**中调用。

- 只能在 **React 的函数组件**和**自定义的 Hook** 中调用 Hook。

  不要在其他 JavaScript 函数中调用。

## 什么时候用Hook?

如果你在编写函数组件并意识到需要向其添加一些 `state`，以前的做法是必须将其转化为 `class`。

现在你可以在现有的**函数组件**中使用 `Hook`。

## 基础 Hook

### `useState `

> `useState` 会返回一对值：**当前**状态和一个让你更新它的函数
>
> 更新函数类似 **class** 组件的 `this.setState`，但是它不会把新的 `state` 和旧的 `state` 进行合并。

```jsx
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [count, setCount] = useState(0);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

在函数组件中，没有 `this`，所以我们不能分配或读取 `this.state`。在函数中，我们可以直接用 `count`:

```jsx
<p>You clicked {count} times</p>
```

### `useEffect`

> `useEffect` 就是一个 Effect Hook，给函数组件增加了操作副作用**(在 React 组件中执行过数据获取、订阅或者手动修改过 DOM)**的能力。
>
>  `useEffect` Hook 是 class 组件中 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合，因此具有更好的**代码复用**和**代码管理**

- 当你调用 `useEffect` 时，就是在告诉 React 在完成对 DOM 的更改后运行你的“副作用”函数。
- 副作用函数还可以通过返回一个函数来指定如何“**清除**”副作用。

下面的示例中，React 会在组件销毁时取消对 `ChatAPI` 的订阅。

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

## 自定义Hook
