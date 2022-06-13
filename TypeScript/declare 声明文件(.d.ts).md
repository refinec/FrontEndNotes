# `declare`声明文件

> 当使用[第三方库](https://so.csdn.net/so/search?q=第三方库&spm=1001.2101.3001.7020)时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。

```ts
declare var 声明全局变量
declare function 声明全局方法
declare class 声明全局类
declare enum 声明全局枚举类型
declare namespace 声明（含有子属性的）全局对象
interface 和 type 声明全局类型
/// <reference /> 三斜线指令
```



## 结构

> 了解常见库的格式以及如何为每种格式书写正确的声明文件。

**识别库的类型是书写声明文件的第一步**。

### 全局库

> 全局库模版: 模版文件[`global.d.ts`](#global.d.ts)

*全局*库是指能在全局命名空间下访问的（例如：不需要使用任何形式的`import`）。 许多库都是简单的暴露出一个或多个全局变量。 比如，如果你使用过 [jQuery](https://jquery.com/)，`$`变量可以被够简单的引用以及经常会在全局库的指南文档上看到如何在HTML里用脚本标签引用库：

```tsx
<script src="http://a.great.cdn.for/someLib.js"></script>
```

**目前，大多数流行的全局访问型库实际上都以**UMD库**的形式进行书写。 UMD库的文档很难与全局库文档两者之间难以区分。 在书写全局声明文件前，一定要确认一下库是否真的不是UMD。**

```ts
// 全局库的代码通常都十分简单
function createGreeting(s) {
    return "Hello, " + s;
}
// 或这样：
window.createGreeting = function(s) {
    return "Hello, " + s;
}
```

当你查看全局库的源代码时，你通常会看到：

- 顶级的`var`语句或`function`声明
- 一个或多个赋值语句到`window.someName`
- 假设DOM原始值像`document`或`window`是存在的

你*不会*看到：

- 检查是否使用或如何使用模块加载器，比如`require`或`define`
- `CommonJS/Node.js`风格的导入如`var fs = require("fs");`
- `define(...)`调用
- 文档里说明了如何去`require`或导入这个库

**注意：** 在书写全局声明文件时，允许在全局作用域里定义很多类型。但它会导致无法处理的命名冲突。

一个简单的规则是使用库定义的全局变量名来声明命名空间类型。 比如，库定义了一个全局的值 `cats`，你可以这样写

```ts
// 这样也保证了库在转换成UMD的时候没有任何的破坏式改变
declare namespace cats {
    interface KittySettings { }
}
// 不要在顶层这样写
interface CatsKittySettings { }
```

### 模块化库

一些库只能工作在模块加载器的环境下。 比如，像 `express`只能在Node.js里工作所以必须使用`CommonJS`的`require`函数加载

```ts
// 对于JavaScript CommonJS （Node.js）
var fs = require("fs");
// 对于TypeScript或ES6，import关键字也具有相同的作用：
import fs = require("fs");
// 通常会在模块化库的文档里看到如下说明：
var someLib = require('someLib');

define(..., ['someLib'], function(someLib) {

});
```

模块库至少会包含下列具有代表性的条目之一：

- 无条件的调用`require`或`define`
- 像`import * as a from 'b';` or `export c;`这样的声明
- 赋值给`exports`或`module.exports`

它们极少包含：

- 对`window`或`global`的赋值

### *UMD*

*UMD*模块是指那些既可以作为模块使用（通过导入）又可以作为全局（在没有模块加载器的环境里）使用的模块。比如 [Moment.js](http://momentjs.com/)，就是这样的形式。 

```js
// 比如，在Node.js或RequireJS里，你可以这样写：
import moment = require("moment");
console.log(moment.format());

// 在纯净的浏览器环境里你也可以这样写：
console.log(moment.format());
```

[UMD模块](https://github.com/umdjs/umd)会检查是否存在模块加载器环境。 这是非常形容观察到的模块，它们会像下面这样：

```js
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    } else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {
```

UMD库的文档里经常会包含通过`require`“**在Node.js里使用**”例子， 和**“在浏览器里使用”**的例子，展示如何**使用 `<script>`标签去加载脚本**。

大多数流行的库现在都能够被当成UMD包。 比如 [jQuery](https://jquery.com/),[Moment.js](http://momentjs.com/),[lodash](https://lodash.com/)和许多其它的。



**针对模块有三种可用的模版， [`module.d.ts`](#module.d.ts), [`module-class.d.ts`](#module-class.d.ts) 和 [`module-function.d.ts`](#module-function.d.ts).**

* 使用[`module-function.d.ts`](#module-function.d.ts)，如果模块能够作为函数*调用*。

```ts
var x = require("foo");
// Note: calling 'x' as a function
var y = x(42);
```

**注意：** 在ES6模块加载器里，顶层的对象（这里以`x`导入）只能具有属性， 顶层的模块对象 *永远不能*被调用。 十分常见的**解决方法**是定义一个 `default`导出到一个可调用的/可构造的对象； 一会模块加载器助手工具能够自己探测到这种情况并且使用 `default`导出来替换顶层对象。

* 使用[`module-class.d.ts`](#module-class.d.ts)如果模块能够使用`new`来*构造*：

```ts
var x = require("bar");
// Note: using 'new' operator on the imported variable
var y = new x("hello");
```

* 如果模块不能被调用或构造，使用[`module.d.ts`](#module.d.ts)文件。

### *模块插件*或*UMD插件*

> 一个*模块插件*可以改变一个模块的结构（UMD或模块）。 例如，在Moment.js里， `moment-range`添加了新的`range`方法到`monent`对象。

使用[`module-plugin.d.ts`](#module-plugin.d.ts)模版

### 全局插件

> 个*全局插件*是全局代码，它们会改变全局对象的结构。 对于 *全局修改的模块*，在运行时存在冲突的可能。

比如，一些库往`Array.prototype`或`String.prototype`里添加新的方法。

```js
var x = "hello, world";
// 对内置类型创建新方法
console.log(x.startsWithHello());

var y = [1, 2, 3];
// 对内置类型创建新方法
console.log(y.reverseAndSort());
```

**使用[`global-plugin.d.ts`](#global-plugin.d.ts)模版**

### 全局修改的模块

> 当一个*全局修改的模块*被导入的时候，它们会改变全局作用域里的值。 比如，存在一些库它们添加新的成员到 `String.prototype`当导入它们的时候。

它们与全局插件相似，但是需要 `require`调用来激活它们的效果。

```ts
// 不使用其返回值的“require”调用
var unused = require("magic-string-time");
/* or */
require("magic-string-time");

var x = "hello, world";
// 对内置类型创建新方法
console.log(x.startsWithHello());

var y = [1, 2, 3];
// 对内置类型创建新方法
console.log(y.reverseAndSort());
```

**使用[`global-modifying-module.d.ts`](#global-modifying-module.d.ts)模版。**

### 使用依赖

1. 如果你的**库依赖于某个全局库**，使用`/// <reference types="..." />`指令：

   ```ts
   /// <reference types="someLib" />
   
   function getThing(): someLib.thing;
   ```

2. 如果你的**库依赖于模块**，使用`import`语句：

   ```ts
   import * as moment from "moment";
   
   function getThing(): moment;
   ```

3. 如果你的**全局库依赖于某个UMD模块**，使用`/// <reference types`指令：

   ```ts
   /// <reference types="moment" />
   
   function getThing(): moment;
   ```

4. 你的**模块或UMD库依赖于一个UMD库**，*不要*使用`/// <reference`指令去声明UMD库的依赖，应使用`import`语句：

   ```ts
   import * as someLib from 'someLib';
   ```

## 举例

### 全局变量

***使用`declare var`声明变量***。 

如果变量是只读的，那么可以使用 `declare const`。 

如果变量拥有块级作用域，还可以使用 `declare let`。

```ts
// 全局变量foo包含了存在组件总数
declare var foo: number;
console.log("Half the number of widgets is " + (foo / 2));
```

### 全局函数

***使用`declare function`声明函数***。

```ts
// 用一个字符串参数调用greet函数向用户显示一条欢迎信息。
declare function greet(greeting: string): void;
greet("hello, world");
```

### 带属性的对象

***使用`declare namespace`描述用点表示法访问的类型或值***。

```ts
// 全局变量myLib包含一个makeGreeting函数， 还有一个属性 numberOfGreetings指示目前为止欢迎数量。
declare namespace myLib {
    function makeGreeting(s: string): string;
    let numberOfGreetings: number;
}
    
let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);

let count = myLib.numberOfGreetings;
```

### 函数重载

```ts
// getWidget函数接收一个数字，返回一个组件，或接收一个字符串并返回一个组件数组。
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];

let x: Widget = getWidget(43);
let arr: Widget[] = getWidget("all of them");
```

### 可重用类型（接口）

***使用`interface`定义一个带有属性的类型***。

```ts
//当指定一个欢迎词时，你必须传入一个GreetingSettings对象。 这个对象具有以下几个属性：
//1- greeting：必需的字符串
//2- duration: 可靠的时长（毫秒表示）
//3- color: 可选字符串，比如‘#ff00ff’
interface GreetingSettings {
  greeting: string;
  duration?: number;
  color?: string;
}

declare function greet(setting: GreetingSettings): void;

greet({
  greeting: "hello world",
  duration: 4000
});
```

### 可重用类型（类型别名）

***使用类型别名来定义类型的短名***：

```ts
// 在任何需要欢迎词的地方，你可以提供一个string，一个返回string的函数或一个Greeter实例
type GreetingLike = string | (() => string) | MyGreeter;

declare function greet(g: GreetingLike): void;
 // 使用
function getGreeting() {
    return "howdy";
}
class MyGreeter extends Greeter { }

greet("hello");
greet(getGreeting);
greet(new MyGreeter());
```

### 组织类型

***使用命名空间组织类型***。

```ts
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

***也可以在一个声明中创建嵌套的命名空间*：**

```ts
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

```ts
// greeter对象能够记录到文件或显示一个警告。 你可以为 .log(...)提供LogOptions和为.alert(...)提供选项。
const g = new Greeter("Hello");
g.log({ verbose: true });
g.alert({ modal: false, title: "Current Greeting" });
```

### 类

使用`declare class`描述一个类或像类一样的对象。 类可以有属性和方法，就和构造函数一样。

```ts
declare class Greeter {
    constructor(greeting: string);

    greeting: string;
    showGreeting(): void;
}
// 通过实例化Greeter对象来创建欢迎词，或者继承Greeter对象来自定义欢迎词。
const myGreeter = new Greeter("hello, world");
myGreeter.greeting = "howdy";
myGreeter.showGreeting();

class SpecialGreeter extends Greeter {
    constructor() {
        super("Very special greetings");
    }
}
```

## 规范

### 普通类型

***不要*** 使用如下大写的类型`Number`，`String`，`Boolean`或`Object`。 这些类型指的是非原始的装盒对象，它们几乎没在JavaScript代码里正确地使用过。

```ts
/* 错误 */
function reverse(s: String): String;
// 使用类型number，string，boolean，object
/* OK */
function reverse(s: string): string;
```

### 泛型

***不要***  定义一个从来没使用过其类型参数的泛型类型。

### 回调函数类型

#### 回调函数返回值类型

*不要*为返回值被忽略的回调函数设置一个`any`类型的返回值类型：

```ts
/* 错误 */
function fn(x: () => any) {
    x();
}

// 应该给返回值被忽略的回调函数设置 void类型 的返回值类型：
/* OK */
function fn(x: () => void) {
    var k = x();
    k.doSomething(); // 使用void相对安全，因为它防止了你不小心使用x的返回值
}
```

#### 回调函数里的可选参数

***不要*  在回调函数里使用可选参数**除非你真的要这么做

```ts
/* 错误 */
interface Fetcher {
    getObject(done: (data: any, elapsedTime?: number) => void): void;
}
/* OK */
interface Fetcher {
    getObject(done: (data: any, elapsedTime: number) => void): void;
}
```

#### 重载与回调函数

***不要*  因为回调函数参数个数不同而写不同的重载，*应该* 只使用最大参数个数写一个重载：**

回调函数总是可以忽略某个参数的，因此没必要为参数少的情况写重载。 参数少的回调函数首先允许错误类型的函数被传入，因为它们匹配第一个重载

```ts
/* 错误 */
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
/* OK */
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

### 函数重载

***不要*   把一般的重载放在精确的重载前面，*应该*排序重载令精确的排在一般的之前**：

```ts
/* OK */
declare function fn(x: HTMLDivElement): string;
declare function fn(x: HTMLElement): number;
declare function fn(x: any): any;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: string, :)
```

***不要*  为仅在末尾参数不同时写不同的重载，*应该*  尽可能使用可选参数：**

```ts
/* 错误 */
interface Example {
    diff(one: string): number;
    diff(one: string, two: string): number;
    diff(one: string, two: string, three: boolean): number;
}
/* OK */
interface Example {
    diff(one: string, two?: string, three?: boolean): number;
}
```

***不要*   为仅在某个位置上的参数类型不同的情况下定义重载，*应该*  尽可能地使用联合类型：**

```ts
/* OK */
interface Moment {
    utcOffset(): number;
    utcOffset(b: number|string): Moment;
}
```

## 使用声明文件

获取类型声明文件只需要使用`npm`，比如，获取`lodash`库的声明文件，只需使用下面的命令：

```sh
npm install --save @types/lodash
```

下载完后，就可以直接在TypeScript里使用`lodash`了。 不论是在模块里还是全局代码里使用。

```ts
import * as _ from "lodash";
_.padStart("Hello TypeScript!", 20, " ");
// 或者如果你没有使用模块，那么你只需使用全局的变量_
_.padStart("Hello TypeScript!", 20, " ");
```

大多数情况下，类型声明包的名字总是与它们在`npm`上的包的名字相同，有`@types/`前缀，在 https://aka.ms/types查找库。

## 模板

### <span id="global.d.ts">global.d.ts</span>

```tsx
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 如果这个库是可调用的(例如，可以作为 myLib (3)调用),
 *~ 在这里包括那些调用签名.
 *~ 否则，删除此部分。
 */
declare function myLib(a: string): string;
declare function myLib(a: number): number;

/*~ 如果希望此库的名称是有效的类型名，可以在此处执行此操作。
 *~
 *~ 例如，写 ‘var x: myLib’
 *~ 确保这是有意义的！如果没有，只需删除此声明并在下面的命名空间中添加类型。
 */
interface myLib {
    name: string;
    length: number;
    extras?: string[];
}

/*~ 如果您的库在全局变量上公开了属性，请将它们放在此处。
 *~ 您还应该在这里放置类型（接口和类型别名）。
 */
declare namespace myLib {
    //~ 可以写 'myLib.timeout = 50;'
    let timeout: number;

    //~ 可以访问 'myLib.version', 但不能改变它
    const version: string;

    //~ 可以通过 ‘let c = new myLib. Cat (42)’创建一些类
    //~ 或参考  'function f(c: myLib.Cat) { ... }
    class Cat {
        constructor(n: number);

        //~ 从cat实例中读出‘ c. age’
        readonly age: number;

        //~ 从cat实例中调用'c.purr()'
        purr(): void;
    }

    //~  将变量声明为 'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
    interface CatSettings {
        weight: number;
        name: string;
        tailLength?: number;
    }

    //~ 可写 'const v: myLib.VetID = 42;'
    //~  或者 'const v: myLib.VetID = "bob";'
    type VetID = string | number;

    //~ 可调用 'myLib.checkCat(c)' or 'myLib.checkCat(c, v);'
    function checkCat(c: Cat, s?: VetID);
}
```

### <span id="global-modifying-module.d.ts">global-modifying-module.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个全局修改的模块模板文件.
 *~ 你应该把它重命名为index.d.ts，并将其放在与模块同名的文件夹中。
 *~ 例如，如果你要为"super-greeter"写一个文件，这个文件应该是'super-greeter/index.d.ts'
 */

/*~ 注意:如果你的全局修改模块是可调用或可构造的，
 *~ 你需要将这里的模式与module-class或module-function模板文件中的模式结合起来。
 */
declare global {
    /*~ 在这里，声明全局命名空间中的内容，或者扩充全局命名空间中的现有声明。
     */
    interface String {
        fancyFormat(opts: StringFormatOptions): string;
    }
}

/*~ 如果您的模块导出 类型或值，请照常编写它们 */
export interface StringFormatOptions {
    fancinessLevel: number;
}

/*~ 例如，在模块上声明一个方法（除了它的全局副作用） */
export function doSomething(): void;

/*~ 如果您的模块没有导出任何内容，则需要这一行。否则，删除它 */
export { };
```

### <span id="global-plugin.d.ts">global-plugin.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 此模板显示如何编写全局插件 */

/*~ 为原始类型编写声明并添加新成员。
 *~ 例如，这将添加一个'toBinaryString'方法来重载内置的数字类型。
 */
interface Number {
    toBinaryString(opts?: MyLibrary.BinaryFormatOptions): string;
    toBinaryString(callback: MyLibrary.BinaryFormatCallback, opts?: MyLibrary.BinaryFormatOptions): string;
}

/*~ 如果需要声明几种类型，请将它们放在一个名称空间中，以避免向全局名称空间添加太多内容。
 */
declare namespace MyLibrary {
    type BinaryFormatCallback = (n: number) => string;
    interface BinaryFormatOptions {
        prefix?: string;
        padding: number;
    }
}
```

### <span id="module-class.d.ts">module-class.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是class类模块的模块模板文件。.
 *~ 您应该将其重命名为index.d.ts，并将其放置在与模块同名的文件夹中。
 *~ 例如，如果您正在为“super-greeter”编写一个文件，那么这个文件应该是“super-greeter/index.d.ts”
 */

/*~ 注意，ES6模块不能直接导出类对象。
 *~ 应使用CommonJS风格导入此文件: import x = require('someLibrary');
 *~
 *~ 请参阅文档以了解ES6模块的此限制的常见解决方法。
 */

/*~ 如果此模块是一个UMD模块，当在模块加载程序环境外部加载时，
 *~ 它会公开一个全局变量“myClassLib”，请在此处声明该全局变量。
 *~ 否则，删除此声明。
 */
export as namespace myClassLib;

/*~ 此声明指定类构造函数是从文件导出的对象
 */
export = MyClass;

/*~ 在此类中编写模块的方法和属性 */
declare class MyClass {
    constructor(someParam?: string);

    someProperty: string[];

    myMethod(opts: MyClass.MyClassMethodOptions): number;
}

/*~ 如果您也想公开模块中的类型，可以将它们放在这个块中。
 */
declare namespace MyClass {
    export interface MyClassMethodOptions {
        width?: number;
        height?: number;
    }
}
```

### <span id="module-function.d.ts">module-function.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是function模块的模块模板文件。.
 *~ 您应该将其重命名为index.d.ts，并将其放置在与模块同名的文件夹中。
 *~ 例如，如果您正在为“super-greeter”编写一个文件，那么这个文件应该是“super-greeter/index.d.ts”
 */

/*~ 注意，ES6模块不能直接导出可调用函数。
 *~ 此文件应使用CommonJS导入: import x = require('someLibrary');
 *~
 *~ 请参阅文档以了解ES6模块的此限制的常见解决方法。
 */

/*~ 如果此模块是在模块加载程序环境外部加载时公开全局变量“myFuncLib”的UMD模块，
 *~ 请在此处声明该全局变量。否则，删除此声明。
 */
export as namespace myFuncLib;

/*~ 此声明指定函数是从文件导出的对象
 */
export = MyFunction;

/*~ 此示例演示如何为函数设置多个重载 */
declare function MyFunction(name: string): MyFunction.NamedReturnType;
declare function MyFunction(length: number): MyFunction.LengthReturnType;

/*~ 如果您也想公开模块中的类型，可以将它们放在这个块中。
 *~ 通常，您需要描述函数返回类型的形状；该类型应该在这里声明，如本例所示。
 */
declare namespace MyFunction {
    export interface LengthReturnType {
        width: number;
        height: number;
    }
    export interface NamedReturnType {
        firstName: string;
        lastName: string;
    }

    /*~ 如果模块也有属性，请在此处声明它们。例如，本声明称本守则是合法的：
     *~   import f = require('myFuncLibrary');
     *~   console.log(f.defaultName);
     */
    export const defaultName: string;
    export let defaultLength: number;
}
```

### <span id="module-plugin.d.ts">module-plugin.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是 模块插件module-plugin 模板文件。
 *~ 您应该将其重命名为index.d.ts，并将其放置在与模块同名的文件夹中。
 *~ 例如，如果您正在为“super-greeter”编写一个文件，那么这个文件应该是“super-greeter/index.d.ts”
 */

/*~ 在这一行中，import the module which this module adds to */
import * as m from 'someModule';

/*~ 如果需要，还可以导入其他模块 */
import * as other from 'anotherModule';

/*~ 在这里，声明与上面导入的模块相同的模块 */
declare module 'someModule' {
    /*~ 在内部，添加新函数、类或变量。
     *~ 如果需要，可以使用原始模块中的未报告类型。 */
    export function theNewMethod(x: m.foo): other.bar;

    /*~ 还可以通过编写接口扩展，向原始模块的现有接口添加新属性  */
    export interface SomeModuleOptions {
        someModuleSetting?: string;
    }

    /*~ 还可以声明新类型，并将它们显示为原始模块中的类型 */
    export interface MyModulePluginOptions {
        size: number;
    }
}
```

### <span id="module.d.ts">module.d.ts</span>

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是module模块模板文件。
 *~ 您应该将其重命名为index.d.ts，并将其放置在与模块同名的文件夹中。
 *~ 例如，如果您正在为“super-greeter”编写一个文件，那么这个文件应该是“super-greeter/index.d.ts”
 */

/*~ 如果此模块是一个UMD模块，当在模块加载程序环境外部加载时，
 *~ 它会公开一个全局变量“myLib”，请声明该全局变量在这里。否则，删除此声明。
 */
export as namespace myLib;

/*~ 如果此模块有方法，请将它们声明为函数，如下所示。
 */
export function myMethod(a: string): string;
export function myOtherMethod(a: number): number;

/*~ 您可以通过导入模块来声明可用的类型 */
export interface someType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ 可以使用const、let或var声明模块的属性 */
export const myField: number;

/*~ 如果模块的虚线名称中有类型、属性或方法，请在“命名空间”中声明它们。 */
export namespace subProp {
    /*~ 例如，根据这个定义，有人可以写：
     *~   import { subProp } from 'yourModule';
     *~   subProp.foo();
     *~ 或者
     *~   import * as yourMod from 'yourModule';
     *~   yourMod.subProp.foo();
     */
    export function foo(): void;
}
```

