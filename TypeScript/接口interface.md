# interface 接口

> 自定义接口，接口中定义了接口名称和一些方法名称。 说得简单些，接口就是自定义规则。
>
> 接口只是用来定义某些方法名称，它并不需要实现这些方法，比如Duck接口中不需要实现swim方法。接口要做的是记住方法名称。
>
> 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
>
> 接口能够描述JavaScript中对象拥有的各种各样的外形。 除了描述带有属性的普通对象外，接口也可以描述函数类型。

## 对象类型

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

### 可选属性

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

#### (注意)可选属性的额外检查

```tsx
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

**注意 **传入`createSquare`的参数拼写为*`colour`*而不是`color`。 在JavaScript里，这会默默地失败。而在typescript里，对象字面量会被特殊对待而且会经过 *额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

```ts
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```

绕开这些检查非常简单， 

​	**最简便的方法**是使用类型断言：

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

​	**怪异的方式**是对象赋值给一个另一个变量，因为 `squareOptions`不会经过额外属性检查，所以编译器不会报错。

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

​	**最佳的方式**是能够添加一个字符串**索引签名**(前提是你能够确定这个对象可能具有某些做为特殊用途使用的额外属性)

```tsx
// 在这表示的是SquareConfig可以有任意数量的属性，并且只要它们不是color和width，那么就无所谓它们的类型是什么
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

### 只读属性

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

## 函数类型

为了使用接口表示函数类型，先给接口定义一个调用签名。 **它就像是一个只有参数列表和返回值类型的函数定义 **。参数列表里的每个参数都需要名字和类型。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

这样定义后，就可以像使用其它接口一样使用这个函数类型的接口。

```ts
let mySearch: SearchFunc; // 1.创建一个函数类型的变量
mySearch = function(source: string, subString: string) { // 2.将一个同类型的函数赋值给这个变量
  let result = source.search(subString);
  return result > -1;
}
```

### 函数类型的类型检查

**函数的参数名不需要与接口里定义的名字相匹配**

```tsx
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean { // 函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。
  let result = src.search(sub);
  return result > -1;
}
```

如果**不指定类型**，TypeScript的类型系统会推断出参数类型，因为函数直接赋值给了 `SearchFunc`类型变量，函数的返回值类型是通过其返回值推断出来的( 如果让这个函数返回数字或字符串，类型检查器会警告我们函数的返回值类型与 `SearchFunc`接口中的定义不匹配)

```tsx
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

## 可索引的类型

> 描述那些能够“通过索引得到”的类型，比如`a[10]`或`ageMap["daniel"]`。 可索引类型具有一个 *索引签名*，它描述了对象索引的类型，还有相应的索引返回值类型。

```tsx
interface StringArray {
  [index: number]: string; //  这个索引签名表示了当用 number去索引StringArray时会得到string类型的返回值
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

### TypeScript支持两种索引签名：字符串和数字。

**可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型 **。 这是因为当使用 `number`来索引时，JavaScript会将它转换成`string`然后再去索引对象。 也就是说用 `100`（一个`number`）去索引等同于使用`"100"`（一个`string`）去索引，因此两者需要保持一致。

```ts
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// 错误：使用数值型的字符串索引，有时会得到完全不同的Animal!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

**字符串索引签名能够很好的描述`dictionary`模式，并且它们也会确保所有属性与其返回值类型相匹配。 因为字符串索引声明了 `obj.property`和`obj["property"]`两种形式都可以。**

```ts
interface NumberDictionary {
  [index: string]: number;
  length: number;    // 可以，length是number类型
  name: string       // 错误，`name`的类型与字符串索引类型返回值的类型不匹配
}
```

最后，你可以**将索引签名设置为只读，这样就防止了给索引赋值 **：

```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error! 不能设置myArray[2]，因为索引签名是只读的
```

## 类类型

