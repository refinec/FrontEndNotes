## 一、创建`store`

Redux `store` 是一个保存和管理应用程序状态的`state`， 可以使用 Redux 对象中的 `createStore()` 来创建一个 redux `store`， 此方法将 `reducer` 函数作为必需参数， 它只需将 `state` 作为参数并返回一个 `state` 即可。

```react
const reducer = (state = 5) => {
  return state;
}
let store = Redux.createStore(reducer);
```

[`createStore()`](https://www.redux.org.cn/docs/api/createStore.html) 的第二个参数是可选的, 用于设置 state 初始状态。这对开发同构应用时非常有用，服务器端 redux 应用的 state 结构可以与客户端保持一致, 那么客户端可以将从网络接收到的服务端 state 直接用于本地数据初始化。

```react
let store = createStore(reducer, window.STATE_FROM_SERVER)
```

## 二、访问数据

#### 1.获取状态 `store.getState()`

可以使用 `getState()` 方法检索 Redux store 对象中保存的当前 `state`。

```react
const store = Redux.createStore(
  (state = 5) => state
);

let currentState = store.getState();
```

#### 2.Store 监听器 `store.subscribe()`

在 Redux `store` 对象上访问数据的另一种方法是 `store.subscribe()`。 这允许将监听器函数订阅到 store，只要 `action` 被 `dispatch` 就会调用它们。 

这个方法的一个简单用途是为 store 订阅一个函数，它只是在每次收到一个 `action` 并且更新 store 时记录一条消息。

```react
// 1. 定义reducer函数
// 作用：根据不同的action对象，返回不同的新的state
// state: 管理的数据初始状态
// action：对象 type 标记当前想要做什么样的修改
const reducer = (state = 0, action) => {
  switch(action.type) {
    case 'ADD':
      return state + 1;
    default:
      return state;
  }
};

// 2. 使用reducer函数生成store实例
const store = Redux.createStore(reducer);

let count = 0;
const addOne = () => (count += 1);

// 3. 通过store实例的subscribe订阅数据变化
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(addOne);

// 4. 通过store实例的dispatch函数提交action更改状态
store.dispatch({type: 'ADD'});
console.log(count);

// 停止监听 state 更新
unsubscribe();
```

## 三、Action

> Action 本质上是 JavaScript 普通对象。我们约定，action 内必须使用一个字符串类型的 `type` 字段来表示将要执行的动作。
>
> 当应用规模越来越大时，建议使用单独的模块或文件来存放 action。

`actions.js`

```js
/*
 * action 类型
 */

export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * 其它的常量
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action 创建函数
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

### reducer

>  `reducer` 只是一个接受状态和动作，然后返回新状态的纯函数。它将 `state` 和 `action` 作为参数，并总是返回一个新的 `state`。

**永远不要**在 reducer 里做这些操作：

- 修改传入参数；
- 执行有副作用的操作，如 **API 请求**和**路由跳转**；
- 调用非纯函数，如 `Date.now()` 或 `Math.random()`。

**只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。**

```react
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if (index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

### `dispatch` 分发

> 调用 `store.dispatch()` 把 action 和特定数据一起发送。

```react
const ADD_NOTE = 'ADD_NOTE';

const notesReducer = (state = 'Initial State', action) => {
  switch(action.type) {
		case ADD_NOTE: 
      return action.text;
    default:
      return state;
  }
};

const addNoteText = (note) => {
  return {
    type: ADD_NOTE,
    text: note
  }
};

const store = Redux.createStore(notesReducer);

console.log(store.getState());
store.dispatch(addNoteText('Hello!'));
console.log(store.getState());
```

## 四、合并多个 Reducers `combineReducers()`

> 当应用程序的状态开始变得越来越复杂时，可能会将 state 分成多个块。定义多个 reducer 来处理应用程序状态的不同部分，然后将这些 reducer 组合成一个 root reducer。 然后将 root reducer 传递给 Redux `createStore()`方法。

为了将多个 reducer 组合在一起，Redux 提供了`combineReducers()`方法。 该方法接受一个对象作为参数，在该参数中定义一个属性，该属性将键与特定 reducer 函数关联。 Redux 将使用给定的键值作为关联状态的名称。

```react
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';

const counterReducer = (state = 0, action) => {
  switch(action.type) {
    case INCREMENT:
      return state + 1;
    case DECREMENT:
      return state - 1;
    default:
      return state;
  }
};

const LOGIN = 'LOGIN';
const LOGOUT = 'LOGOUT';

const authReducer = (state = {authenticated: false}, action) => {
  switch(action.type) {
    case LOGIN:
      return {
        authenticated: true
      }
    case LOGOUT:
      return {
        authenticated: false
      }
    default:
      return state;
  }
};

const rootReducer = Redux.combineReducers({
  auth: authReducer,
  count: counterReducer
});

const store = Redux.createStore(rootReducer);
```

## 五、redux中异步请求 `middleware`

> Redux 中间件又称为Redux Thunk 中间件

要使用 Redux Thunk 中间件，请将其作为参数传递给 `Redux.applyMiddleware()`。 然后将此函数作为第二个可选参数提供给 `createStore()` 函数， 看一下编辑器底部的代码。 然后，要创建一个异步的 action，需要在 action creator 中返回一个以 `dispatch` 为参数的函数。 在这个函数中，可以 dispatch action 并执行异步请求。

在此示例中，使用 `setTimeout()` 模拟异步请求。 通常在执行异步行为之前 dispatch action，以便应用程序状态知道正在请求某些数据（例如，这个状态可以显示加载图标）。 然后，一旦收到数据，就会发送另一个 action，该 action 的 data 是请求返回的数据同时也代表 API 操作完成。

请记住，正在将 `dispatch` 作为参数传递给这个特殊的 action creator。 需要 dispatch action 时只需将 action 直接传递给 dispatch，中间件就可以处理其余的部分。

```react
const REQUESTING_DATA = 'REQUESTING_DATA'
const RECEIVED_DATA = 'RECEIVED_DATA'

const requestingData = () => { return {type: REQUESTING_DATA} }
const receivedData = (data) => { return {type: RECEIVED_DATA, users: data.users} }

const handleAsync = () => {
  return function(dispatch) {
 

    dispatch(requestingData());

    setTimeout(function() {
      let data = {
        users: ["Jeff", "William", "Alice"]
      };
  

      dispatch(receivedData(data));
    }, 2500);
  }
};

const defaultState = {
  fetching: false,
  users: []
};

const asyncDataReducer = (state = defaultState, action) => {
  switch(action.type) {
    case REQUESTING_DATA:
      return {
        fetching: true,
        users: []
      }
    case RECEIVED_DATA:
      return {
        fetching: false,
        users: action.users
      }
    default:
      return state;
  }
};

const store = Redux.createStore(
  asyncDataReducer,
  Redux.applyMiddleware(ReduxThunk.default)
);
```

