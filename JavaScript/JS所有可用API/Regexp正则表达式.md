# 正则表达式

1. **match():**
2. **replace():**
3. **search():**
4. **split():**

## ES6

### 1. matchAll()

​	返回一个正则表达式在当前字符串的所有匹配.

​	它返回的是一个遍历器（Iterator），而不是数组

```javascript
const string = 'test1test2test3';
// g 修饰符加不加都可以
const regex = /t(e)(st(\d?))/g;
for (const match of string.matchAll(regex)) {
    console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```
​	**(重要)**遍历器转为数组是非常简单的，使用 **...运算符 ** 和 **Array.from()** 方法就可以了

```javascript
// 转为数组方法一
[...string.matchAll(regex)]
// 转为数组方法二
Array.from(string.matchAll(regex))
```

