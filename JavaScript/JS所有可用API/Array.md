# Array



## ES6

### 1. Array.from()

将**类似数组的对象(有length属性)**和**可遍历（Set和Map）**的对象(部署了 Iterator 接口的数据结构)转为真正的数组

任何有length属性的对象，都可以通过Array.from方法转为数组，而此时扩展运算符就无法转换

```javascript
// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']
// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

常见的类似数组的对象是 DOM 操作返回的 **NodeList **集合，以及函数内部的**arguments**对象

* 可接受**第二个参数**，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组

  ```javascript
  // 第一个参数指定了第二个参数运行的次数
  Array.from({ length: 2 }, () => 'jack')
  // ['jack', 'jack']
  ```

* 如果map函数里面用到了this关键字，还可以传入Array.from的**第三个参数**，用来绑定this

### 2. Array.of()

用于将一组值，转换为数组

这个方法的主要目的，是弥补数组构造函数Array()的不足。因为参数个数的不同，会导致Array()的行为有差异

```javascript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

### 3. copyWithin(target, start = 0, end = this.length)

在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组

- target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
- start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
- end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

### 4.  find() 和 findIndex()

* **find(function(value, index, arr){}):** 用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员。如果没有符合条件的成员，则返回**undefined**。

  接受第二个参数，用来绑定回调函数的this对象

这两个方法都可以发现**NaN**，弥补了数组的**indexOf**方法的不足

```javascript
[NaN].indexOf(NaN)
// -1
[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

### 5. fill()

使用给定值，填充一个数组

还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置

如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象

```javascript
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]
let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```

### 6. entries()，keys() 和 values()

用于遍历数组，keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历,它们都**返回一个遍历器对象 ** 。

如果不使用for...of循环，可以手动调用遍历器对象的next方法，进行遍历：

```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```

### 7. includes()

**返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。 **用来替代indexOf()方法

**第二个参数 **表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。

```javascript
[NaN].includes(NaN)
// true
```

* Map 和 Set 数据结构有一个has方法，需要注意与includes区分。
  - Map 结构的has方法，是用来查找键名的，比如**Map.prototype.has(key)、WeakMap.prototype.has(key)、Reflect.has(target,     propertyKey) **。
  - Set 结构的has方法，是用来查找值的，比如**Set.prototype.has(value)、WeakSet.prototype.has(value) **。

### 8. flat(n)，flatMap()

* **flat():**变成一维的数组。该方法**返回一个新数组 **，对原数据没有影响。如果不管有多少层嵌套，都要转成一维数组，可以用**Infinity **关键字作为参数。

  **如果原数组有空位，flat()方法会跳过空位**：

  ```javascript
  [1, 2, , 4, 5].flat()
  // [1, 2, 4, 5]
  ```

* **flatMap(function(currentValue, index, array){}):** 对原数组的每个成员执行一个函数（相当于执行Array.prototype.map()），然后**对返回值组成的数组执行flat()方法,只能展开一层数组 **。该方法返回一个新数组，不改变原数组。

  还可以有**第二个参数，用来绑定遍历函数里面的this **

  