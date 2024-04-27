> async 函数是 Generator 函数的语法糖,它将 Generator 函数的`星号*`替换成`async`，将`yield`替换成`await`。核心逻辑是迭代执行`next`函数。

**async函数对 Generator 函数的改进：**

1. **内置执行器 **

2. **更好的语义 **

3. **更广的适用性 **

   co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）

4. **返回值是 `Promise` **

   **async函数的返回值是 Promise 对象 **，这比 **Generator 函数的返回值是 Iterator 对象 **方便多了。你可以用then方法指定下一步的操作。

   进一步说，**async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖 **

### 基本用法

```javascript
// 指定 50 毫秒以后，输出hello world
function timeout(ms) {
    return new Promise((resolve) => {
        setTimeout(resolve, ms);
    });
}
async function asyncPrint(value, ms) {
    await timeout(ms);
    console.log(value);
}
asyncPrint('hello world', 50);
```

* **async函数返回一个 Promise 对象，内部return语句返回的值，会成为then方法回调函数的参数。async函数内部抛出错误，会导致返回的 Promise 对象变为reject状态 **。抛出的错误对象会被catch方法回调函数接收到。

  await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到

* async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，**除非遇到return语句或者抛出错误 **。也就是说，只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数

* 正常情况下，await命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。

  另一种情况是，await命令后面是一个thenable对象（即定义then方法的对象），那么await会将其等同于 Promise 对象。

* **借助await命令就可以让程序停顿指定的时间**

  ```javascript
  function sleep(interval) {
      return new Promise(resolve => {
          setTimeout(resolve, interval);
      })
  }
  // 用法
  async function one2FiveInAsync() {
      for(let i = 1; i <= 5; i++) {
          console.log(i);
          await sleep(1000);
      }
  }
  one2FiveInAsync();
  ```

* **任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行 **。

  如果希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以**将第一个await放在try...catch结构里面 **，这样不管这个异步操作是否成功，第二个await都会执行。

  另一种方法是**await后面的 Promise 对象再跟一个catch方法 **，处理前面可能出现的错误

* **多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发**

* await命令只能用在async函数之中，如果用在普通函数，就会报错

  ```javascript
  async function dbFuc(db) {
      let docs = [{}, {}, {}];
      // 报错
      docs.forEach(function (doc) {
          await db.post(doc);
      });
  }
  
  //上面代码可能不会正常工作，原因是这时三个db.post操作将是并发执行，也就是同时执行，而不是继发执行。正确的写法是采用for循环。
  async function dbFuc(db) {
      let docs = [{}, {}, {}];
      for (let doc of docs) {
          await db.post(doc);
      }
  }
  
  // 另一种方法是使用数组的reduce方法
  // 下面reduce方法的第一个参数是async函数，导致该函数的第一个参数是前一步操作返回的 Promise 对象，所以必须使用await等待它操作结束。另外，reduce方法返回的是docs数组最后一个成员的async函数的执行结果，也是一个 Promise 对象，导致在它前面也必须加上await
  async function dbFuc(db) {
      let docs = [{}, {}, {}];
      await docs.reduce(async (_, doc) => {
          await _;
          await db.post(doc);
      }, undefined);
  }
  ```

* **希望多个请求并发执行，可以使用`Promise.all`方法**。当三个请求都会resolved时，下面两种写法效果相同

  ```javascript
  async function dbFuc(db) {
      let docs = [{}, {}, {}];
      let promises = docs.map((doc) => db.post(doc));
      let results = await Promise.all(promises);
      console.log(results);
  }
  // 或者使用下面的写法
  async function dbFuc(db) {
      let docs = [{}, {}, {}];
      let promises = docs.map((doc) => db.post(doc));
      let results = [];
      for (let promise of promises) {
          results.push(await promise);
      }
      console.log(results);
  }
  ```

* **async 函数可以保留运行堆栈**

  ```javascript
  const a = () => {
      b().then(() => c());
  };
  ```

  上面代码中，函数a内部运行了一个异步任务b()。当b()运行的时候，函数a()不会中断，而是继续执行。等到b()运行结束，可能a()早就运行结束了，b()所在的上下文环境已经消失了。如果b()或c()报错，错误堆栈将不包括a()。

  ```javascript
  const a = async () => {
      await b();
      c();
  };
  
  ```

  上面代码中，b()运行的时候，a()是暂停执行，上下文环境都保存着。一旦b()或c()报错，错误堆栈将包括a()。

### 实例：按顺序完成异步操作

**依次远程读取一组 URL，然后按照读取的顺序输出结果 **:

```javascript
async function logInOrder(urls) {
    for (const url of urls) {
        const response = await fetch(url);
        console.log(await response.text());
    }
}
```

上面代码确实大大简化，**问题是所有远程操作都是继发。只有前一个 URL 返回结果，才会去读取下一个 URL **，这样做效率很差，非常浪费时间。我们需要的是**并发发出远程请求 **。

```javascript
async function logInOrder(urls) {
    // 并发读取远程URL
    const textPromises = urls.map(async url => {
        const response = await fetch(url);
        return response.text();
    });
    // 按次序输出
    for (const textPromise of textPromises) {
        console.log(await textPromise);
    }
}
```

上面代码中，虽然map方法的参数是async函数，但它是并发执行的，因为只有async函数内部是继发执行，外部不受影响。后面的for..of循环内部使用了await，因此实现了按顺序输出。

### 顶层 await



### 总结

1. `async/await` 是 `Generator` 函数的语法糖,它将 `Generator` 函数的`星号*`替换成`async`，将`yield`替换成`await`。核心逻辑是迭代执行`next`函数。
2. `async` 函数返回一个`Promise` 对象
3. `async/await` 使用同步的方式去写异步代码
4. `Promise`通过`then`链来解决多层回调的问题，`async/await` 进一步优化它，使得代码结构更清晰。
