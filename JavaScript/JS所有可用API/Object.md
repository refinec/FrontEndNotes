# Object



## ES6

### 1.Object.is()

**用来比较两个值是否严格相等 **

* 相等运算符（==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0

* 不同之处只有两个：一是**+0不等于-0 **，二是**NaN等于自身 **:

  ```javascript
  +0 === -0 //true
  NaN === NaN // false
  Object.is(+0, -0) // false
  Object.is(NaN, NaN) // true
  ```

### 2. Object.assign(target,source[,source....])

**用于对象的合并（浅拷贝），将源对象（source）的所有可枚举属性，复制到目标对象（target）.只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false） **。

* 如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性

* 属性名为 Symbol 值的属性，也会被`Object.assign`拷贝

  ```javascript
  Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
  // { a: 'b', Symbol(c): 'd' }
  ```

  `Object.assign`只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制

  ```javascript
  const source = {
      get foo() { return 1 }
  };
  const target = {};
  Object.assign(target, source)
  // { foo: 1 }
  ```

  **用途：**

  1. **为对象添加属性**

     ```javascript
     class Point {
         constructor(x, y) {
             Object.assign(this, {x, y});
         }
     }
     ```

  2. **为对象添加方法**

     ```javascript
     Object.assign(SomeClass.prototype, {
         someMethod(arg1, arg2) {
             ···
         },
         anotherMethod() {
             ···
         }
     });
     // 等同于下面的写法
     SomeClass.prototype.someMethod = function (arg1, arg2) {
         ···
     };
     SomeClass.prototype.anotherMethod = function () {
         ···
     };
     ```

  3. **克隆对象、合并多个对象**

### 3. Object.getOwnPropertyDescriptors()

ES5 的`Object.getOwnPropertyDescriptor()`方法会返回某个对象属性的描述对象（descriptor）

`Object.getOwnPropertyDescriptors()`方法，返回指定对象所有自身属性（非继承属性）的描述对象

* 该方法的引入目的，主要是为了解决`Object.assign()`无法正确拷贝get属性和set属性的问题

  ```javascript
  const source = {
      set foo(value) {
          console.log(value);
      }
  };
  const target2 = {};
  Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
  Object.getOwnPropertyDescriptor(target2, 'foo')
  // { get: undefined,
  //   set: [Function: set foo],
  //   enumerable: true,
  //   configurable: true }
  
  // 上面代码中，两个对象合并的逻辑可以写成一个函数。
  const shallowMerge = (target, source) => Object.defineProperties(
      target,
      Object.getOwnPropertyDescriptors(source)
  );
  ```

* 配合`Object.create()`方法，将对象属性克隆到一个新对象。这属于浅拷贝

  ```javascript
  const clone = Object.create(Object.getPrototypeOf(obj),
                              Object.getOwnPropertyDescriptors(obj));
  // 或者
  const shallowClone = (obj) => Object.create(
      Object.getPrototypeOf(obj),
      Object.getOwnPropertyDescriptors(obj)
  );
  ```

### 4. __proto__属性，Object.setPrototypeOf()，Object.getPrototypeOf()

* __proto__属性（前后各两个下划线），用来读取或设置当前对象的prototype对象

* **Object.setPrototypeOf()** 设置一个对象的prototype对象，返回参数对象本身

  ```javascript
  // 格式
  Object.setPrototypeOf(object, prototype)
  // 用法
  const o = Object.setPrototypeOf({}, null);
  该方法等同于下面的函数。
  function setPrototypeOf(obj, proto) {
      obj.__proto__ = proto;
      return obj;
  }
  ```

### 5. Object.keys()，Object.values()，Object.entries()

1. **Object.keys():** 返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名
2. **Object.values():** 返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值

3. **Object.entries():**方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组

   ```javascript
   // 将对象转为真正的Map结构
   const obj = { foo: 'bar', baz: 42 };
   const map = new Map(Object.entries(obj));
   map // Map { foo: "bar", baz: 42 }
   ```

### 6. (重要)Object.fromEntries()

是`Object.entries()`的逆操作，用于**将一个键值对数组转为对象 **

```javascript
Object.fromEntries([
    ['foo', 'bar'],
    ['baz', 42]
])
// { foo: "bar", baz: 42 }
```

将键值对的数据结构还原为对象，因此特别适合**将 Map 结构转为对象 **

```javascript
// 例一
const entries = new Map([
    ['foo', 'bar'],
    ['baz', 42]
]);
Object.fromEntries(entries)
// { foo: "bar", baz: 42 }
// 例二
const map = new Map().set('foo', true).set('bar', false);
Object.fromEntries(map)
// { foo: true, bar: false }
```

配合`URLSearchParams`对象，**将查询字符串转为对象 **

```javascript
Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
// { foo: "bar", baz: "qux" }
```

