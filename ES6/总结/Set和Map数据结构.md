# Set 和 Map 数据结构



## 总结

### 1. Set

* Set函数可以接受**一个数组、类数组 （或者具有 iterable 接口的其他数据结构）**作为参数，用来初始化

* 成员的值都是唯一的，**没有重复的值**

  ```javascript
  // 去除数组的重复成员
  [...new Set(array)]
  
  // 去除字符串里面的重复字符
  [...new Set('ababbc')].join('')
  // "abc"
  ```

* Set本身是一个构造函数，用来生成 Set 数据结构

* 如果想在遍历操作中，同步改变原来的 Set 结构，目前没有直接的方法，但有两种变通方法。

  * 一种是利用原 Set 结构映射出一个新的结构，然后赋值给原来的 Set 结构；
  
  * 另一种是利用**`Array.from`**方法
  
  ```javascript
  // 方法一
  let set = new Set([1, 2, 3]);
  set = new Set([...set].map(val => val * 2));
  // set的值是2, 4, 6
  // 方法二
  let set = new Set([1, 2, 3]);
  set = new Set(Array.from(set, val => val * 2));
  // set的值是2, 4, 6
  ```

#### Set 实例的属性和方法

1. **Set.prototype.constructor**：构造函数，默认就是Set函数。
2. **Set.prototype.`size`**：返回Set实例的成员总数。

3. **Set.prototype.`add(value)`**：添加某个值，返回 Set 结构本身。
4. **Set.prototype.`delete(value)`**：删除某个值，返回一个布尔值，表示删除是否成功。
5. **Set.prototype.`has(value)`**：返回一个布尔值，表示该值是否为Set的成员。
6. **Set.prototype.`clear()`**：清除所有成员，没有返回值。

#### 遍历操作

1. **Set.prototype.`keys()`**：返回键名的**遍历器**
2. **Set.prototype.`values()`**：返回键值的遍历器
3. **Set.prototype.`entries()`**：返回键值对的遍历器
4. **Set.prototype.`forEach()`**：使用回调函数遍历每个成员

由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以keys方法和values方法的行为完全一致。

### 2.WeakSet

* 与 Set 类似，也是不重复的值的集合。但是，**WeakSet 的成员只能是对象,接受一个数组或类似数组的对象（具有 Iterable 接口）作为参数**，而不能是其他类型的值。注意，是参数数组的成员成为 WeakSet 的成员，而不是参数数组本身。这意味着，**参数数组的成员只能是对象**

* WeakSet 中的对象都是 **弱引用**，即垃圾回收机制不考虑 WeakSet 对该对象的引用.也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于WeakSet 之中

* **WeakSet 没有size属性，不可遍历**

#### 方法

- **WeakSet.prototype.`add(value)`**：向 WeakSet 实例添加一个新成员。
- **WeakSet.prototype.`delete(value)`**：清除 WeakSet 实例的指定成员。
- **WeakSet.prototype.`has(value)`**：返回一个布尔值，表示某个值是否在 WeakSet 实例之中。

#### 作用：

​		**储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏 **

### 3. Map

Object本质上是**键值对的集合（Hash 结构）**，但是**传统上只能用字符串当作键**。这给它的使用带来了很大的限制

```javascript
const data = {};
const element = document.getElementById('myDiv');
data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

Map 数据结构。**它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应**，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。

* Map 也可以**接受一个数组作为参数，该数组的成员是一个个表示键值对的数组**

  ```javascript
  const map = new Map([
      ['name', '张三'],
      ['title', 'Author']
  ]);
  map.size // 2
  map.has('name') // true
  map.get('name') // "张三"
  map.has('title') // true
  map.get('title') // "Author"
  ```

* 不仅仅是数组，**任何具有 Iterator 接口**、且**每个成员都是一个双元素的数组**的数据结构都可以当作Map构造函数的参数。这就是说，<u>**Set**和**Map**都可以用来生成新的 Map</u>

  ```javascript
  const set = new Set([
      ['foo', 1],
      ['bar', 2]
  ]);
  const m1 = new Map(set);
  m1.get('foo') // 1
  const m2 = new Map([['baz', 3]]);
  const m3 = new Map(m2);
  m3.get('baz') // 3
  ```

* 只有对同一个对象的引用，Map 结构才将其视为同一个键

  ```javascript
  const map = new Map();
  map.set(['a'], 555);
  map.get(['a']) // undefined
  ```

* 一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键

  * **`0`和`-0`就是一个键**
  * **`布尔值true`和`字符串true`则是两个不同的键**
  * **`undefined`和`null`也是两个不同的键**
  * **`NaN`不严格相等于自身，但 Map 将其视为同一个键**

  ```javascript
  let map = new Map();
  map.set(-0, 123);
  map.get(+0) // 123
  map.set(true, 1);
  map.set('true', 2);
  map.get(true) // 1
  map.set(undefined, 3);
  map.set(null, 4);
  map.get(undefined) // 3
  map.set(NaN, 123);
  map.get(NaN) // 123
  ```

* **结合数组的forEach方法、map方法、filter方法，可以实现 Map 的遍历和过滤（Map 本身没有map和filter方法），forEach方法还可以接受第二个参数，用来绑定this**

#### 实例的属性和操作方法

- **`size`属性**返回 Map 结构的成员总数。

- **Map.prototype.`set(key, value)`**方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
- **Map.prototype.`get(key)`**方法读取key对应的键值，如果找不到key，返回undefined。
- **Map.prototype.`has(key)`**方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
- **Map.prototype.`delete(key)`**方法删除某个键，返回true。如果删除失败，返回false。
- **Map.prototype.`clear()`**方法清除所有成员，没有返回值

#### 遍历方法

- `Map.prototype.keys()`：返回键名的遍历器。
- `Map.prototype.values()`：返回键值的遍历器。
- `Map.prototype.entries()`：返回所有成员的遍历器。
- `Map.prototype.forEach()`：遍历 Map 的所有成员

#### 与其他数据结构的互相转换

##### 1. Map 转为数组

​	使用扩展运算符`...`

##### 2. **数组 转为** **Map**

​	将数组传入 Map 构造函数，就可以转为 Map。但**至少是二维数组 **，每个元素数组包含2个元素

##### 3.**Map** 转为对象(使用`for..of`)

​	如果所有 Map 的键都是字符串，它可以无损地转为对象。如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。

```javascript
function strMapToObj(strMap) {
    let obj = Object.create(null);
    for (let [k,v] of strMap) {
        obj[k] = v;
    }
    return obj;
}
const myMap = new Map()
.set('yes', true)
.set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```
##### 4. **对象转为** Map(使用`Object.entries()`)

对象转为 Map 可以通过`Object.entries()`

```javascript
let obj = {"a":1, "b":2};
let map = new Map(Object.entries(obj));

function objToStrMap(obj) {
    let strMap = new Map();
    for (let k of Object.keys(obj)) {
        strMap.set(k, obj[k]);
    }
    return strMap;
}
objToStrMap({yes: true, no: false})
// Map {"yes" => true, "no" => false}
```

##### 5. **Map** **转为** **JSON**

* Map 转为 JSON 要区分两种情况。一种情况是，Map 的键名都是字符串，这时可以选择转为对象 JSON

  ```javascript
  function strMapToJson(strMap) {
      return JSON.stringify(strMapToObj(strMap));
  }
  let myMap = new Map().set('yes', true).set('no', false);
  strMapToJson(myMap)
  // '{"yes":true,"no":false}'
  ```

* Map 的键名有非字符串，这时可以选择转为数组 JSON

  ```javascript
  function mapToArrayJson(map) {
      return JSON.stringify([...map]);
  }
  let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
  mapToArrayJson(myMap)
  // '[[true,7],[{"foo":3},["abc"]]]'
  ```

##### 6. **JSON** **转为** **Map**

* JSON 转为 Map，正常情况下，所有键名都是字符串

  ```javascript
  function jsonToStrMap(jsonStr) {
      return objToStrMap(JSON.parse(jsonStr));
  }
  jsonToStrMap('{"yes": true, "no": false}')
  // Map {'yes' => true, 'no' => false}
  ```

* 整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作

  ```javascript
  function jsonToMap(jsonStr) {
      return new Map(JSON.parse(jsonStr));
  }
  jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
  // Map {true => 7, Object {foo: 3} => ['abc']}
  ```

### 4. WeakMap

#### WeakMap与Map的区别有两点

1. WeakMap**只接受对象作为键名（null除外）**，不接受其他类型的值作为键名

2. WeakMap的**键名所指向的对象，不计入垃圾回收机制**

   WeakMap的设计目的在于，有时我们想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用

   ```javascript
   const e1 = document.getElementById('foo');
   const e2 = document.getElementById('bar');
   const arr = [
       [e1, 'foo 元素'],
       [e2, 'bar 元素'],
   ];
   
   // e1和e2是两个对象，我们通过arr数组对这两个对象添加一些文字说明。这就形成了arr对e1和e2的引用
   // 一旦不再需要这两个对象，我们就必须手动删除这个引用，否则垃圾回收机制就不会释放e1和e2占用的内存。一旦忘了写，就会造成内存泄露
   arr [0] = null;
   arr [1] = null;
   ```

   WeakMap 就是为了解决这个问题而诞生的，它的**键名所引用的对象都是弱引用**，**键值依然是正常引用**，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。
   
   基本上，如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。

#### **WeakMap** **的用途**

1. **应用的典型场合就是 DOM 节点作为键名，添加数据**

   下面代码中，`document.getElementById('logo')`是一个 DOM 节点，每当发生click事件，就更新一下状态。我们将这个状态作为键值放在 WeakMap 里，对应的键名就是这个节点对象。一旦这个 DOM 节点删除，该状态就会自动消失，不存在内存泄漏风险

   ```javascript
   let myWeakmap = new WeakMap();
   myWeakmap.set(
       document.getElementById('logo'),
       {timesClicked: 0})
   ;
   document.getElementById('logo').addEventListener('click', function() {
       let logoData = myWeakmap.get(document.getElementById('logo'));
       logoData.timesClicked++;
   }, false);
   ```

2. **部署私有属性**

   下面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏

   ```javascript
   const _counter = new WeakMap();
   const _action = new WeakMap();
   class Countdown {
       constructor(counter, action) {
           _counter.set(this, counter);
           _action.set(this, action);
       }
       dec() {
           let counter = _counter.get(this);
           if (counter < 1) return;
           counter--;
           _counter.set(this, counter);
           if (counter === 0) {
               _action.get(this)();
           }
       }
   }
   const c = new Countdown(2, () => console.log('DONE'));
   c.dec()
   c.dec()
   // DONE
   ```

   