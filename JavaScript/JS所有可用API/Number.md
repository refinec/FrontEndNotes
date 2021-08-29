# Number



## ES6

### 1. Number.isFinite(), Number.isNaN()

* `Number.isFinite()`用来**检查一个数值是否为有限的 **（finite），即不是Infinity。参数类型不是数值，Number.isFinite一律返回false。

* `Number.isNaN()`用来**检查一个值是否为NaN **,参数类型不是NaN，Number.isNaN一律返回false

### 2. Number.parseInt(), Number.parseFloat()

​	将全局方法`parseInt()`和`parseFloat()`，移植到Number对象上面，行为完全保持不变

### 3. Number.isInteger()

​	用来**判断一个数值是否为整数**。如果参数不是数值，Number.isInteger返回false

### 4. Number.EPSILON

​	**代表极小的常量Number.EPSILON，它表示 1 与大于 1 的最小浮点数之间的差 **。

​	是 JavaScript 能够表示的最小精度。误差如果小于这个值，就可以认为已经没有意义了，即不存在误差了

### 5. Number.MAX_SAFE_INTEGER**和**Number.MIN_SAFE_INTEGER

​	最小安全整数和最大安全整数

### 6. Number.isSafeInteger()

​	判断一个整数是否落在最小安全整数和最大安全整数范围之内