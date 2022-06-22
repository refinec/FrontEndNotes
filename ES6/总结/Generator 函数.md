# Generator 函数

## 语法

> Generator 函数，原生具有 Iterator 接口，它返回一个 Generator 对象，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象（Iterator Object），生成器函数在执行时能暂停，后面又能从暂停处继续执行。调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止

* **yield表达式如果用在另一个表达式之中，必须放在圆括号里面**

  ```javascript
  function* demo() {
    console.log('Hello' + yield); // SyntaxError
    console.log('Hello' + yield 123); // SyntaxError
    console.log('Hello' + (yield)); // OK
    console.log('Hello' + (yield 123)); // OK
  }
  ```

* **yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号**

  ```javascript
  function* demo() {
      foo(yield 'a', yield 'b'); // OK
      let input = yield; // OK
      }
  ```

* **任意一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口**。

### next方法的参数

* **yield表达式本身没有返回值，或者说总是返回`undefined `。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值，但第一次使用next方法时，传递参数是无效的 **

  ```javascript
  function* f() {
      for(var i = 0; true; i++) {
          // 每次yield 执行完之后停止执行，next的参数作为该yield表达式的返回值赋值给reset
          var reset = yield i; 
          if(reset) { i = -1; }
      }
  }
  var g = f();
  g.next() // { value: 0, done: false }
  g.next() // { value: 1, done: false }
  g.next(true) // { value: 0, done: false }
  
  // 下面代码中，第二次运行next方法的时候不带参数，导致 y 的值等于2 * undefined（即NaN），除以 // 3 以后还是NaN，因此返回对象的value属性也等于NaN。第三次运行Next方法的时候不带参数，所以z
  // 等于undefined，返回对象的value属性等于5 + NaN + undefined，即NaN。
  // 如果向next方法提供参数，返回结果就完全不一样了。上面代码第一次调用b的next方法时，返回x+1
  // 的值6；第二次调用next方法，将上一次yield表达式的值设为12，因此y等于24，返回y / 3的值8；第
  // 三次调用next方法，将上一次yield表达式的值设为13，因此z等于13，这时x等于5，y等于24，所
  // 以return语句的值等于42。
  function* foo(x) {
      var y = 2 * (yield (x + 1));
      var z = yield (y / 3);
      return (x + y + z);
  }
  
  var a = foo(5);
  a.next() // Object{value:6, done:false}
  a.next() // Object{value:NaN, done:false}
  a.next() // Object{value:NaN, done:true}
  
  var b = foo(5);
  b.next() // { value:6, done:false }
  b.next(12) // { value:8, done:false }
  b.next(13) // { value:42, done:true }
  
  ```

### for...of 循环

* **for...of循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法 **。一旦next方法的返回对象的done属性为true 或者 遇到`return`语句，for...of循环就会中止

* 利用for...of循环，可以写出**遍历任意对象（object）**的方法。原生的 JavaScript 对象没有遍历接口，无法使用for...of循环，通过 Generator 函数为它加上这个接口，就可以用了。

  ```javascript
  function* objectEntries(obj) {
      let propKeys = Reflect.ownKeys(obj);
      for (let propKey of propKeys) {
          yield [propKey, obj[propKey]];
      }
  }
  let jane = { first: 'Jane', last: 'Doe' };
  for (let [key, value] of objectEntries(jane)) {
      console.log(`${key}: ${value}`);
  }
  // first: Jane
  // last: Doe
  
  // 第二种方法，将 Generator 函数加到对象的Symbol.iterator属性上
  function* objectEntries() {
      let propKeys = Object.keys(this);
      for (let propKey of propKeys) {
          yield [propKey, this[propKey]];
      }
  }
  let jane = { first: 'Jane', last: 'Doe' };
  
  jane[Symbol.iterator] = objectEntries;
  
  for (let [key, value] of jane) {
      console.log(`${key}: ${value}`);
  }
  // first: Jane
  // last: Doe
  ```

* 除了**`for...of` **循环以外，**`扩展运算符...`   **、**`解构赋值` **和**`Array.from` **方法内部调用的，都是遍历器接口。这意味着，它们都可以将 Generator 函数返回的 Iterator 对象，作为参数

### 实例方法

#### Generator.prototype.throw()

Generator 函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获：

```javascript
var g = function* () {
    try {
        yield;
    } catch (e) {
        console.log('内部捕获', e);
    }
};
var i = g();
i.next();
try {
    i.throw('a'); // 第一个错误被 Generator 函数体内的catch语句捕获
    i.throw('b'); // 第二次抛出错误，由于 Generator 函数内部的catch语句已经执行过了,所以这个错误就被抛出了 Generator 函数体，被函数体外的catch语句捕获
} catch (e) {
    console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```

* throw方法可以接受一个参数，该参数会被catch语句接收，建议抛出Error对象的实例
* 注意，不要混淆遍历器对象的throw方法和全局的throw命令。上面代码的错误，是用遍历器对象的throw方法抛出的，而不是用throw命令抛出的。后者只能被函数体外的catch语句捕获。

* throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法。因为第一次执行next方法，等同于启动执行 Generator 函数的内部代码，否则 Generator 函数还没有开始执行，这时throw方法抛错只可能抛出在函数外部。

* **throw方法被捕获以后，会附带执行下一条yield表达式 **。也就是说，会附带执行一次next方法。只要 Generator 函数内部部署了try...catch代码块，那么遍历器的throw方法抛出的错误，不影响下一次遍历。所以只在 Generator 函数内部写一次catch语句就可以了。**一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了(即使在外部被捕获仍旧无法改变) **。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了

  ```javascript
  function* g() {
      yield 1;
      console.log('throwing an exception');
      throw new Error('generator broke!');
      yield 2;
      yield 3;
  }
  function log(generator) {
      var v;
      console.log('starting generator');
      try {
          v = generator.next();
          console.log('第一次运行next方法', v);
      } catch (err) {
          console.log('捕捉错误', v);
      }
      try {
          v = generator.next();
          console.log('第二次运行next方法', v);
      } catch (err) {
          console.log('捕捉错误', v);
      }
      try {
          v = generator.next();
          console.log('第三次运行next方法', v);
      } catch (err) {
          console.log('捕捉错误', v);
      }
      console.log('caller done');
  }
  log(g());
  // starting generator
  // 第一次运行next方法 { value: 1, done: false }
  // throwing an exception
  // 捕捉错误 { value: 1, done: false }
  // 第三次运行next方法 { value: undefined, done: true }
  // caller done
  ```

#### Generator.prototype.return()

Generator 函数返回的遍历器对象，还有一个return方法，可以**返回给定的值，并且终结遍历 Generator 函数 **。

* 遍历器对象调用return方法后，返回值的value属性就是return方法的参数,如果return方法调用时，不提供参数，则返回值的value属性为undefined.

```javascript
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}
var g = gen();
g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

*  如果 Generator 函数内部有try...finally代码块，且正在执行try代码块，那么return方法会导致立刻进入finally代码块，执行完以后，整个函数才会结束。下面代码中，**调用return()方法后，就开始执行finally代码块，不执行try里面剩下的代码了，然后等到finally代码块执行完，再返回return()方法指定的返回值 **。

  ```javascript
  function* numbers () {
      yield 1;
      try {
          yield 2;
          yield 3;
      } finally {
          yield 4;
          yield 5;
      }
      yield 6;
  }
  var g = numbers();
  g.next() // { value: 1, done: false }
  g.next() // { value: 2, done: false }
  g.return(7) // { value: 4, done: false }
  g.next() // { value: 5, done: false }
  g.next() // { value: 7, done: true }
  ```

### next()、throw()、return() 的共同点

next()、throw()、return()这三个方法本质上是同一件事，可以放在一起理解。它们的作用都是让 Generator 函数恢复执行，并且**使用不同的语句替换yield表达式**。

* **next()是将yield表达式替换成一个值,如果next方法没有参数，就相当于替换成undefined**

  ```javascript
  const g = function* (x, y) {
      let result = yield x + y;
      return result;
  };
  const gen = g(1, 2);
  gen.next(); // Object {value: 3, done: false}
  gen.next(1); // Object {value: 1, done: true}
  // 相当于将 let result = yield x + y
  // 替换成 let result = 1;
  ```

* **throw()是将yield表达式替换成一个throw语句**

  ```javascript
  gen.throw(new Error('出错了')); // Uncaught Error: 出错了
  // 相当于将 let result = yield x + y
  // 替换成 let result = throw(new Error('出错了'));
  ```

* **return()是将yield表达式替换成一个return语句**

  ```javascript
  gen.return(2); // Object {value: 2, done: true}
  // 相当于将 let result = yield x + y
  // 替换成 let result = return 2;
  ```

### yield* 表达式

> 用来在一个 Generator 函数里面执行另一个 Generator 函数，yield* 后面跟一个遍历器对象

```javascript
function* inner() {
    yield 'hello!';
}
function* outer1() {
    yield 'open';
    yield inner();
    yield 'close';
}
var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一个遍历器对象
gen.next().value // "close"

function* outer2() {
    yield 'open'
    yield* inner()
    yield 'close'
}
var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
```

* yield*后面的 Generator 函数（没有return语句时），等同于在 Generator 函数内部，部署一个for...of循环。yield*后面的 Generator 函数（没有return语句时），不过是**for...of **的一种简写形式，**完全可以用后者替代前者 **。反之，在有return语句时，则需要用**var value = yield* iterator的形式获取return语句的值。 **

  ```javascript
  function* concat(iter1, iter2) {
      yield* iter1;
      yield* iter2;
  }
  // 等同于
  function* concat(iter1, iter2) {
      for (var value of iter1) {
          yield value;
      }
      for (var value of iter2) {
          yield value;
      }
  }
  ```

* 如果yield*后面跟着一个数组，由于数组原生支持遍历器，因此就会遍历数组成员。yield命令后面如果不加星号，返回的是整个数组，加了星号就表示返回的是数组的遍历器对象。任何数据结构只要有 Iterator 接口，就可以被yield*遍历

* **yield*命令可以很方便地取出嵌套数组的所有成员（重要） **

  ```javascript
  function* iterTree(tree) {
      if (Array.isArray(tree)) {
          for(let i=0; i < tree.length; i++) {
              yield* iterTree(tree[i]);
          }
      } else {
          yield tree;
      }
  }
  const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];
  for(let x of iterTree(tree)) {
      console.log(x);
  }
  // a
  // b
  // c
  // d
  // e
  
  // 扩展运算符...默认调用 Iterator 接口，所以上面这个函数也可以用于嵌套数组的平铺
  [...iterTree(tree)] // ["a", "b", "c", "d", "e"]
  ```

* **使用yield*语句遍历完全二叉树**

  ```javascript
  // 下面是二叉树的构造函数，
  // 三个参数分别是左树、当前节点和右树
  function Tree(left, label, right) {
      this.left = left;
      this.label = label;
      this.right = right;
  }
  // 下面是中序（inorder）遍历函数。
  // 由于返回的是一个遍历器，所以要用generator函数。
  // 函数体内采用递归算法，所以左树和右树要用yield*遍历
  function* inorder(t) {
      if (t) {
          yield* inorder(t.left);
          yield t.label;
          yield* inorder(t.right);
      }
  }
  // 下面生成二叉树
  function make(array) {
      // 判断是否为叶节点
      if (array.length == 1) return new Tree(null, array[0], null);
      return new Tree(make(array[0]), array[1], make(array[2]));
  }
  let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);
  // 遍历二叉树
  var result = [];
  for (let node of inorder(tree)) {
      result.push(node);
  }
  result
  // ['a', 'b', 'c', 'd', 'e', 'f', 'g']
  ```

### 作为对象属性的 Generator 函数

```javascript
let obj = {
    * myGeneratorMethod() {
        ···
    }
};
```

### Generator 函数的this

* **Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的prototype对象上的方法（但Generator 函数也不能跟new命令一起用，会报错 ） **.但是，如果把g当作普通的构造函数，并不会生效，因为g返回的总是遍历器对象，而不是this对象.

  ```javascript
  function* g() {}
  g.prototype.hello = function () {
      return 'hi!';
  };
  let obj = g();
  obj instanceof g // true
  obj.hello() // 'hi!'
  
  function* g() {
      this.a = 11;
  }
  let obj = g();
  obj.next();
  obj.a // undefined
  ```


### Generator 与状态机

**Generator 是实现状态机的最佳结构 **。比如，下面的clock函数就是一个状态机。

```javascript
var ticking = true;
var clock = function() {
    if (ticking)
        console.log('Tick!');
    else
        console.log('Tock!');
    ticking = !ticking;
}

// 用 Generator 实现,少了用来保存状态的外部变量ticking，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅
var clock = function* () {
    while (true) {
        console.log('Tick!');
        yield;
        console.log('Tock!');
        yield;
    }
};
```

### Generator 与协程

> 协程（coroutine）是一种程序运行的方式，可以理解成“协作的线程”或“协作的函数”。协程既可以用单线程实现，也可以用多线程实现。前者是一种特殊的子例程，后者是一种特殊的线程。

* **协程与子例程的差异**

  **可以并行执行、交换执行权的线程（或函数），就称为协程**。在内存中，子例程只使用一个栈（stack），而**协程是同时存在多个栈，但只有一个栈是在运行状态，也就是说，协程是以多占用内存为代价，实现多任务的并行 **。

* **协程与普通线程的差异**

  不难看出，协程适合用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己的执行上下文、可以分享全局变量。它们的不同之处在于，同一时间可以有多个线程处于运行状态，但是运行的协程只能有一个，其他协程都处于暂停状态。此外，普通的线程是抢先式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。

  由于 JavaScript 是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误的时候，可以找到原始的调用栈。不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。

  Generator 函数是 ES6 对协程的实现，但属于不完全实现。Generator 函数被称为“半协程”（semi-coroutine），意思是只有 Generator 函数的调用者，才能将程序的执行权还给 Generator 函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用yield表达式交换控制权。

### Generator 与上下文

JavaScript 代码运行时，会产生一个全局的上下文环境（context，又称运行环境），包含了当前所有的变量和对象。然后，执行函数（或块级代码）的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文，变成当前（active）的上下文，由此形成一个上下文环境的堆栈（context stack）。这个堆栈是“后进先出”的数据结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有代码执行完成，堆栈清空。Generator 函数不是这样，它执行产生的上下文环境，一旦遇到yield命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会冻结在当前状态。等到对它执行next命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。

## 异步应用

ES6 诞生以前，异步编程的方法，大概有下面四种。

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象