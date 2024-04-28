## Hooks

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

### **为什么要用Hooks？**

1. class 是学习 React 的一大屏障，你必须去理解 JavaScript 中 `this` 的工作方式。还不能忘记绑定事件处理器，而Hook 解决了这一点

2. 代码复用和代码管理

   通过使用 Hook，你可以把组件内相关的副作用组织在一起（例如创建订阅及取消订阅），而不要把它们拆分到不同的生命周期函数里。

### Hooks使用规则

只能在 **React 的函数组件**和**自定义的 Hooks** 中调用 Hook。不要在**循环**、**条件判断**或者**子函数**中调用。

> 安装名为 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) 的 ESLint 插件来强制执行这两条规则（注意：**自定义 Hook 必须以 “`use`” 开头**，这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了Hook的规则）
>
> `npm install eslint-plugin-react-hooks --save-dev`
>
> ```json
> // 你的 ESLint 配置
> {
>   "plugins": [
>     // ...
>     "react-hooks"
>   ],
>   "rules": {
>     // ...
>     "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
>     "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
>   }
> }
> ```

### 什么时候用Hook?

如果你在编写函数组件并意识到需要向其添加一些 `state`，以前的做法是必须将其转化为 `class`。

现在你可以在现有的**函数组件**中使用 `Hook`。

### 一、基础 Hook

#### `useState `

> `useState` 会返回一对值：**当前**状态和一个让你更新它的函数（更新函数类似 **class** 组件的 `this.setState`，但是它不会把新的 `state` 和旧的 `state` 进行合并）

```jsx
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [count, setCount] = useState(0);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
  setCount(1) // 函数异步执行
	// 要想正确更新数据，则使用函数
  setCount((preCount) => {
    return preCount + 1
  })
}
```

**在函数组件中，没有 `this`**，所以我们不能分配或读取 `this.state`。在函数中，我们可以直接用 `count`:

```jsx
<p>You clicked {count} times</p>
```

##### setState的执行流程

setState的执行流程(函数组件)：

1. 先判断组件当前处于什么状态
   * 渲染阶段：不会检查state值是否相同
   * 非渲染阶段(渲染结束)：会检查state值是否相同
2. 在非渲染阶段
   * 如果值不相同，则对组件进行重渲染
   * 如果值相同，则不对组件进行重渲染
     * 但是在某些情况下，如果值相同，React会继续执行当前组件的重渲染，但这个渲染不会触发其子组件的重渲染，这次渲染不会产生实际的效果

#### `useEffect`

> `useEffect` 就是一个 Effect Hook，给函数组件增加了操作<u>副作用</u>**(在 React 组件中执行过数据获取、订阅或者手动修改过 DOM)**的能力。
>
> 专门用来处理那些不能直接写在组件内部的代码，比如：**ajax请求数据、记录日志、检查登录、设置定时器等**，总的来说，就是那些和组件渲染无关的，但可能对组件产生副作用的代码。

> `useEffect` Hook 是 class 组件中 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合，因此具有更好的**代码复用**和**代码管理**

 `useEffect` 中的回调函数，在每次组件渲染完毕后执行。回调函数通过返回一个函数来指定如何“**清除**”副作用。

**格式**：`useEffect(() => { /* 编写那些会产生副作用的代码 */ }, deps)`

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

##### 为什么使用`useEffect`?

1. **关注点分离**

   为了解决在使用`class` 创建组件的时候，class 中生命周期函数经常包含不相关的逻辑，但又把相关逻辑分离到了几个不同方法中的问题。

   而使用多个`useEffect`实现关注点分离，把相关逻辑代码组合在一起，方便查找，类比于Vue3的`setup`

##### Effect的清除副作用执行时机？

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

##### 通过跳过 Effect 进行性能优化 (使用`useEffect`的第二参数，依赖项)

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

1. 要使用此优化方式，请确保数组中包含了**<u>所有</u>外部作用域中会随时间变化并且在 effect 中使用的变量**，否则你的代码会引用到先前渲染中的旧变量。

2. 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，effect 内部的 props 和 state 就会一直拥有其初始值，所以它永远都不需要重复执行，传入 `[]` 作为第二个参数更接近大家更熟悉的 `componentDidMount` 和 `componentWillUnmount` 思维模式。

#### `useContext`

> `const contextValue = useContext(MyContext);` 
>
> 接收一个 context 对象（`React.createContext` 的返回值）并返回该 context 的当前值。
>
>  context 的当前值由上层组件中距离当前组件最近的 `<MyContext.Provider>` 的 `value` prop 决定。

 `useContext`需要配合**Context.Provider**使用，调用了 `useContext` 的组件总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，你可以 [通过使用 memoization 来优化](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。



```react
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

### 二、额外的 Hook

#### `useReducer` : `useState`的替代方案

> 即把对state的相关操作，全部放在一个作用域内

```react
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

在某些场景下，`useReducer` 会比 `useState` 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 `useReducer` 还能给那些会触发深更新的组件做性能优化，因为[你可以向子组件传递 `dispatch` 而不是回调函数](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) 。

```react
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

##### 第三参数：惰性初始化

你可以选择惰性地创建初始 state。为此，需要将 `init` 函数作为 `useReducer` 的第三个参数传入，这样初始 state 将被设置为 `init(initialArg)`。

这么做可以将用于计算 state 的逻辑提取到 reducer 外部，这也为将来对重置 state 的 action 做处理提供了便利：

```react
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

##### 跳过 dispatch

> 如果 Reducer Hook 的返回值与当前 state 相同（使用 [`Object.is` 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 来比较），React 将跳过子组件的渲染及副作用的执行

#### `useCallback`

> 为什么使用`useCallback`？当`state`发生变化时，组件会重新渲染，并且组件内的函数、方法也会重新创建，这是没有必要的，所以需要`useCallback`进行优化，保持引用稳定。

**格式：**`useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`

> 如果不提供参数`deps`,那么使用`useCallback`毫无意义，当`state`发生变化时，组件会重新渲染，回调函数`fn`也会重新创建。所以使用`useCallback`提供`deps`是必要的，所有回调函数`fn`中引用的值都应该出现在依赖项数组`deps`中。
>
> 另外，如果提供的`deps`为`[]`,那么回调函数`fn`只会在组件初始化的时候创建，内部依赖将始终为初始值。

**示例：**

把内联**回调函数**及**依赖项数组**作为参数传入 `useCallback`，返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 回调函数。该内联回调函数仅在某个依赖项改变时才会更新

```react
const App = () => {
  const memoizedCallback = useCallback(
    () => {
      doSomething(a, b);
    },
    [a, b], // 所有回调函数中引用的值都应该出现在依赖项数组中。
  );
  return (
  	<button onClick={memoizedCallback}></button>
  )
}
export default App;
```

推荐启用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 中的 [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) 规则。此规则会在添加错误依赖时发出警告并给出修复建议

#### `useMemo`  类似vue computed

> 如果你在渲染期间执行了高开销的计算，则可以使用 `useMemo` 来进行优化。

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 值。把**“创建”函数**和**依赖项数组**作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。如果没有提供**依赖项数组**，`useMemo` 在每次渲染时都会计算新的值。

#### `useRef` 获取/操作DOM

> `const refContainer = useRef(initialValue);`  创建一个存储DOM对象的容器

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内持续存在。

```react
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

本质上，`useRef` 就像是可以在其 `.current` 属性中保存一个可变值的“盒子”。而`React.createRef()` 创建的ref，无论该节点如何改变，React 都会将 ref 对象的 `.current` 属性设置为相应的 DOM 节点。

`useRef()` 比 `ref` 属性更有用，它可以[很方便地保存任何可变值](https://zh-hans.reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables)：

```react
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {
      // ...
    });
    intervalRef.current = id;
    return () => {
      clearInterval(intervalRef.current);
    };
  });

  // ...
}
```

注意📢：

当 ref 对象内容发生变化时，`useRef` 并*不会*通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用[回调 ref](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node) 来实现：

```react
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

#### `useImperativeHandle`

> 格式：`useImperativeHandle(ref, createHandle, [deps])`
>
> `useImperativeHandle` 可以让你在使用 `ref` 时暴露子组件中的方法给父组件。
>
> `useImperativeHandle` 应当与 [`forwardRef`](https://zh-hans.reactjs.org/docs/react-api.html#reactforwardref) 一起使用：

渲染 `<FancyInput ref={inputRef} />` 的父组件可以调用 `inputRef.current.focus()`。

```react
import { forwardRef } from "react";

const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef();
  // 实现聚焦逻辑函数
  const focusHandler = () => {
    inputRef.current.focus();
  }
  // 暴露函数给父组件调用
  useImperativeHandle(ref, () => {
    return {
      focusHandler
    }
  });
  return <input ref={inputRef} ... />;
});
```

#### `useLayoutEffect`

> 尽可能使用 `useEffect` 以避免阻塞视觉更新

其函数签名与 `useEffect` 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，`useLayoutEffect` 内部的更新计划将被同步刷新。

注意📢：

* `useLayoutEffect` 与 `componentDidMount`、`componentDidUpdate` 的调用阶段是一样的。但是，我们推荐你**一开始先用 `useEffect`**，只有当它出问题的时候再尝试使用 `useLayoutEffect`
* 如果你使用服务端渲染，请记住，*无论* `useLayoutEffect` *还是* `useEffect` 都无法在 Javascript 代码加载完成之前执行。这就是为什么在服务端渲染组件中引入 `useLayoutEffect` 代码时会触发 React 告警。解决这个问题，需要将代码逻辑移至 `useEffect` 中（如果首次渲染不需要这段逻辑的情况下），或是将该组件延迟到客户端渲染完成后再显示（如果直到 `useLayoutEffect` 执行之前 HTML 都显示错乱的情况下）
* 若要从服务端渲染的 HTML 中排除依赖布局 effect 的组件，可以通过使用 `showChild && <Child />` 进行条件渲染，并使用 `useEffect(() => { setShowChild(true); }, [])` 延迟展示组件。这样，在客户端渲染完成之前，UI 就不会像之前那样显示错乱了

#### `useDebugValue`

`useDebugValue(value)` : `useDebugValue` 可用于在 React 开发者工具中显示自定义 hook 的标签。

```react
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // 在开发者工具中的这个 Hook 旁边显示标签
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}

```

`useDebugValue` 接受一个格式化函数作为可选的第二个参数。该函数只有在 Hook 被检查时才会被调用。它接受 debug 值作为参数，并且会返回一个格式化的显示值。

例如，一个返回 `Date` 值的自定义 Hook 可以通过格式化函数来避免不必要的 `toDateString` 函数调用：

`useDebugValue(date, date => date.toDateString());`

#### `useDeferredValue`

格式：`const deferredValue = useDeferredValue(value);`

`useDeferredValue` 接受一个值，并返回该值的新副本，该副本将推迟到更紧急地更新之后。如果当前渲染是一个紧急更新的结果，比如用户输入，React 将返回之前的值，然后在紧急渲染完成后渲染新的值。

该 hook 与使用防抖和节流去延迟更新的用户空间 hooks 类似。使用 `useDeferredValue` 的好处是，React 将在其他工作完成（而不是等待任意时间）后立即进行更新，并且像 [`startTransition`](https://zh-hans.reactjs.org/docs/react-api.html#starttransition) 一样，延迟值可以暂停，而不会触发现有内容的意外降级。

`useDeferredValue` 仅延迟你传递给它的值。如果你想要在紧急更新期间防止子组件重新渲染，则还必须使用 React.memo 或 React.useMemo 记忆该子组件：

```react
function Typeahead() {
  const query = useSearchQuery('');
  const deferredQuery = useDeferredValue(query);

  // Memoizing 告诉 React 仅当 deferredQuery 改变，
  // 而不是 query 改变的时候才重新渲染
  const suggestions = useMemo(() =>
    <SearchSuggestions query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <>
      <SearchInput query={query} />
      <Suspense fallback="Loading results...">
        {suggestions}
      </Suspense>
    </>
  );
}
```

记忆该子组件告诉 React 它仅当 `deferredQuery` 改变而不是 `query` 改变的时候才需要去重新渲染。这个限制不是 `useDeferredValue` 独有的，它和使用防抖或节流的 hooks 使用的相同模式。

#### `useTransition`

> `const [isPending, startTransition] = useTransition();` 返回一个状态值表示过渡任务的等待状态，以及一个启动该过渡任务的函数

`startTransition` 允许你通过标记更新将提供的回调函数作为一个过渡任务：

```react
startTransition(() => {
  setCount(count + 1);
})
```

`isPending` 指示过渡任务何时活跃以显示一个等待状态：

```react
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);
  
  function handleClick() {
    startTransition(() => {
      setCount(c => c + 1);
    })
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```

注意📢：

* 过渡任务中触发的更新会让更紧急地更新先进行，比如点击。

* 过渡任务中的更新将不会展示由于再次挂起而导致降级的内容。这个机制允许用户在 React 渲染更新的时候继续与当前内容进行交互。

#### `useId`

> `const id = useId();`

`useId` 是一个用于生成横跨服务端和客户端的稳定的唯一 ID 的同时避免 hydration 不匹配的 hook。但`useId` 不用于生成列表中的键。键应该从数据中生成。

```react
function NameFields() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id + '-firstName'}>First Name</label>
      <div>
        <input id={id + '-firstName'} type="text" />
      </div>
      <label htmlFor={id + '-lastName'}>Last Name</label>
      <div>
        <input id={id + '-lastName'} type="text" />
      </div>
    </div>
  );
}
```

> 注意：
>
> `useId` 生成一个包含 `:` 的字符串 token。这有助于确保 token 是唯一的，但在` CSS 选择器`或 `querySelectorAll` 等 API 中不受支持。
>
> `useId` 支持 `identifierPrefix` 以防止在多个根应用的程序中发生冲突。 要进行配置，请参阅 [`hydrateRoot`](https://zh-hans.reactjs.org/docs/react-dom-client.html#hydrateroot) 和 [`ReactDOMServer`](https://zh-hans.reactjs.org/docs/react-dom-server.html) 的选项。

### 三、library-hooks

> 以下 hook 是为库作者提供的，用于将库深入集成到 React 模型中，通常不会在应用程序代码中使用。

#### `usesyncexternalstore`



#### `useinsertioneffect`



## 如何自定义Hooks？

> 自定义Hooks就是一个普通函数，该函数名称以`use`开头，本质是一个调用其他钩子函数的钩子函数

在React中创建自定义Hooks时，应当遵循一些最佳实践和注意事项，以确保它们的正确性和可重用性。以下是关于自定义Hooks需要注意的一些要点：

1. **命名约定**

   自定义Hooks应该以`use`开头，这不仅是一个约定，也让React能够识别你的Hook是否遵守Hooks的规则。

2. **调用顺序**

   自定义Hooks内的所有原始Hooks（如useState, useEffect等）调用必须始终保持相同的顺序。这是因为React依赖于Hook调用的顺序来正确地管理状态。

3. **不要在`循环`、`条件`或`嵌套函数`中调用Hooks**

   正如React的规则所指，Hooks应该总是在组件的顶层调用，不要在循环、条件或嵌套函数中调用它们。

4. **保持简洁**

   自定义Hooks应该保持简洁和专注。它们旨在封装一些可以在多个组件间共享的`逻辑`。

5. **返回值**

   考虑到以后的适应性，最好是返回一个对象，这样添加新的返回值时不需要重新排列解构语句。

   例如，使用`const { data, error } = useCustomHook()`而不是`const [data, error] = useCustomHook()`。

6. **React依赖项传递**

   如果你的Hook内部使用了像`useEffect`、`useCallback`、`useMemo`这样的React Hooks，确保正确处理依赖项列表。

7. **逻辑复用而非状态复用**

   自定义Hooks通常用来共享逻辑而非状态。如果你发现自己在复用太多的状态相关代码，那可能组件抽象才是更好的方式。

8. **封装上下文**

   如果你在多个组件中使用相同的上下文，你可以将`useContext(MyContext)`封装在自定义Hook中，这样能够简化组件中的代码。

```react
export default function useToggle() {
  // 可复用的逻辑代码
  const [value, setValue] = useState(true);
  const toggle = () => setValue(!value);
  
  // 哪些状态和回调函数需要在 其他组件或自定义hook 中使用的就return出去
  return {
    value,
    toggle
  }
}
```

## 为什么不应在循环、条件或嵌套函数中调用Hooks？

在React中，不应在循环、条件判断或嵌套函数中调用Hooks的原因主要归结于两个关键点：**Hooks的调用顺序**和**React的Hooks规则**。

1. **Hooks的调用顺序**：
   React依赖于Hooks的调用顺序来正确地追踪组件的状态和副作用。每次组件渲染时，Hooks都必须以相同的顺序被调用。这是因为React内部没有将Hook与其名字关联，而是依赖它们调用的顺序来确定哪个状态对应于哪个`useState`或`useEffect`。

   如果将Hooks放在循环、条件判断或嵌套函数内部，那么它们的调用顺序就可能在组件的不同渲染周期内发生变化。这将导致React无法正确追踪Hook的状态，从而出现不可预知的错误。

2. **React的Hooks规则**：
   因为上述对调用顺序的依赖，React提出了一个规则，即“`只在React函数组件的最顶层调用Hooks`”。这意味着在循环、条件判断或嵌套函数中调用Hooks是违反这一规则的。遵守这个规则能够确保Hooks的调用顺序在组件的每次渲染中保持一致。

为了避免这些问题，可以采用以下策略：

- 如果逻辑需要根据条件判断执行，可以将条件判断放在Hook内部。例如，在`useEffect`内部检查某个条件是否成立，而不是条件控制`useEffect`的调用。
- 对于循环生成的Hooks，可以考虑重新设计你的组件结构，使得每个循环项成为一个新的组件，在这些新组件中独立地调用Hooks。
- 如果需要根据条件选择性地创建状态或执行副作用，可以考虑使用状态或副作用内部的逻辑判断，或者使用不同组件来封装这些逻辑并根据条件渲染它们。
