# interface 接口

> 自定义接口，接口中定义了接口名称和一些方法名称。 说得简单些，接口就是自定义规则。
>
> 接口只是用来定义某些方法名称，它并不需要实现这些方法，比如Duck接口中不需要实现swim方法。接口要做的是记住方法名称。
>
> 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

## 接口工作

```tsx
//interface LabelledValue {
//  label: string;
//}
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

类型检查器会查看`printLabel`的调用。 `printLabel`有一个参数，并要求这个对象参数有一个名为`label`类型为`string`的属性。`LabelledValue`接口就好比一个名字，用来描述上面例子里的要求。 它代表了有一个 `label`属性且类型为`string`的对象。 

## 可选属性

> 接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在。 可选属性在应用“option bags”模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号

```tsx
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

**可选属性的好处**:

1. 是可以对可能存在的属性进行预定义

2. 是可以捕获引用了不存在的属性时的错误。

## 只读属性

>  在属性名前用 `readonly`来指定只读属性，并且对象属性只能在对象刚刚创建的时候修改其值

```tsx
interface Point {
    readonly x: number;
    readonly y: number;
}
// 通过赋值一个对象字面量来构造一个Point。 赋值后， x和y再也不能被改变了。
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有`ReadonlyArray<T>`类型，它与`Array<T>`相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改：

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

上面代码的最后一行，可以看到就算把整个`ReadonlyArray`赋值到一个普通数组也是不可以的。 但是可以用类型断言重写：

```ts
a = ro as number[];
```

### **readonly与const的选择**

 做为变量使用的话用 `const`，若做为属性则使用`readonly`。

