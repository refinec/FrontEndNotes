### 创建拥有作用域的css文件

只要创建一个`xxx.module.css`的文件即可

```css
.p1 {
  color: red;
}
```

## 使用

```react
import classes from './xxx.module.css'

const App = () => {
  return (
  	<div className={classes.p1}></div>
  )
}
```

在html标签上`class`属性值的形式就为：`xxx_p1_9v2n6`, `9v2n6`是由react生成的唯一标识。