> JavaScript 模块化标准包括ES Module、CommonJS、AMD 和 UMD 

### **ES Module**

> ES Module 是 ECMAScript 标准中的模块化方案，它是 JavaScript 语言原生支持的模块化格式。它使用 `import` 和 `export` 关键字来实现模块的导入和导出。

```js
// moduleA.js
export const value = 1;

export function sayHello(name) {
  console.log(`Hello, ${name}!`);
}
```

在另一个文件中可以这样导入：

```javascript
import { value, sayHello } from './moduleA.js';
console.log(value);
sayHello('World');
```

也可以使用动态导入语法 `import()`：

```js
button.addEventListener('click', () => {
  import('./moduleA.js').then(module => {
    module.sayHello('ESModule');
  });
});
```

- **加载方式**
  - ES Module 既可以**动态加载**，也可以**静态加载**。在静态加载时，浏览器或者工具（如 Node.js 的 ES Module 支持）会根据 `import` 语句**<u>在编译阶段解析模块的依赖关系</u>**，并以合理的方式加载模块。
  - 动态导入 `import()` 是一种异步加载方式，它返回一个 `Promise`，当模块加载并执行完成后，`Promise` 会 `resolve` 为模块的命名空间对象。

- **适用场景**

  目前已经成为主流的模块化方案，广泛应用于现代前端开发（如 Vue、React 等框架都支持 ES Module）。在浏览器端和 Node.js（从 8.x 版本开始逐步支持 ES Module）环境中都有良好的支持，是开发现代 JavaScript 应用程序的首选模块化格式。

### **CommonJS**（服务端）

> CommonJS 是服务器端模块化规范，主要用于 Node.js 环境。它使用 `require` 关键字来引入模块，使用 `module.exports` 或 `exports` 来导出模块。

```js
// math.js
const add = (a,b) => a + b;
const subtract = (a,b) => a - b;

module.exports = {
  add: add,
  subtract: subtract
};
```

```js
// 在另一个文件中可以通过 require 来引入
const math = require('./math.js');
console.log(math.add(1,2)); // 输出 3
```

- **加载方式**
  - CommonJS 是基于文件的模块加载系统。它是`同步加载`模块的，也就是说，当代码执行到 `require` 语句时，会立即读取并执行被引用的模块文件，然后将模块的导出内容返回。
  - 这种同步加载方式在服务器端环境（如 Node.js）通常是可行的，因为服务器端的文件系统可以快速访问本地文件，并且同步加载可以保证模块的依赖关系在执行过程中能够正确地按顺序加载和解析。
- **适用场景**
  - 主要适用于服务器端开发，如使用 Node.js 构建的后端应用。它能够很好地处理文件系统操作、网络请求等后端任务的模块化需求。

### **AMD**（浏览器端）

> AMD （Asynchronous Module Definition）是一种浏览器端的模块化规范。它主要用于解决浏览器端异步加载模块的问题，因为浏览器端需要通过网络获取模块文件，不能像服务器端那样同步加载。

AMD 使用 `define` 函数来定义模块，使用 `require` 函数来加载模块。例如：

```js
// 定义一个模块
define('math', ['module1', 'module2'], function(module1, module2) {
  const add = (a,b) => a + b;
  const subtract = (a,b) => a - b;

  return {
    add: add,
    subtract: subtract
  };
});
// 加载模块
require(['math'], function(math) {
  console.log(math.add(1,2));
});
```

- **加载方式**
  - AMD 是**异步加载模块**的。它会在后台通过脚本标签或者其他异步方式加载模块文件，而不会阻塞主线程的执行。这样可以提高浏览器端的性能，因为模块文件可以在后台逐步加载，同时其他页面内容也可以继续渲染。
  - 这种异步加载机制非常适合浏览器环境，尤其是在需要加载多个模块文件时，能够有效地利用网络带宽，减少页面加载时间。
- **适用场景**
  - 适用于浏览器端的前端开发，尤其是在需要同时加载多个模块文件的场景下。很多早期的前端项目和框架（如RequireJS）都采用 AMD 规范来实现模块化。

### **UMD**（多端）

> UMD **（Universal Module Definition）**是一种通用模块定义格式，它的目标是能够在不同的环境（包括 CommonJS、AMD 和浏览器全局变量环境）下使用。它结合了 CommonJS 和 AMD 的特点，通过一些条件判断来适配不同的环境。

```js
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD 环境
    define(['module1', 'module2'], factory);
  } else if (typeof module === 'object' && module.exports) {
    // CommonJS 环境
    module.exports = factory(require('module1'), require('module2'));
  } else {
    // 浏览器全局变量环境
    root.myModule = factory(root.module1, root.module2);
  }
}(this, function (module1, module2) {
  // 模块的实现代码
  return {
    // 公开的接口
  };
}));
```

- **加载方式**

  UMD 模块会根据不同的环境来决定加载方式。在 AMD 环境下使用异步加载，在 CommonJS 环境下使用同步加载，在浏览器全局变量环境下直接将模块定义为全局变量。

- **适用场景**

  当需要开发一个可以在多种环境（如同时支持 Node.js 后端和浏览器前端）下使用的模块库时，UMD 是一个很好的选择。它为模块的跨环境使用提供了便利，使得开发者无需为不同的环境编写不同的模块定义代码。