## 一、创建`store`

Redux `store` 是一个保存和管理应用程序状态的`state`， 可以使用 Redux 对象中的 `createStore()` 来创建一个 redux `store`， 此方法将 `reducer` 函数作为必需参数， 它只需将 `state` 作为参数并返回一个 `state` 即可。

```react
const reducer = (state = 5) => {
  return state;
}
let store = Redux.createStore(reducer);
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
const reducer = (state = 0, action) => {
  switch(action.type) {
    case 'ADD':
      return state + 1;
    default:
      return state;
  }
};
const store = Redux.createStore(reducer);

let count = 0;
const addOne = () => (count += 1);
store.subscribe(addOne);

store.dispatch({type: 'ADD'});
console.log(count);
store.dispatch({type: 'ADD'});
console.log(count);
```

## 三、Action(Reducer)

`reducer` 将 `state` 和 `action` 作为参数，并且它总是返回一个新的 `state`。 要知道这是 reducer 的**唯一**的作用。 

它不应有任何其他的作用：比如它不应调用 API 接口，也不应存在任何潜在的副作用。 reducer 只是一个接受状态和动作，然后返回新状态的纯函数。

Redux 的另一个关键原则是 `state` 是只读的。 换句话说，`reducer` 函数必须**始终**返回一个新的 `state`，并且永远不会直接修改状态。

```react
const defaultState = {
  login: false
};

const reducer = (state = defaultState, action) => {
  if (action.type === 'LOGIN') return ({
    login: true
  });
  else return state;
};

const store = Redux.createStore(reducer);

const loginAction = () => {
  return {
    type: 'LOGIN'
  }
};
```

### `dispatch` 分发

调用 `store.dispatch()` 将从 `action` 发送给 store。

```react
const LOGIN = "LOGIN";
const LOGOUT = "LOGOUT";

const defaultState = {
  authenticated: false
};

const authReducer = (state = defaultState, action) => {
  switch (action.type) {
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

const store = Redux.createStore(authReducer);

const loginUser = () => {
  return {
    type: 'LOGIN'
  }
};

const logoutUser = () => {
  return {
    type: 'LOGOUT'
  }
};
```

```react
store.dispatch(loginUser()); // store.dispatch({ type: 'LOGIN' });
```

### `dispatch` 分发并包含数据

> 把 action 和特定数据一起发送。

```react
const ADD_NOTE = 'ADD_NOTE';

const notesReducer = (state = 'Initial State', action) => {
  switch(action.type) {
  
case ADD_NOTE: return action.text;
    
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

