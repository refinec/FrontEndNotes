# Class



## 总结

### 基本语法

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return '(' + this.x + ', ' + this.y + ')';
    }
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```

**注意：**`constructor`方法，这就是构造方法，而`this`关键字则代表实例对象。**类的数据类型就是函数，类本身就指向构造函数 **，<u>**类的所有方法都定义在类的`prototype` 属性上面**</u>，***类的内部所有定义的方法，都是不可枚举的（non-enumerable）***。

* ***在类的实例上面调用方法，其实就是调用原型上的方法***

  `b.constructor === B.prototype.constructor`
  
  ```javascript
  class Point {
      constructor() {
          // ...
      }
      toString() {
          // ...
      }
      toValue() {
          // ...
      }
  }
  // 等同于
  Point.prototype = {
      constructor() {},
      toString() {},
      toValue() {},
  };
  
  class B {}
  let b = new B();
  b.constructor === B.prototype.constructor // true
  ```

#### **constructor** 方法

> constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加

* ***constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象***（指向隔壁老王👈）

  ```javascript
  class Foo {
      constructor() {
          return Object.create(null); // constructor函数返回一个全新的对象，结果导致实例对象不是Foo类的实例
      }
  }
  new Foo() instanceof Foo
  // false
  ```


#### 类的实例

与 ES5 一样，***实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）***

```javascript
//定义类
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return '(' + this.x + ', ' + this.y + ')';
    }
}
var point = new Point(2, 3);
point.toString() // (2, 3)
point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

#### 取值函数（getter）和存值函数（setter）

> 与 ES5 一样，在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为

```javascript
class MyClass {
    constructor() {
        // ...
    }
    get prop() {
        return 'getter';
    }
    set prop(value) {
        console.log('setter: '+value);
    }
}

let inst = new MyClass();
inst.prop = 123;
// setter: 123
inst.prop
// 'getter'
```

* **存值函数和取值函数是设置在属性的 `Descriptor 对象`上的**

  下面代码中，存值函数和取值函数是定义在html属性的描述对象上面，这与 ES5 完全一致

  ```javascript
  class CustomHTMLElement {
      constructor(element) {
          this.element = element;
      }
      get html() {
          return this.element.innerHTML;
      }
      set html(value) {
          this.element.innerHTML = value;
      }
  }
  var descriptor = Object.getOwnPropertyDescriptor(
      CustomHTMLElement.prototype, "html"
  );
  "get" in descriptor  // true
  "set" in descriptor  // true
  ```

#### 属性表达式

类的属性名，可以采用表达式。下面代码中，Square类的方法名getArea，是从表达式得到的：

```javascript
let methodName = 'getArea';
class Square {
    constructor(length) {
        // ...
    }
    [methodName]() {
        // ...
    }
}
```

#### Class表达式

与函数一样，类也可以使用表达式的形式定义。需要注意的是，这个类的名字是Me，但是Me只在 Class 的内部可用，指代当前类。在 Class 外部，这个类只能用MyClass引用。

```javascript
const MyClass = class Me {
    getClassName() {
        return Me.name;
    }
};

let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined

```

如果类的内部没用到的话，可以省略Me

```javascript
const MyClass = class { /* ... */ };
```

***采用 Class 表达式，可以写出立即执行的 Class***

```javascript
let person = new class {
    constructor(name) {
        this.name = name;
    }
    sayName() {
        console.log(this.name);
    }
}('张三');
person.sayName(); // "张三"
```

#### 注意点

1. **类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式**

2. **类不存在变量提升（hoist），这一点与 ES5 完全不同，ES6 不会把类的声明提升到代码头部，必须保证子类在父类之后定义**

3. **name属性总是返回紧跟在class关键字后面的类名**

4. **Generator方法**

   ***在某个方法之前加上星号（\*），就表示该方法是一个 Generator 函数***

   `Symbol.iterator`方法返回一个Foo类的默认遍历器，`for...of`循环会自动调用这个遍历器。

   ```javascript
   class Foo {
       constructor(...args) {
           this.args = args;
       }
       * [Symbol.iterator]() {
           for (let arg of this.args) {
               yield arg;
           }
       }
   }
   for (let x of new Foo('hello', 'world')) {
       console.log(x);
   }
   // hello
   // world
   ```

5. **this的指向**

   类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。

   ```javascript
   class Logger {
       printName(name = 'there') {
           this.print(`Hello ${name}`);
       }
       print(text) {
           console.log(text);
       }
   }
   const logger = new Logger();
   const { printName } = logger;
   printName(); // TypeError: Cannot read property 'print' of undefined
   ```

   上面代码中，printName方法中的this，默认指向Logger类的实例。但是，如**果将这个方法提取出来单独使用，this会指向该方法运行时所在的环境（由于 class 内部是严格模式，所以 this 实际指向的是undefined）**，从而导致找不到print方法而报错.

   **解决方法： **

   1. **在构造方法中绑定this，这样就不会找不到print方法了**

      ```js
      class Logger {
          constructor() {
              this.printName = this.printName.bind(this);
          }
          // ...
      }
      ```
   
   2. **使用箭头函数**
   
      > **箭头函数内部的this总是指向定义时所在的对象**。下面代码中，箭头函数位于构造函数内部，它的定义生效的时候，是在构造函数执行的时候。这时，箭头函数所在的运行环境，肯定是实例对象，所以this会总是指向实例对象。
   
      ```javascript
      class Obj {
          constructor() {
              this.getThis = () => this;
          }
      }
      const myObj = new Obj();
      myObj.getThis() === myObj // true
      ```
   
      3. **使用Proxy，获取方法的时候，自动绑定this**
   
         ```js
         function selfish (target) {
             const cache = new WeakMap();
             const handler = {
                 get (target, key) {
                     const value = Reflect.get(target, key);
                     if (typeof value !== 'function') {
                         return value;
                     }
                     if (!cache.has(value)) {
                         cache.set(value, value.bind(target));
                     }
                     return cache.get(value);
                 }
             };
             const proxy = new Proxy(target, handler);
             return proxy;
         }
         const logger = selfish(new Logger());
         ```

#### 静态方法、属性（static）

在一个**方法前，加上`static`关键字，就表示该方法不会被实例继承(但可被子类继承)**，而是直接通过类来调用，这就称为“静态方法”。

* ***如果静态方法包含this关键字，这个this指的是类，而不是实例。静态方法也可以与非静态方法重名***

```javascript
class Foo {
    static classMethod() {
        return 'hello';
    }
}
Foo.classMethod() // 'hello'
var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function

class Foo {
    static bar() {
        this.baz();
    }
    static baz() {
        console.log('hello');
    }
    baz() {
        console.log('world');
    }
}
Foo.bar() // hello
```

* ***父类的静态方法，可以被子类继承***

```javascript
class Foo {
    static classMethod() {
        return 'hello';
    }
}
class Bar extends Foo {
}
Bar.classMethod() // 'hello'
```

* ***静态方法也是可以从super对象上调用的***

  ```javascript
  class Foo {
      static classMethod() {
          return 'hello';
      }
  }
  class Bar extends Foo {
      static classMethod() {
          return super.classMethod() + ', too';
      }
  }
  Bar.classMethod() // "hello, too"
  ```

* **在静态属性的前面，加上static关键字**

  ```javascript
  // ES5写法
  // 静态属性定义在类的外部。整个类生成以后，再生成静态属性。
  class Foo {
  }
  Foo.prop = 1;
  Foo.prop // 1
  
  // ES6写法
  class MyClass {
      static myStaticProp = 42;
      constructor() {
          console.log(MyClass.myStaticProp); // 42
      }
  }
  ```

#### 私有方法和私有属性（#）

私有方法和私有属性，是***只能在类的内部访问的方法和属性，外部不能访问***。这是常见需求，有利于代码的封装，但 ES6 不提供，只能通过变通方法模拟实现。

1. **将私有方法移出模块，因为模块内部的所有方法都是对外可见的**

   ```javascript
   // 下面代码中，foo是公开方法，内部调用了bar.call(this, baz)。这使得bar实际上成为了当前模块的私有方法
   class Widget {
       foo (baz) {
           bar.call(this, baz);
       }
       // ...
   }
   function bar(baz) {
       return this.snaf = baz;
   }
   ```

2. **利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值**

   ```javascript
   const bar = Symbol('bar');
   const snaf = Symbol('snaf');
   export default class myClass{
       // 公有方法
       foo(baz) {
           this[bar](baz);
       }
       // 私有方法
       [bar](baz) {
           return this[snaf] = baz;
       }
       // ...
   };
   
   // 上面代码中，bar和snaf都是Symbol值，一般情况下无法获取到它们，因此达到了私有方法和私有属性的效果。但是也不是绝对不行，Reflect.ownKeys()依然可以拿到它们
   
   const inst = new myClass();
   Reflect.ownKeys(myClass.prototype)
   // [ 'constructor', 'foo', Symbol(bar) ]
   ```

3. **在属性名、方法之前，使用#表示**

   ```javascript
   class IncreasingCounter {
       #count = 0;
       get value() {
           console.log('Getting the current value!');
           return this.#count;
       }
       increment() {
           this.#count++;
       }
   }
   
   class Foo {
     #a;
     #b;
     constructor(a, b) {
       this.#a = a;
       this.#b = b;
     }
     #sum() {
       return #a + #b;
     }
     printSum() {
       console.log(this.#sum());
     }
   }
   
   ```

   * **私有属性也可以设置 getter 和 setter 方法**

     ```javascript
     class Counter {
         #xValue = 0;
         constructor() {
             super();
             // ...
         }
         get #x() { return #xValue; }
         set #x(value) {
         this.#xValue = value;
     	}
     }
     ```

   * ***私有属性不限于从this引用，只要是在类的内部，实例也可以引用私有属性***

     ```javascript
     class Foo {
         #privateValue = 42;
         static getPrivateValue(foo) {
             return foo.#privateValue;
         }
     }
     Foo.getPrivateValue(new Foo()); // 42
     ```

   * **私有属性和私有方法前面，也可以加上static关键字，表示这是一个静态的私有属性或私有方法**

     下面代码中，`#totallyRandomNumber`是私有属性，`#computeRandomNumber()`是私有方法，只能在FakeMath这个类的内部调用，外部调用就会报错。
     
     ```javascript
     class FakeMath {
         static PI = 22 / 7;
         static #totallyRandomNumber = 4;
         static #computeRandomNumber() {
        		return FakeMath.#totallyRandomNumber;
         }
         static random() {
             console.log('I heard you like random numbers…')
             return FakeMath.#computeRandomNumber();
         }
     }
     FakeMath.PI // 3.142857142857143
     FakeMath.random()
     // I heard you like random numbers…
     // 4
     FakeMath.#totallyRandomNumber // 报错
     FakeMath.#computeRandomNumber() // 报错
     ```

#### `new.target` 属性

new是从构造函数生成实例对象的命令

* ES6 为new命令引入了一个`new.target`属性，**该属性一般用在构造函数之中，返回new命令作用于的那个构造函数 **

* 如果构造函数不是通过`new命令`或`Reflect.construct()`调用的，**`new.target`会返回`undefined`**，因此**这个属性可以用来确定构造函数是怎么调用的**。

  下面代码确保构造函数只能通过new命令调用

  ```javascript
  function Person(name) {
      if (new.target !== undefined) {
          this.name = name;
      } else {
          throw new Error('必须使用 new 命令生成实例');
      }
  }
  // 另一种写法
  function Person(name) {
      if (new.target === Person) {
          this.name = name;
      } else {
          throw new Error('必须使用 new 命令生成实例');
      }
  }
  var person = new Person('张三'); // 正确
  var notAPerson = Person.call(person, '张三');  // 报错
  ```

  ***Class 内部调用new.target，返回当前 Class***

  ```javascript
  class Rectangle {
      constructor(length, width) {
          console.log(new.target === Rectangle);
          this.length = length;
          this.width = width;
      }
  }
  var obj = new Rectangle(3, 4); // 输出 true
  ```

* 需要注意的是，***子类继承父类时，`new.target`会返回子类***

  ```javascript
  class Rectangle {
      constructor(length, width) {
          console.log(new.target === Rectangle);
          // ...
      }
  }
  class Square extends Rectangle {
      constructor(length, width) {
          super(length, width);
      }
  }
  var obj = new Square(3); // 输出 false
  ```

  **利用这个特点，可以写出不能独立使用、必须继承后才能使用的类**

  ```javascript
  class Shape {
      constructor() {
          if (new.target === Shape) {
              throw new Error('本类不能实例化');
          }
      }
  }
  class Rectangle extends Shape {
      constructor(length, width) {
          super();
          // ...
      }
  }
  var x = new Shape();  // 报错
  var y = new Rectangle(3, 4);  // 正确
  ```

### Class的继承

> 通过`extends`关键字实现继承

```javascript
class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y); // 调用父类的constructor(x, y)
        this.color = color;
    }
    toString() {
        return this.color + ' ' + super.toString(); // 调用父类的toString()
    }
}
```

子类必须在**constructor**方法中调用**super方法**，否则新建实例时会报错。这是**因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法**。如果不调用super方法，子类就得不到this对象。实质是先将父类实例对象的属性和方法，加到this上面（所以必须先调用super方法），然后再用子类的构造函数修改this。

* 需要注意的地方是，**在子类的构造函数中，只有调用super之后，才可以使用this关键字**，否则会报错。这是因为子类实例的构建，基于父类实例，只有super方法才能调用父类实例。

  ```javascript
  class Point {
      constructor(x, y) {
          this.x = x;
          this.y = y;
      }
  }
  class ColorPoint extends Point {
      constructor(x, y, color) {
          this.color = color; // ReferenceError
          super(x, y);
          this.color = color; // 正确
      }
  }
  ```

* **父类的静态方法，也会被子类继承**

#### Object.getPrototypeOf()

用来从子类上获取父类，可以使用这个方法判断，**一个类是否继承了另一个类**。

#### super 关键字

> super这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

1. **super作为函数调用时，代表父类的构造函数**。ES6 要求，子类的构造函数必须执行一次super函数

   super虽然代表了父类A的构造函数，但是返回的是子类B的实例，即super内部的this指的是B的实例，因此`super()`在这里相当于**`A.prototype.constructor.call(this)` **。**作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。**

   ```javascript
   // new.target指向当前正在执行的函数。可以看到，在super()执行时，它指向的是子类B的构造函数，而不是父类A的构造函数。也就是说，super()内部的this指向的是B。
   class A {
       constructor() {
           console.log(new.target.name);
       }
   }
   class B extends A {
       constructor() {
           super();
       }
   }
   new A() // A
   new B() // B
   ```


2. ***super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。***

   ```javascript
   class A {
       p() {
           return 2;
       }
   }
   class B extends A {
       constructor() {
           super();
           // super.p()，就是将super当作一个对象使用。这时，super在普通方法之中，指向A.prototype，所以super.p()就相当于A.prototype.p()。
           console.log(super.p()); // 2
       }
   }
   let b = new B();
   ```

   * 注意，由于**super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的**

     ```javascript
     class A {
         constructor() {
             this.p = 2;
         }
     }
     class B extends A {
         get m() {
             return super.p; // p是父类A实例的属性，super.p就引用不到它
         }
     }
     let b = new B();
     b.m // undefined
     
     
     class A {}
     A.prototype.x = 2; // 如果属性定义在父类的原型对象上，super就可以取到
     class B extends A {
         constructor() {
             super();
             console.log(super.x) // 2
         }
     }
     let b = new B();
     ```

   * **在子类普通方法中通过super调用父类的方法时，父类方法内部的this指向当前的子类实例。由于this指向子类实例，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性**

     ```javascript
     class A {
         constructor() {
             this.x = 1;
         }
         print() {
             console.log(this.x);
         }
     }
     class B extends A {
         constructor() {
             super();
             this.x = 2;
         }
         m() {
             // super.print()虽然调用的是A.prototype.print()，但是A.prototype.print()内部的this指向子类B的实例
             super.print();
         }
     }
     let b = new B();
     b.m() // 2
     ```

   * **如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象。**

     ```javascript
     class Parent {
         static myMethod(msg) {
             console.log('static', msg);
         }
         myMethod(msg) {
             console.log('instance', msg);
         }
     }
     class Child extends Parent {
         static myMethod(msg) {
             super.myMethod(msg);
         }
         myMethod(msg) {
             super.myMethod(msg);
         }
     }
     Child.myMethod(1); // static 1
     var child = new Child();
     child.myMethod(2); // instance 2
     
     ```

   * **另外，在子类的静态方法中通过super调用父类的方法时，父类方法内部的this指向当前的子类，而不是子类的实例 **

     ```javascript
     class A {
         constructor() {
             this.x = 1;
         }
         static print() {
             console.log(this.x);
         }
     }
     class B extends A {
         constructor() {
             super();
             this.x = 2;
         }
         static m() {
             super.print();
         }
     }
     B.x = 3;
     B.m() // 3
     ```

   * ***使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错***。

     ```javascript
     class A {}
     class B extends A {
         constructor() {
             super();
             console.log(super.valueOf() instanceof B); // true
         }
     }
     let b = new B();
     ```

     上面代码中，super.valueOf()表明super是一个对象，因此就不会报错。同时，由于**super使得this指向B的实例**，所以super.valueOf()返回的是一个B的实例。

   * 最后，**由于对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。**

   ```javascript
   var obj = {
       toString() {
           return "MyObject: " + super.toString();
       }
   };
   obj.toString(); // MyObject: [object Object]
   ```

#### 类的 prototype 属性和__proto__属性

> Class 作为构造函数的语法糖，同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链。

1. 子类的`__proto__`属性，表示**构造函数的继承，总是指向父类**。

2. 子类**prototype属性的`__proto__`属性**，表示**方法的继承，总是指向父类的prototype属性**。

```javascript
class A {
}
class B extends A {
}
B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

​	因为，类的继承是按照下面的模式实现的:

```javascript
class A {
}
class B {
}
// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);
// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);
const b = new B();
```

《对象的扩展》一章给出过`Object.setPrototypeOf`方法的实现。

```javascript
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
 }
```

这两条继承链，可以这样理解：

* 作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；
* 作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

```javascript
B.prototype = Object.create(A.prototype);
// 等同于
B.prototype.__proto__ = A.prototype;
```

上面代码的A，只要是一个有prototype属性的函数，就能被B继承。由于函数都有prototype属性（除了Function.prototype函数），因此A可以是任意函数

**子类继承Object类:**

1. A其实就是构造函数Object的复制，A的实例就是Object的实例

   ```javascript
   class A extends Object {
   }
   A.__proto__ === Object // true
   A.prototype.__proto__ === Object.prototype // true
   ```

2. 不存在任何继承

   ```javascript
   class A {
   }
   A.__proto__ === Function.prototype // true
   A.prototype.__proto__ === Object.prototype // true
   ```

   这种情况下，A作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承`Function.prototype`。但是，A调用后返回一个空对象（即Object实例），所以`A.prototype.__proto__`指向构造函数（`Object`）的`prototype`属性

#### 实例的 __proto__ 属性

1. 子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型

   ```javascript
   var p1 = new Point(2, 3);
   var p2 = new ColorPoint(2, 3, 'red');
   p2.__proto__ === p1.__proto__ // false
   p2.__proto__.__proto__ === p1.__proto__ // true
   ```

   因此，通过子类实例的`__proto__.__proto__`属性，可以修改父类实例的行为

   ```javascript
   p2.__proto__.__proto__.printName = function () {
       console.log('Ha');
   };
   p1.printName() // "Ha"
   ```

#### 原生构造函数的继承

ECMAScript 的原生构造函数大致有下面这些:

1. **Boolean()**
2. **Number()**
3. **String()**
4. **Array()**
5. **Date()**
6. **Function()**
7. **RegExp()**
8. **Error()**
9. **Object()**

1. 以前，这些原生构造函数是无法继承的，比如，不能自己定义一个Array的子类。因为ES5 是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数。比如，Array构造函数有一个内部属性[[DefineOwnProperty]]，用来定义新属性时，更新length属性，这个内部属性无法在子类获取，导致子类的length属性行为不正常。

2. ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。下面是一个继承Array的例子。

   ```javascript
   class MyArray extends Array {
       constructor(...args) {
           super(...args);
       }
   }
   var arr = new MyArray();
   arr[0] = 12;
   arr.length // 1
   arr.length = 0;
   arr[0] // undefined
   ```

   上面这个例子也说明，**extends关键字不仅可以用来继承类，还可以用来继承原生的构造函数**。因此可以在原生数据结构的基础上，定义自己的数据结构。下面就是定义了一个带版本功能的数组。

   ```javascript
   class VersionedArray extends Array {
       constructor() {
           super();
           this.history = [[]];
       }
       commit() {
           this.history.push(this.slice());
       }
       revert() {
           this.splice(0, this.length, ...this.history[this.history.length - 1]);
       }
   }
   var x = new VersionedArray();
   x.push(1);
   x.push(2);
   x // [1, 2]
   x.history // [[]]
   x.commit();
   x.history // [[], [1, 2]]
   x.push(3);
   x // [1, 2, 3]
   x.history // [[], [1, 2]]
   x.revert();
   x // [1, 2]
   ```

   注意，继承Object的子类，有一个[行为差异](http://stackoverflow.com/questions/36203614/super-does-not-pass-arguments-when-instantiating-a-class-extended-from-object)

   ```javascript
   class NewObj extends Object{
       constructor(){
           super(...arguments);
       }
   }
   var o = new NewObj({attr: true});
   o.attr === true  // false
   ```

   上面代码中，**NewObj继承了Object，但是无法通过super方法向父类Object传参**。这是因为 ES6 改变了Object构造函数的行为，一旦发现Object方法不是通过new Object()这种形式调用，ES6 规定Object构造函数会忽略参数。

#### Mixin 模式的实现

> Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口。它的最简单实现如下

```javascript
const a = {
    a: 'a'
};
const b = {
    b: 'b'
};
// c对象是a对象和b对象的合成，具有两者的接口
const c = {...a, ...b}; // {a: 'a', b: 'b'}
```

下面是一个更完备的实现，将多个类的接口“混入”（mix in）另一个类。

```javascript
function mix(...mixins) {
    class Mix {
        constructor() {
            for (let mixin of mixins) {
                copyProperties(this, new mixin()); // 拷贝实例属性
            }
        }
    }
    for (let mixin of mixins) {
        copyProperties(Mix, mixin); // 拷贝静态属性
        copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
    }
    return Mix;
}
function copyProperties(target, source) {
    for (let key of Reflect.ownKeys(source)) {
        if ( key !== 'constructor'
            && key !== 'prototype'
            && key !== 'name'
           ) {
            let desc = Object.getOwnPropertyDescriptor(source, key);
            Object.defineProperty(target, key, desc);
        }
    }
}
```

上面代码的mix函数，可以将多个对象合成为一个类。使用的时候，只要继承这个类即可。

```javascript
class DistributedEdit extends mix(Loggable, Serializable) {
    // ...
}
```

