### Promise对象的两个特点

1. **对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）**

2. **一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。**

   如果某些事件不断地反复发生，一般来说，使用 [Stream](https://nodejs.org/api/stream.html) 模式是比部署Promise更好的选择

### 基本用法

1. **promise内是同步，then内是异步 **

2. **reject函数的参数通常是Error对象的实例**，表示抛出的错误；**resolve函数的参数除了正常的值以外，还可能是另一个 Promise 实例**

3. **调用resolve或reject并不会终结 Promise 内语句的执行，并且同步语句会先于resolve或reject执行**，因为立即 resolved 的 **Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务**。所以，最好**在它们前面加上return语句**，这样就不会有意外。

   ```js
   new Promise((resolve, reject) =>{
       console.log(1)
       resolve(2)
       console.log(3)
   }).then(d=>{console.log(d)})
   // 输出：1 3 2
   ```

### Promise实例方法

#### 1. Promise.prototype.then()

#### 2. Promise.prototype.catch()

> 跟传统的`try/catch`代码块不同的是，**如果没有使用`catch`方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码**，即不会有任何反应。这就是说，**Promise 内部的错误不会影响到 Promise 外部的代码**，通俗的说法就是“Promise 会吃掉错误”。

不过，**Node** 有一个**`unhandledRejection `**事件，专门监听未捕获的reject错误，上面的脚本会触发这个事件的监听函数，可以在监听函数里面抛出错误。

```javascript
// unhandledRejection事件的监听函数有两个参数，第一个是错误对象，第二个是报错的 Promise 实例，它可以用来了解发生错误的环境信息
process.on('unhandledRejection', function (err, p) {
    throw err;
});
```

catch方法返回的还是一个 Promise 对象，因此后面还可以接着调用then方法.

#### 3.Promise.prototype.finally()

finally方法的回调函数不接受任何参数，用于指定不管 Promise 对象最后状态如何，都会执行的操作。

```javascript
promise
    .then(result => {···})
    .catch(error => {···})
    .finally(() => {···});
```

### Promise 类方法

#### 1. Promise.all()

> 用于将多个 Promise 实例，包装成一个新的 Promise 实例

* **Promise.all()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例,如果不是，就会先调用Promise.resolve方法，将参数转为 Promise 实例 **

* Promise.all()的状态由各个Promise 实例决定，分成两种情况。

  （1）只有各个Promise 实例的状态都变成fulfilled，Promise.all()的状态才会变成fulfilled，此时各个Promise 实例的返回值组成一个数组，传递给Promise.all()的回调函数。

  （2）只要各个Promise 实例之中有一个被rejected，Promise.all()的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给Promise.all()的回调函数。

  ```javascript
  const databasePromise = connectDatabase();
  const booksPromise = databasePromise
  .then(findAllBooks);
  const userPromise = databasePromise
  .then(getCurrentUser);
  Promise.all([
      booksPromise,
      userPromise
  ])
      .then(([books, user]) => pickTopRecommendations(books, user));
  ```

**注意：**

下面代码中，**p1会resolved，p2首先会rejected **，但是**p2有自己的catch方法 **，该方法返回的是一个新的 Promise 实例，p2指向的实际上是这个实例。<u>该实例执行完catch方法后，也会变成resolved</u>，导致

**Promise.all()方法参数里面的两个实例都会resolved **，因此会调用then方法指定的回调函数，而不会调用catch方法指定的回调函数。如果p2没有自己的catch方法，就会调用Promise.all()的catch方法

```javascript
const p1 = new Promise((resolve, reject) => {
    resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
    throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
    .then(result => console.log(result))
    .catch(e => console.log(e));
// ["hello", Error: 报错了]
```

#### 2.Promise.race()

> 将多个 Promise 实例，包装成一个新的 Promise 实例。
>
> 只要**各个Promise 实例之中有一个实例率先改变状态(不管状态时`fulfilled`还是`rejected`)，`Promise.race()`的状态就跟着改变，那个率先改变的 Promise 实例的返回值，就传递给p的回调函数**

* `Promise.race()`方法的参数与`Promise.all()`方法一样，如果不是 Promise 实例，就会先调用下面讲到的`Promise.resolve()`方法，将参数转为 Promise 实例，再进一步处理

```js
const p1 = new Promise((resolve, reject) => {
    resolve('hello');
})

const p2 = new Promise((resolve, reject) => {
    throw new Error('报错了');
})

Promise.race([p1, p2])
    .then(result => console.log(result))
    .catch(e => console.log(e));
// 'hello'
```

#### 3. Promise.allSettled()

> 接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。
>
> 只有等到所有这些参数实例都返回结果，不管是`fulfilled`还是`rejected`，包装实例才会结束。

**该方法返回的新的 Promise 实例，一旦结束，状态总是`fulfilled `**，不会变成`rejected`。状态变成`fulfilled`后，Promise 的监听函数接收到的参数是一个数组，每个成员对应一个传入`Promise.allSettled()`的 Promise 实例

```javascript
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);
const allSettledPromise = Promise.allSettled([resolved, rejected]);
allSettledPromise.then(function (results) {
    console.log(results);
});
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```

#### 4. Promise.any()

> 接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。
>
> **只要参数实例有一个变成`fulfilled`状态 **，包装实例就会变成`fulfilled`状态；如果所有参数实例都变成`rejected`状态，包装实例就会变成`rejected`状态

**`Promise.any()`跟`Promise.race()`方法很像，只有一点不同，就是不会因为某个 Promise 变成rejected状态而结束**

#### 5. Promise.resolve()

将现有对象转为 Promise 对象，Promise.resolve()方法就起到这个作用

```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

* **参数是一个** **Promise** **实例**

  如果参数是 Promise 实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例

* **参数是一个**thenable**对象**

  thenable对象指的是具有then方法的对象，比如下面这个对象。Promise.resolve方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then方法

  ```javascript
  let thenable = {
      then: function(resolve, reject) {
          resolve(42);
      }
  };
  ```

* **参数不是具有then方法的对象，或根本就不是对象**

  返回一个新的 Promise 对象，状态为resolved。

  返回 Promise 实例的状态从一生成就是resolved，所以回调函数会立即执行。Promise.resolve方法的参数，会同时传给回调函数。

* **不带有任何参数**

  > 直接返回一个resolved状态的 Promise 对象

  需要注意的是，立即resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。下面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log('one')则是立即执行，因此最先输出。

  ```javascript
  setTimeout(function () {
      console.log('three');
  }, 0);
  Promise.resolve().then(function () {
      console.log('two');
  });
  console.log('one');
  // one
  // two
  // three
  ```

#### 6. Promise.reject()

> 返回一个新的 Promise 实例，该实例的状态为rejected
>

​	Promise.reject()方法的参数，**会原封不动地作为reject的理由 **，变成后续方法的参数。这一点与Promise.resolve方法不一致

```javascript
const thenable = {
    then(resolve, reject) {
        reject('出错了');
    }
};
Promise.reject(thenable)
    .catch(e => {
    console.log(e === thenable)
})
// true
```

上面代码中，Promise.reject方法的参数是一个thenable对象，执行以后，后面catch方法的参数不是reject抛出的“出错了”这个字符串，而是thenable对象

#### 7. Promise.try()

不知道或者不想区分，函数`f`是同步函数还是异步操作，但是想用 Promise 来处理它。因为这样就可以不管f是否包含异步操作，都用then方法指定下一步流程，用catch方法处理f抛出的错误。一般就会采用下面的写法。

**(重要)** **`new Promise()`和 `async`函数让同步函数同步执行，异步函数异步执行，并且让它们具有统一的 API** 

```javascript
// 因此如果f是同步的，就会得到同步的结果；如果f是异步的，就可以用then指定下一步，就像下面的写法。
// 需要注意的是，async () => f()会吃掉f()抛出的错误。所以，如果想捕获错误，要使用promise.catch方法
const f = () => console.log('now');
(async () => f())();
console.log('next');
// now
// next


const f = () => console.log('now');
(
    () => new Promise(
        resolve => resolve(f())
    )
)();
console.log('next');
// now
// next

```

提供Promise.try方法替代上面的写法

```javascript
const f = () => console.log('now');
Promise.try(f);
console.log('next');
// now
// next
```



### 应用

#### **加载图片**

将图片的加载写成一个Promise，一旦加载完成，Promise的状态就发生变化.

#### Generator 函数与 Promise 的结合

使用 Generator 函数管理流程，遇到异步操作的时候，通常返回一个Promise对象

```javascript
// Generator 函数g之中，有一个异步操作getFoo，它返回的就是一个Promise对象。函数run用来处理这个Promise对象，并调用下一个next方法

function getFoo () {
    return new Promise(function (resolve, reject){
        resolve('foo');
    });
}
const g = function* () {
    try {
        const foo = yield getFoo();
        console.log(foo);
    } catch (e) {
        console.log(e);
    }
};
function run (generator) {
    const it = generator();
    function go(result) {
        if (result.done) return result.value;
        return result.value.then(function (value) {
            return go(it.next(value));
        }, function (error) {
            return go(it.throw(error));
        });
    }
    go(it.next());
}
run(g);
```

