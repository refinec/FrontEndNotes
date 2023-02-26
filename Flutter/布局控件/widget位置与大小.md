## 居中元素 `Center`

> [`Center`](https://api.flutter-io.cn/flutter/widgets/Center-class.html) widget 可以将它的子 widget 同时以水平和垂直方向居中。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Text(
      'Lorem ipsum',
      style: bold24Roboto,
    ),
  ),
);
```

## 设置绝对位置 `Stack | Positioned`

> 默认情况下，widget 相对于其父元素定位。

使用 [`Positioned`](https://api.flutter-io.cn/flutter/widgets/Positioned-class.html) widget 的 x-y 坐标指定一个 widget 的绝对位置，另外， `Positioned` 需被放在一个 [`Stack`](https://api.flutter-io.cn/flutter/widgets/Stack-class.html) widget 中。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Stack(
    children: [
      Positioned(
        // red box
        left: 24,
        top: 24,
        
        child: Container(
          child: Text(
            'Lorem ipsum',
          ),
        ),
      ),
    ],
  ),
);
```

## 旋转缩放元素 `Transform`

> 想要旋转一个 widget，将它放在 [`Transform`](https://api.flutter-io.cn/flutter/widgets/Transform-class.html) widget 中。
>
> 使用 `Transform` widget 的 `alignment` 和 `origin` 属性分别来指定转换原点（支点）的相对和绝对位置信息。

对于简单的 2D 旋转，例如在 Z 轴上旋转的，创建一个新的 [`Matrix4`](https://api.flutter-io.cn/flutter/vector_math_64/Matrix4-class.html) 对象，并使用它的 `rotateZ()` 方法以弧度单位（角度 × π / 180）指定旋转系数。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Transform(
      alignment: Alignment.center,
      transform: Matrix4.identity()..rotateZ(15 * 3.1415927 / 180),
      child: Container(
        child: Text(
          'Lorem ipsum',
          style: bold24Roboto,
          textAlign: TextAlign.center,
        ),
      ),
    ),
  ),
);
```

当你缩放一个父 widget 时，它的子 widget 也会相应被缩放。

```dart
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Transform(
      alignment: Alignment.center,
      transform: Matrix4.identity()..scale(1.5),
      child: Container(
        child: Text(
          'Lorem ipsum',
          style: bold24Roboto,
          textAlign: TextAlign.center,
        ),
      ),
    ),
  ),
);
```

