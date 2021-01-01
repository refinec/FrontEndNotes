# Math

## 加减乘除

1. **_.add(augend,  addend) **：返回两个数**相加**的结果

   ```javascript
   _.add(6,4); // 10
   ```

2. **_.subtract(minuend, subtrahend) **: 返回两个数**相减**的差

3. **_.divide(dividend, divisor)** ：返回两个数**相除**的结果

   ```javascript
   _.divide(6, 4); // 1.5
   ```

4. **_.multiply(multiplier, multiplicand) ：** 返回两个数**相乘**结果

   ```javascript
   _.multiply(6, 4); // 24
   ```

## 数字精度相关

1. **_.ceil(number, [precision=0])** : 返回**向上舍入 **的值， precision（精度）可以理解为保留几位小数

   ```javascript
   _.ceil(4.006); // 5
   _.ceil(6.004, 2); // 6.01
   _.ceil(6040, -2); // 6100
   ```

2. **_.floor(number, [precision=0]) ** : 返回**向下舍入**的值，precision（精度）可以理解为保留几位小数

   ```javascript
   _.floor(4.006); // 4
   _.floor(0.046, 2); // 0.04
   _.floor(4060, -2); // 4000
   ```

3. **_.round(number, [precision=0])：** 返回**四舍五入 **的数字，precision（精度） 四舍五入

   ```javascript
   _.round(4.006); // 4
   _.round(4.006, 2); // 4.01
   _.round(4060, -2); // 4100
   ```

   