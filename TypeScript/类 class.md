# 类 class

> 传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件，现在JavaScript程序员将能够使用基于类的面向对象的方式，基于类的继承并且对象是由类构建出来

## 继承

```ts
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal { // Dog派生类从基类Animal中继承了属性和方法。
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

```ts
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { 
        // 派生类包含了一个构造函数，它 必须调用 super()，它会执行基类的构造函数。 
        // 并且在构造函数里访问 this的属性之前，一定要调用 super()。
        super(name); 
    }
    // 重写继承自父类的move方法
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { 
        super(name); 
    }
    move(distanceInMeters = 45) { // 重写继承自父类的move方法
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
// 虽然tom被声明为 Animal类型，但因为它的值是 Horse
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
// Slithering...
// Sammy the Python moved 5m.
tom.move(34); // 调用 tom.move(34)时，它会调用 Horse里重写的方法
// Galloping...
// Tommy the Palomino moved 34m.
```

## 公共，私有与受保护的修饰符

