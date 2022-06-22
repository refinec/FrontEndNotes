# Iterator 和 for...of 循环

## Iterator（遍历器）概念

目前表示“**集合**”的数据结构有**`数组Array`**、**`对象Object`**，**`Map`**和**`Set`**。

`Iterator`为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

下面是一个模拟next方法返回值的例子:

```js
function makeIterator(array) {
    var nextIndex = 0;
    return {
        next: function() {
            return nextIndex < array.length ?
                {value: array[nextIndex++], done: false} :
            {value: undefined, done: true};
        }
    };
}
var it = makeIterator(['a', 'b']);
it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }
```

如果使用 TypeScript 的写法，遍历器接口（Iterable）、指针对象（Iterator）和next方法返回值的规格可以描述如下

```ts
interface Iterable {
    [Symbol.iterator]() : Iterator,
}
interface Iterator {
    next(value?: any) : IterationResult,
}
interface IterationResult {
    value: any,
    done: boolean,
}
```

## 默认 Iterator 接口

**原生具备 Iterator 接口的数据结构如下**:

1. `Set`
2. `Map `
3. `Array `
4. `String`
5. `TypedArray`
6. 函数的 `arguments` 对象
7. `NodeList` 对象

```js
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();
iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

默认的 `Iterator` 接口部署在数据结构的Symbol.iterator属性上，凡是部署了Symbol.iterator属性的数据结构，就称为部署了遍历器接口。**调用这个接口，就会返回一个遍历器对象**。

`Symbol.iterator`属性本身是一个函数，是当前数据结构默认的遍历器生成函数。

属性名Symbol.iterator，<u>它是一个表达式，返回Symbol对象的iterator属性</u>，这是一个预定义好的、类型为 Symbol 的特殊值，所以要放在方括号内

```js
const obj = {
    [Symbol.iterator] : function () {
        return {
            next: function () {
                return {
                    value: 1,
                    done: true
                };
            }
        };
    }
};
```

**注意：** Iterator 接口的数据结构，不用自己写遍历器生成函数，for...of循环会自动遍历它们。而其他数据结构（主要是对象）的 Iterator 接口，都需要自己在Symbol.iterator属性上面部署，这样才会被for...of循环遍历。

* 一个对象如果要具备可被for...of循环调用的 Iterator 接口，就必须在Symbol.iterator的属性上部署遍历器生成方法（原型链上的对象具有该方法也可）

  ```js
  class RangeIterator {
      constructor(start, stop) {
          this.value = start;
          this.stop = stop;
      }
      [Symbol.iterator]() { return this; } // 执行后返回当前对象的遍历器对象
      next() {
          var value = this.value;
          if (value < this.stop) {
              this.value++;
              return {done: false, value: value};
          }
          return {done: true, value: undefined};
      }
  }
  function range(start, stop) {
      return new RangeIterator(start, stop);
  }
  for (var value of range(0, 3)) {
      console.log(value); // 0, 1, 2
  }
  ```

  ```js
  # 下面代码首先在构造函数的原型链上部署Symbol.iterator方法，调用该方法会返回遍历器对象iterator，调用该对象的next方法，在返回一个值的同时，自动将内部指针移到下一个实例
  function Obj(value) {
      this.value = value;
      this.next = null;
  }
  Obj.prototype[Symbol.iterator] = function() {
      var iterator = { next: next };
      var current = this;
      function next() {
          if (current) {
              var value = current.value;
              current = current.next;
              return { done: false, value: value };
          } else {
              return { done: true };
          }
      }
      return iterator;
  }
  var one = new Obj(1);
  var two = new Obj(2);
  var three = new Obj(3);
  one.next = two;
  two.next = three;
  for (var i of one){
      console.log(i); // 1, 2, 3
  }
  ```

  ```js
  let obj = {
      data: [ 'hello', 'world' ],
      [Symbol.iterator]() {
          const self = this;
          let index = 0;
          return {
              next() {
                  if (index < self.data.length) {
                      return {
                          value: self.data[index++],
                          done: false
                      };
                  } else {
                      return { value: undefined, done: true };
                  }
              }
          };
      }
  };
  ```

* **对于类似数组的对象（存在数值键名和length属性），部署 Iterator 接口，有一个简便方法，就是Symbol.iterator方法直接引用数组的 Iterator 接口**

  ```js
  NodeList.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
  // 或者
  NodeList.prototype[Symbol.iterator] = [][Symbol.iterator];
  [...document.querySelectorAll('div')] // 可以执行了
  
  let iterable = {
      0: 'a',
      1: 'b',
      2: 'c',
      length: 3,
      [Symbol.iterator]: Array.prototype[Symbol.iterator] // 类数组对象调用数组的Symbol.iterator方法
  };
  for (let item of iterable) {
      console.log(item); // 'a', 'b', 'c'
  }
  ```

## 调用 Iterator 接口的场合

1. **解构赋值**

   对数组和 Set 结构进行解构赋值时，会默认调用Symbol.iterator方法

   ```js
   let set = new Set().add('a').add('b').add('c');
   let [x,y] = set;
   // x='a'; y='b'
   let [first, ...rest] = set;
   // first='a'; rest=['b','c'];
   ```

2. **...扩展运算符**

   ...扩展运算符也会调用默认的 Iterator 接口。这提供了一种简便机制，可以将任何部署了 Iterator 接口的数据结构，转为数组。

   ```js
   // 例一
   var str = 'hello';
   [...str] //  ['h','e','l','l','o']
   // 例二
   let arr = ['b', 'c'];
   ['a', ...arr, 'd']
   // ['a', 'b', 'c', 'd']
   ```

3. **yield***

   `yield*`后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口

   ```js
   let generator = function* () {
       yield 1;
       yield* [2,3,4];
       yield 5;
   };
   var iterator = generator();
   iterator.next() // { value: 1, done: false }
   iterator.next() // { value: 2, done: false }
   iterator.next() // { value: 3, done: false }
   iterator.next() // { value: 4, done: false }
   iterator.next() // { value: 5, done: false }
   iterator.next() // { value: undefined, done: true }
   ```

4. **其他场合**

   由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。

   - for...of
   - Array.from()
   - Map(), Set(),     WeakMap(), WeakSet()（比如new     Map([['a',1],['b',2]])）
   - Promise.all()
   - Promise.race()

## 字符串的 Iterator 接口

字符串是一个类似数组的对象，也原生具有 Iterator 接口

```js
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"
var iterator = someString[Symbol.iterator]();
iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

覆盖原生的Symbol.iterator方法，达到修改遍历器行为的目的

```js
var str = new String("hi");
[...str] // ["h", "i"]
str[Symbol.iterator] = function() {
    return {
        next: function() {
            if (this._first) {
                this._first = false;
                return { value: "bye", done: false };
            } else {
                return { done: true };
            }
        },
        _first: true
    };
};
[...str] // ["bye"]
str // "hi"
```

## Iterator 接口与 Generator 函数

```js
let myIterable = {
    [Symbol.iterator]: function* () {
        yield 1;
        yield 2;
        yield 3;
    }
}
[...myIterable] // [1, 2, 3]
// 或者采用下面的简洁写法
let obj = {
    * [Symbol.iterator]() {
        yield 'hello';
        yield 'world';
    }
};
for (let x of obj) {
    console.log(x);
}
// "hello"
// "world"
```

## 遍历器对象的 return()，throw()：场景意外中断遍历

遍历器对象除了具有next方法，还可以具有return方法和throw方法。如果你自己写遍历器对象生成函数，那么next方法是必须部署的，return方法和throw方法是否部署是可选的。

**return方法的使用场合**是，如果**for...of循环提前退出（通常是因为出错，或者有break语句）**，就会调用return方法。如果一个对象在完成遍历前，**需要清理或释放资源，就可以部署return方法**。

```js
function readLinesSync(file) {
    return {
        [Symbol.iterator]() {
            return {
                next() {
                    return { done: false };
                },
                return() {
                    file.close();
                    return { done: true };
                }
            };
        },
    };
}
```

**下面的两种情况，都会触发执行return方法**

```js
// 情况一
for (let line of readLinesSync(fileName)) {
    console.log(line);
    break;
}
// 情况二
for (let line of readLinesSync(fileName)) {
    console.log(line);
    throw new Error();
}
```

**注意**

* **return方法必须返回一个对象**，这是 Generator 规格决定的。

* **throw方法主要是配合 Generator 函数使用**，一般的遍历器对象用不到这个方法

## for...of 循环、for...in循环

**for...in循环有几个缺点**：

- 数组的键名是数字，但是for...in循环是**以字符串作为键名“0”、“1”、“2”**等等。

- for...in循环不仅遍历数字键名，**还会遍历手动添加的其他键，甚至包括原型链上的键**。

- 某些情况下，for...in循环会**以任意顺序遍历键名**。

总之，**for...in循环主要是为遍历对象而设计的，不适用于遍历数组**。

**for...of循环可以使用的范围包括**:

* 数组
* Set 和 Map 结构
* 某些类似数组的对象（比如arguments对象、DOM NodeList 对象）
*  Generator 对象
* 字符串

**`for...of`不同于forEach方法，它可以与break、continue和return配合使用，而数组的forEach方法使用break时无法跳出终止循环。**

for...of循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。这一点跟for...in循环也不一样

```js
let arr = [3, 5, 7];
arr.foo = 'hello';
for (let i in arr) {
    console.log(i); // "0", "1", "2", "foo"
}
for (let i of arr) {
    console.log(i); //  "3", "5", "7"
}
```

Set 和 Map 结构也原生具有 Iterator 接口，可以直接使用for...of循环

```js
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
    console.log(e);
}
// Gecko
// Trident
// Webkit
var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
    console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

对于普通的对象，for...of结构不能直接使用，会报错，必须部署了 Iterator 接口后才能使用。但是，这样情况下，**for...in循环依然可以用来遍历键名**

```js
let es6 = {
    edition: 6,
    committee: "TC39",
    standard: "ECMA-262"
};
for (let e in es6) {
    console.log(e);
}
// edition
// committee
// standard
for (let e of es6) {
    console.log(e);
}
// TypeError: es6[Symbol.iterator] is not a function
```

一种**解决方法是**，使用Object.keys方法将对象的键名生成一个数组，然后遍历这个数组:

```js
for (var key of Object.keys(someObject)) {
    console.log(key + ': ' + someObject[key]);
}
```

另一个方法是使用 Generator 函数将对象重新包装一下

```js
function* entries(obj) {
    for (let key of Object.keys(obj)) {
        yield [key, obj[key]];
    }
}
for (let [key, value] of entries(obj)) {
    console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```



## **计算生成的数据结构**

有些数据结构是在现有数据结构的基础上，计算生成的。比如，**ES6 的数组、Set、Map** 都部署了以下三个方法，调用后都返回遍历器对象。

- **`entries()`** 返回一个遍历器对象，用来遍历[键名, 键值]组成的数组。对于数组，键名就是索引值；对于 Set，键名与键值相同。Map 结构的 Iterator 接口，默认就是调用entries方法。
- **`keys()`** 返回一个遍历器对象，用来遍历所有的键名。
- **`values()`** 返回一个遍历器对象，用来遍历所有的键值。

这三个方法调用后生成的遍历器对象，所遍历的都是计算生成的数据结构。

```js
let arr = ['a', 'b', 'c'];
 for (let pair of arr.entries()) {
  console.log(pair);
 }
 // [0, 'a']
 // [1, 'b']
 // [2, 'c']
```

