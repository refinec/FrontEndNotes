## Refs & DOM

```jsx
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创造一个 textInput DOM 元素的 ref
    this.textInput = React.createRef();
  }
  focus() {
    // 使用原始的 DOM API 显式地聚焦在 text input 上
    // 注意：我们通过访问 “current” 来获得 DOM 节点
    this.textInput.current.focus();
  }
  render() {
    // 使用 `ref` 回调函数以在实例的一个变量中存储文本输入 DOM 元素
    //（比如，this.textInput）。
    return (
      <input
        type="text"
        ref={this.textInput}
        />
    );
  }
}
```

