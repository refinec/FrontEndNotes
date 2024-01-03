## 1. 初始化数组

> 当`Array().fill()`使用在数据量大的时候，性能远不如写一个 for 循环往空数组 push

如果想要初始化一个指定长度的一维数组，并指定默认值，可以这样：

```javascript
const array = Array(6).fill(''); 
// ['', '', '', '', '', '']
```

如果想要初始化一个指定长度的二维数组，并指定默认值，可以这样：

```javascript
const matrix = Array(6).fill(0).map(() => Array(5).fill(0));
// 或者
const matrix = [...new Array(6)].map(() => new Array(6).fill(0));
// 或者
const matrix = [...Array(6)].map(() => Array(6).fill(0));
// [[0, 0, 0, 0, 0],
//  [0, 0, 0, 0, 0],
//  [0, 0, 0, 0, 0],
//  [0, 0, 0, 0, 0],
//  [0, 0, 0, 0, 0],
//  [0, 0, 0, 0, 0]]
```

三维

```js
[...Array(6)].map(() => [...Array(6)].map(() => Array(6).fill(0)))
```

## 2. 过滤错误值

过滤数组中的`false、0、''、null、undefined`等值，可以这样：

```javascript
const array = [1, 0, undefined, 6, 7, '', false];

array.filter(Boolean);
// [1, 6, 7]
```

## 3. 计算代码性能

1. 使用`performance.now()`

   ```js
   const startTime = performance.now(); 
   // 某些程序
   for(let i = 0; i < 1000; i++) {
       console.log(i)
   }
   const endTime = performance.now();
   const totaltime = endTime - startTime;
   console.log(totaltime); // 30.299999952316284
   ```

2. 使用`console.time `和`console.timeEnd`

   ```js
   console.time();
   // 某些程序
   for(let i = 0; i < 1000; i++) {
       console.log(i)
   }
   console.timeEnd();
   ```

## 4. 拼接数组(3种方式)

1. **使用扩展运算符**

   ```js
   const start = [1, 2] 
   const end = [5, 6, 7] 
   const numbers = [9, ...start, ...end, 8] // [9, 1, 2, 5, 6, 7 , 8]
   ```

2. **实用`Array`的`concat`方法**

   > 使用concat()方法时，如果需要合并的数组很大，那么concat() 函数会在创建单独的新数组时消耗大量内存。

   ```js
   const start = [1, 2, 3, 4] 
   const end = [5, 6, 7] 
   start.concat(end); // [1, 2, 3, 4, 5, 6, 7]
   ```

3. **使用`Array.prototype.push`**

   > 这种方式就能在很大程度上较少内存的使用

   ```js
   Array.prototype.push.apply(start, end)
   ```

## 5. 类数组转为数组

1. `Array.prototype.slice.call(arguments);`
2. `[...arguments]`

## 6. 缩短console.log()

```js
const c = console.log.bind(document) 
c(996) 
c("hello world")
```

## 7. 数字取整

1. **使用`~~`运算符来消除数字的小数部分**

   ```js
   ~~3.1415926  // 3
   ```

   其实这个运算符的作用有很多，通常是用来将变量转化为数字类型的，不同类型的转化结果不一样：

   - 如果是数字类型的字符串，就会转化为纯数字；
   - 如果字符串包含数字之外的值，就会转化为0；
   - 如果是布尔类型，**true会返回1**，**false会返回0**；

2. 使用**按位与**来实现数字的取整操作，只需要在数字后面加上`|0`即可：

   ```js
   23.9 | 0   // 23
   -23.9 | 0   // -23
   ```

3. 使用 `>>`

   ```js
   3.09 >> 0 // 3
   ```

## 8. 检查对象是否为空

```js
Object.keys({}).length  // 0
Object.keys({key: 1}).length  // 1
```

## 9. 获取数组中的最后一项

```js
arr.slice(-1)[0]
```

## 10. 打乱数组

使用`Math.random()`方法。

```js
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
list.sort(() => {
    return Math.random() - 0.5;
});
// 输出
(9) [2, 5, 1, 6, 9, 8, 4, 3, 7]
// Call it again
(9) [4, 1, 7, 5, 3, 8, 2, 9, 6]
```

## 11. 将十进制转换为二进制或十六进制

```js
const num = 10;

num.toString(2); // 输出: "1010"
num.toString(16); // 输出: "a"
num.toString(8); // 输出: "12"
```

## 12. 生成星级评分

```js
const StartScore = rate => "★★★★★☆☆☆☆☆".slice(5 - rate, 10 - rate);
const start = StartScore(3);
// start => '★★★☆☆'
```

## 13. 生成范围随机整数

```js
const RandomNum = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
const num = RandomNum(1, 10); // 5
```

## 14.检测数据类型

> 可确定的类型：undefined、null、string、number、boolean、array、object、symbol、date、regexp、function、asyncfunction、arguments、set、map、weakset、weakmap

```js
function DataType(tgt, type) {
    const dataType = Object.prototype.toString.call(tgt).replace(/\[object (\w+)\]/, "$1").toLowerCase();
    return type ? dataType === type : dataType;
}

DataType("test"); // "string"
DataType(20220314); // "number"
DataType(true); // "boolean"
DataType([], "array"); // true
DataType({}, "array"); // false
```

## 15. 克隆对象

```js
const _obj = {a: 0, b: 1, c: 2};
const obj = {..._obj};
const obj = JSON.parse(JSON.stringify(_obj));
// obj => { a: 0, b: 1, c: 2 }
```

## 16. 删除无用的对象属性

```js
const obj = {a: 0, b: 1, c: 2};
const {a, ...rest} = obj;
// rest => { b: 1, c: 2 }
```

## 17. 只运行一次的函数

```js
function Func() {
    console.log("x");
    Func = function () {
        console.log("y");
    }
}
```

## 18. 延迟加载函数

当函数中的复杂判断分支越来越多时，可以大大节省资源开销，可以使用闭包来实现。

```js
function Func() {
    if (a === b) {
        console.log("x");
    } else {
        console.log("y");
    }
}

// replace with
function Func() {
    if (a === b) {
        Func = function () {
            console.log("x");
        }
    } else {
        Func = function () {
            console.log("y");
        }
    }
    return Func();
}
```

## 19. 检测非空参数

```js
function IsRequired() {
    throw new Error("param is required");
}

function Func(name = IsRequired()) {
    console.log("I Love " + name);
}

Func(); // "param is required"
Func("You"); // "I Love You"
```

## 20. 字符串创建函数

```js
const Func = new Function("name", "console.log(\"I Love \" + name)");
```

## 21. 遇到错误时自动跳转`stackoverflow`查找该问题

```js
try {
    Func();
} catch (e) {
    location.href = "https://stackoverflow.com/search?q=[js]+" + e.message;
}
```

## 22. 优雅地处理 Async/Await 参数

```js
function AsyncTo(promise) {
    return promise.then(data => [null, data]).catch(err => [err]);
}

const [err, res] = await AsyncTo(Func());
```

# DOM技巧

## 1. 给所有DOM元素加上边框

```js
[].forEach.call($$("*"), dom => {
    dom.style.outline = "1px solid #" + (~~(Math.random() * (1 << 24))).toString(16);
});
```

## 2. 过滤XSS

```js
function FilterXss(content) {
    let elem = document.createElement("div");
    elem.innerText = content;
    const result = elem.innerHTML;
    elem = null;
    return result;
}
```

