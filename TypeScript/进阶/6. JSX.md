# JSX

> [JSX](https://facebook.github.io/jsx/)是一种嵌入式的类似XML的语法， 它可以被转换成合法的JavaScript。

## 基本用法

使用JSX必须做两件事：

1. 给文件一个`.tsx`扩展名
2. 启用`jsx`选项

TypeScript具有**三种JSX模式**：`preserve`，`react`和`react-native`。

> 可以通过在命令行里使用`--jsx`标记或[tsconfig.json](https://www.tslang.cn/docs/handbook/tsconfig-json.html)里的选项来指定模式
>
> 注意：`React`标识符是写死的硬编码，React（大写的R）是可用的。

- `preserve`模式下生成代码中会保留JSX以供后续的转换操作使用（比如：[Babel](https://babeljs.io/)），并且输出文件会带有`.jsx`扩展名。 
- `react`模式会生成`React.createElement`，在使用前不需要再进行转换操作了，输出文件的扩展名为`.js`。 
- `react-native`相当于`preserve`，它也保留了所有的JSX，但是输出文件的扩展名是`.js`。

| 模式           | 输入      | 输出                         | 输出文件扩展名 |
| :------------- | :-------- | :--------------------------- | :------------- |
| `preserve`     | `<div />` | `<div />`                    | `.jsx`         |
| `react`        | `<div />` | `React.createElement("div")` | `.js`          |
| `react-native` | `<div />` | `<div />`                    | `.js`          |

## 类型检查

假设有这样一个JSX表达式`<expr />`，`expr`可能引用环境自带的某些东西（比如，在DOM环境里的`div`或`span`）或者是你自定义的组件。 这是非常重要的，原因有如下两点：

1. 对于React，**固有元素**会生成字符串（`React.createElement("div")`），然而由你自定义的组件却不会生成（`React.createElement(MyComponent)`）。
2. 传入JSX元素里的属性类型的查找方式不同。 **固有元素属性***本身*就支持，然而自定义的组件会自己去指定它们具有哪个属性。

**固有元素总是以一个小写字母开头，基于值的元素总是以一个大写字母开头**。

### 固有元素

固有元素使用特殊的接口`JSX.IntrinsicElements`来查找。

默认地，如果这个接口没有指定，会全部通过，不对固有元素进行类型检查。

如果这个接口存在，那么固有元素的名字需要在`JSX.IntrinsicElements`接口的属性里查找。 例如：

```tsx
declare namespace JSX {
    interface IntrinsicElements {
        foo: any,
        // [elemName: string]: any;// 也可以在JSX.IntrinsicElements上指定一个用来捕获所有字符串索引
    }
}

<foo />; // 正确
<bar />; // 错误, 因为它没在JSX.IntrinsicElements里指定
```

### 基于值的元素

基于值的元素会简单的在它所在的作用域里按标识符查找

```tsx
import MyComponent from "./myComponent";
                                
<MyComponent />; // 正确
<SomeOtherComponent />; // 错误
```

有**两种方式可以定义基于值的元素**：

1. 无状态函数组件 (SFC)
2. 类组件

由于**这两种基于值的元素在JSX表达式里无法区分**，因此TypeScript首先会尝试将表达式做为无状态函数组件进行解析。如果解析成功，那么TypeScript就完成了表达式到其声明的解析操作。

如果按照无状态函数组件解析失败，那么TypeScript会继续尝试以类组件的形式进行解析。

如果依旧失败，那么将输出一个错误。

### 无状态函数组件

正如其名，组件被定义成JavaScript函数，它的第一个参数是`props`对象。 TypeScript会强制它的返回值可以赋值给`JSX.Element`。

```ts
interface FooProp {
    name: string;
    X: number;
    Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
    return <AnotherComponent name={prop.name} />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

由于无状态函数组件是简单的JavaScript函数，所以我们还可以利用函数重载。

```ts
interface ClickableProps {
    children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
    home: JSX.Element;
}

interface SideProps extends ClickableProps {
    side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
    ...
}
```

### 类组件

定义类组件的类型: *元素类的类型*  和  *元素实例的类型*。

现在有`<Expr />`，*元素类的类型*为`Expr`的类型。 如果 `Expr `是ES6的类，那么**类类型就是类的构造函数和静态部分**。 如果`Expr `是个工厂函数，**类类型为这个函数**。

一旦建立起了类类型，**实例类型由类构造器或调用签名（如果存在的话）的返回值的联合构成**。 

**在ES6类的情况下，实例类型为这个类的实例的类型，并且如果是工厂函数，实例类型为这个函数返回值类型。**

```ts
class MyComponent {
    render() {}
}

// 使用构造签名
var myComponent = new MyComponent();

// 元素类的类型 => MyComponent
// 元素实例的类型 => { render: () => void }

function MyFactoryFunction() {
    return {
    render: () => {
    }
    }
}

// 使用调用签名
var myComponent = MyFactoryFunction();

// 元素类的类型 => FactoryFunction
// 元素实例的类型 => { render: () => void }
```

元素的实例类型必须赋值给`JSX.ElementClass`或抛出一个错误。 默认的`JSX.ElementClass`为`{}`，但是它可以被扩展用来限制JSX的类型以符合相应的接口。

```ts
declare namespace JSX {
    interface ElementClass {
    render: any;
    }
}

class MyComponent {
    render() {}
}
function MyFactoryFunction() {
    return { render: () => {} }
}

<MyComponent />; // 正确
<MyFactoryFunction />; // 正确

class NotAValidComponent {}
function NotAValidFactoryFunction() {
    return {};
}

<NotAValidComponent />; // 错误
<NotAValidFactoryFunction />; // 错误
```

### 属性类型检查

<!--余下待完....-->





## 小结

1. **TypeScript在`.tsx`文件里禁用了使用尖括号的类型断言，使用`as`操作符改写。`as`操作符在`.ts`和`.tsx`里都可用，并且与尖括号类型断言行为是等价的。**

   ```tsx
   var foo = <foo>bar;
   <!-- 改为 -->
   var foo = bar as foo;
   ```

   

