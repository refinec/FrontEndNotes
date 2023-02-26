`React.StrictMode`组件会开启React对自身的一些检查，检查你的组件中是否写有副作用的代码，React的严格模式，在处于严格模式下时，会主动重复的调用一些函数，以使副作用呈现，所以以下函数在严格的开发模式下会触发两次：

* 类组件的`constructor`、`render`、`shouldComponentUpdate`方法
* 类组件的静态方法`getDerivedStateFromProps`
* 函数组件的函数体
* 参数为函数的`setState`
* 参数为函数的`useState`、`useMemo`、`useReducer`

重复的调用会使副作用凸显出来，你可以尝试在函数组件的函数体中调用`console.log`，你会发现它会执行两次。

