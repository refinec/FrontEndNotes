## 矩形圆角 `BorderRadius`

> 用 [`BoxDecoration`](https://api.flutter-io.cn/flutter/painting/BoxDecoration-class.html) 对象的 `borderRadius` 属性。新建一个 [`BorderRadius`](https://api.flutter-io.cn/flutter/painting/BorderRadius-class.html) 对象来指定各个圆角的半径大小。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Container(
      // red circle
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.red[400],
        borderRadius: const BorderRadius.all(
          Radius.circular(8),
        ), 
      ),
      child: Text(
        'Lorem ipsum',
        style: bold24Roboto,
      ),
    ),
  ),
);
```

## 盒子阴影 (box shadows) `BoxShadow`

> 使用 BoxDecoration 的 `boxShadow` 属性来生成一系列 [`BoxShadow`](https://api.flutter-io.cn/flutter/painting/BoxShadow-class.html) widget。你可以定义一个或多个 `BoxShadow` widget，这些 widget 共同用于设置阴影深度、颜色等等。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  margin: const EdgeInsets.only(bottom: 16),
  decoration: BoxDecoration(
    color: Colors.grey[300],
  ),
  child: Center(
    child: Container(
      // red box
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.red[400],
        boxShadow: const <BoxShadow>[
          BoxShadow(
            color: Color(0xcc000000),
            offset: Offset(0, 2),
            blurRadius: 4,
          ),
          BoxShadow(
            color: Color(0x80000000),
            offset: Offset(0, 6),
            blurRadius: 20,
          ),
        ], 
      ),
      child: Text(
        'Lorem ipsum',
        style: bold24Roboto,
      ),
    ),
  ),
);
```

## 生成圆、椭圆 `BoxShape`

使用 [`BoxDecoration`](https://api.flutter-io.cn/flutter/painting/BoxDecoration-class.html) 的 `shape` 属性，它的类型是 [`BoxShape` 枚举](https://api.flutter-io.cn/flutter/painting/BoxShape.html)。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Container(
      // red circle
      decoration: BoxDecoration(
        color: Colors.red[400],
        shape: BoxShape.circle, 
      ),
      
      padding: const EdgeInsets.all(16),
      width: 160,
      height: 160,
      child: Text(
        'Lorem ipsum',
        style: bold24Roboto,
        textAlign: TextAlign.center, 
      ),
    ),
  ),
);
```

