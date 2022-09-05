# Hooks

> `Hook`可以让你在不编写 `class` 的情况下使用 `state` 以及其他的 React **特性**，所以Hook 在 class 内部是**不**起作用的

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

> `useState` 会返回一对值：**当前**状态和一个让你更新它的函数（更新函数类似 **class** 组件的 `this.setState`，但是它不会把新的 `state` 和旧的 `state` 进行合并）

```jsx
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [count, setCount] = useState(0);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

**在函数组件中，没有 `this`**，所以我们不能分配或读取 `this.state`。在函数中，我们可以直接用 `count`:

```jsx
<p>You clicked {count} times</p>
```

### `useEffect`

> `useEffect` 就是一个 Effect Hook，给函数组件增加了操作<u>副作用</u>**(在 React 组件中执行过数据获取、订阅或者手动修改过 DOM)**的能力。
>
>  `useEffect` Hook 是 class 组件中 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合，因此具有更好的**代码复用**和**代码管理**

- 当你调用 `useEffect` 时，就是在告诉 React 在完成对 DOM 的更改后运行你的“副作用”函数。副作用函数通过返回一个函数来指定如何“**清除**”副作用。

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
    return () => { // 可选的清除函数
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

#### 为什么使用`useEffect`?

1. **关注点分离**

   为了解决在使用`class` 创建组件的时候，class 中生命周期函数经常包含不相关的逻辑，但又把相关逻辑分离到了几个不同方法中的问题。

   而使用多个`useEffect`实现关注点分离，把相关逻辑代码组合在一起，方便查找，类比于Vue3的`setup`

#### Effect的清除副作用执行时机？

> effect 的清除阶段在每次重新渲染 (`componentDidUpdate`) 时都会执行，而不是仅仅只在卸载组件(`componentWillUnmount`)的时候执行一次

因为 `useEffect`的 默认处理是：它会在调用一个新的 effect 之前对前一个 effect 进行清理。

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // 运行第一个 effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // 运行下一个 effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // 运行下一个 effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // 清除最后一个 effect
```

此默认行为保证了一致性，避免了在 class 组件中因为没有处理更新逻辑而导致常见的 bug。

#### 通过跳过 Effect 进行性能优化 (使用`useEffect`的第二参数)

在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：

```react
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

换算成 Effect

```react
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新。
						 // 如果 count 的值是 5，而且我们的组件重渲染的时候 count 还是等于 5，React 将对前一次渲染的 [5] 和后一次渲染的 [5] 进行比较。因为数组中的所有元素都是相等的(5 === 5)，React 会跳过这个 effect，这就实现了性能的优化。

useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // 仅在 props.friend.id 发生变化时，重新订阅
```

**注意📢：**

1. 要使用此优化方式，请确保数组中包含了**所有外部作用域中会随时间变化并且在 effect 中使用的变量**，否则你的代码会引用到先前渲染中的旧变量。

2. 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，effect 内部的 props 和 state 就会一直拥有其初始值，所以它永远都不需要重复执行，传入 `[]` 作为第二个参数更接近大家更熟悉的 `componentDidMount` 和 `componentWillUnmount` 思维模式。

### `useContext`
