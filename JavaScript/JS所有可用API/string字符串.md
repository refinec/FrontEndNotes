# 字符串

#### 1. String.fromCharCode()

​	用于从`\u0000~\uFFFF`f范围内 Unicode 码点返回对应字符

#### 2. charAt()、charCodeAt()



## ES6

##### 1. String.fromCodePoint()

​	可以识别大于`0xFFFF`的字符 

##### 2. String.raw()

​	专用于模板字符串的标签函数

##### 3. codePointAt()

​	正确处理 4 个字节储存的字符，返回一个字符的码点

##### 4. normalize()

​	将字符的不同表示方法统一为同样的形式

    ```javascript
'\u01D1'==='\u004F\u030C' //false
'\u01D1'.normalize() === '\u004F\u030C'.normalize() // true
    ```

##### 5. **includes(),startsWith(), endsWith()：** 

​	参数一是参数字符串。支持第二个参数，表示开始搜索的位置

##### 6. repeat()

​	repeat方法**返回一个新字符串 **，表示将原字符串重复n次

##### 7. padStart()，padEnd()

​	字符串补全长度.第一参数是字符串补全生效的最大长度 ，第二参数是用来补全的字符串

##### 8. trimStart()，trimEnd()

​	消除头部、尾部的空格、tab 键、换行符等不可见的空白符，它们都**返回新字符串 **，不会修改原始字符串

##### 9. String.raw() 

​	专用于模板字符串的标签函数.该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。

    ```javascript
String.raw`Hi\n${2+3}!`
// 实际返回 "Hi\\n5!"，显示的是转义后的结果 "Hi\n5!"
String.raw`Hi\u000A!`;
// 实际返回 "Hi\\u000A!"，显示的是转义后的结果 "Hi\u000A!"

String.raw`Hi\\n`
// 返回 "Hi\\\\n"
String.raw`Hi\\n` === "Hi\\\\n" 
// true
    ```

