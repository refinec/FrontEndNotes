# 日期Date

1. **_.now():** **获得 Unix 纪元 *(1 January* *1970 00:00:00* *UTC)* 直到现在的毫秒数 **

   ```javascript
   // 记录延迟函数调用的毫秒数
   _.defer(function(stamp){
       console.log(_.now() - stamp)
   }, _.now());
   ```

2. **_.isDate(value) ** ：**检查 value 是否为 Date 对象 **

   如果 value 是一个日期对象，那么返回 true，否则返回 false。

   ```javascript
   _.isDate(new Date); // true
   ```

   