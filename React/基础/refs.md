## 字符串形式的ref(不推荐，有效率问题)

```jsx
class Demo extends React.component{
    showData = () => {
        const { input1 } = this.refs;
        alert(input1.value)
    }
    render() {
        return (
            <div>
                <input ref="input1" type="text"></input>
                <button onClick={this.showData}></button>
            </div>
        )
    }
}
ReactDOM.render(<Demo/>,document.getElementById('app'))
```

## 回调ref

> 如果 `ref` 回调函数是以内联函数的方式定义的，在**更新过程**中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。
>
> 这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，**但是大多数情况下它是无关紧要的**。

```jsx
# 内联形式
class Demo extends React.component{
    showData = () => {
        const { input1 } = this;
        alert(input1.value)
    }
    render() {
        // currentNode参数是当前结点
        return (
            <div>
                <input ref={currentNode => this.input1=currentNode} type="text"></input>
                <button onClick={this.showData}></button>
            </div>
        )
    }
}
ReactDOM.render(<Demo/>,document.getElementById('app'))

# class的绑定形式
class Demo extends React.component{
    showData = () => {
        const { input1 } = this;
        alert(input1.value)
    }
    saveInput = (c)=>{
        this.input1 = c;
    }
    render() {
        // currentNode参数是当前结点
        return (
            <div>
                {/*<input ref={currentNode => this.input1=currentNode} type="text"></input>*/}
                <input ref={this.saveInput} type="text"></input>
                <button onClick={this.showData}></button>
            </div>
        )
    }
}
ReactDOM.render(<Demo/>,document.getElementById('app'))
```

## createRef

```jsx
class Demo extends React.component{
    /**
     * React.createRef调用后可以返回一个容器，该容器可以存储被ref所标识的节点
     * 但是该容器只能存一个，后面存入的会顶掉前面的
     */
    myRef = React.createRef();
    myRef2 = React.createRef()
    showData = () => {
        alert(this.myRef.current.value)
    }
    render() {
        return (
            <div>
                <input ref={this.myRef} type="text"></input>
                <button onClick={this.showData}></button>
            </div>
        )
    }
}
ReactDOM.render(<Demo/>,document.getElementById('app'))
```

