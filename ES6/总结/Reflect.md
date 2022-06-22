# Reflect

## Reflect对象的设计目的

Reflect对象 与 Proxy对象一样，也是 ES6 为了操作对象而提供的新 API

1. 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。

2. **修改某些 Object方法 的返回结果**，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false `

   ```js
   // 老写法
   try {
       Object.defineProperty(target, property, attributes);
       // success
   } catch (e) {
       // failure
   }
   // 新写法
   if (Reflect.defineProperty(target, property, attributes)) {
       // success
   } else {
       // failure
   }
   ```


3.  **让Object操作都变成函数行为**。某些Object操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

4. **Reflect对象的方法与Proxy对象的方法一一对应**，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为

   ```js
   Proxy(target, {
       set: function(target, name, value, receiver) {
           var success = Reflect.set(target, name, value, receiver);
           if (success) {
               console.log('property ' + name + ' on ' + target + ' set to ' + value);
           }
           return success;
       }
   });
   
   new Proxy(obj, {
     get(target, name) {
       console.log('get', target, name);
       return Reflect.get(target, name);
     },
     deleteProperty(target, name) {
       console.log('delete' + name);
       return Reflect.deleteProperty(target, name);
     },
     has(target, name) {
       console.log('has' + name);
       return Reflect.has(target, name);
     }
   });
   
   // 老写法
   Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1
   // 新写法
   Reflect.apply(Math.floor, undefined, [1.75]) // 1
   ```

## 静态方法

Reflect对象一共有 13 个静态方法：

**第一个参数不是对象，会报错**

1. Reflect.**get**(target, name, receiver)
2. Reflect.**set**(target, name, value, receiver)
3. Reflect.**has**(target, name)
4. Reflect.**defineProperty**(target, name, desc)
5. Reflect.**deleteProperty**(target, name)
6. Reflect.**apply**(target, thisArg,  args)
7. Reflect.**preventExtensions**(target)
8. Reflect.**isExtensible**(target)
9. Reflect.**ownKeys**(target)
10. Reflect.**getOwnPropertyDescriptor**(target,  name)
11. Reflect.**construct**(target, args)
12. Reflect.**getPrototypeOf**(target)
13. Reflect.**setPrototypeOf**(target, prototype)

## 1.Reflect.**get**(target, name, receiver)

> Reflect.get方法查找并返回target对象的name属性，如果没有该属性，则返回undefined

```js
var myObject = {
    foo: 1,
    bar: 2,
    get baz() {
        return this.foo + this.bar;
    },
}
Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz') // 3
```

**如果name属性部署了读取函数（getter），读取函数的this绑定receiver**

```js
var myObject = {
    foo: 1,
    bar: 2,
    get baz() {
        return this.foo + this.bar;
    },
};
var myReceiverObject = {
    foo: 4,
    bar: 4,
};
Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

## 2.Reflect.**set**(target, name, value, receiver)

> Reflect.set方法设置target对象的name属性等于value

```js
var myObject = {
    foo: 1,
    set bar(value) {
        return this.foo = value;
    },
}
myObject.foo // 1
Reflect.set(myObject, 'foo', 2);
myObject.foo // 2
Reflect.set(myObject, 'bar', 3)
myObject.foo // 3
```

**如果name属性设置了赋值函数，则赋值函数的this绑定receiver**

```js
var myObject = {
    foo: 4,
    set bar(value) {
        return this.foo = value;
    },
};
var myReceiverObject = {
    foo: 0,
};
Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 4
myReceiverObject.foo // 1
```

**注意**，如果 Proxy对象和 Reflect对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了receiver，那么Reflect.set会触发Proxy.defineProperty拦截：

```js
let p = {
    a: 'a'
};
let handler = {
    set(target, key, value, receiver) {
        console.log('set');
        Reflect.set(target, key, value, receiver)
    },
    defineProperty(target, key, attribute) {
        console.log('defineProperty');
        Reflect.defineProperty(target, key, attribute);
    }
};
let obj = new Proxy(p, handler);
obj.a = 'A';
// set
// defineProperty
```

上面代码中，Proxy.set拦截里面使用了Reflect.set，而且传入了receiver，导致触发Proxy.defineProperty拦截。这是因为Proxy.set的receiver参数总是指向当前的 Proxy实例（即上例的obj），而Reflect.set一旦传入receiver，就会将属性赋值到receiver上面（即obj），导致触发defineProperty拦截。如果Reflect.set没有传入receiver，那么就不会触发defineProperty拦截

## 3.Reflect.**has**(target, name)

> Reflect.has方法对应name in obj里面的in运算符

```js
var myObject = {
    foo: 1,
};
// 旧写法
'foo' in myObject // true
// 新写法
Reflect.has(myObject, 'foo') // true
```

## 4.Reflect.**defineProperty**(target, name, desc)

> Reflect.defineProperty方法基本等同于Object.defineProperty，用来为对象定义属性。

```js
function MyDate() {
    /*…*/
}
// 旧写法
Object.defineProperty(MyDate, 'now', {
    value: () => Date.now()
});
// 新写法
Reflect.defineProperty(MyDate, 'now', {
    value: () => Date.now()
});
```

**与Proxy.defineProperty配合使用**

```js
const p = new Proxy({}, {
    defineProperty(target, prop, descriptor) {
        console.log(descriptor);
        return Reflect.defineProperty(target, prop, descriptor);
    }
});
p.foo = 'bar';
// {value: "bar", writable: true, enumerable: true, configurable: true}
p.foo // "bar"
```

上面代码中，Proxy.defineProperty对属性赋值设置了拦截，然后使用Reflect.defineProperty完成了赋值。

- `writable` : 表示能否修改属性的值，即值是可写的还是只读。
- `enumaerable ` : 目标属性是否可被枚举（遍历）。用来控制所描述的属性，是否将被包括在for…in循环之中。具体来说，如果一个属性的enumerable为false，下面三个操作不会取到该属性：
  1. for…in循环
  2. Object.keys方法
  3. JSON.stringify方法
- `configurable` : 表示能否通过 **delete** 删除属性、能否修改属性的特性，或者将属性修改为访问器（getter、setter）属性。

## 5.Reflect.**deleteProperty**(target, name)

> Reflect.deleteProperty方法等同于delete obj[name]，用于删除对象的属性

```js
const myObj = { foo: 'bar' };
// 旧写法
delete myObj.foo;

// 新写法
// 如果删除成功，或者被删除的属性不存在，返回true；
// 删除失败，被删除的属性依然存在，返回false
Reflect.deleteProperty(myObj, 'foo'); 
```

## 6.Reflect.**apply**(target, thisArg,  args)

> Reflect.apply方法等同于`Function.prototype.apply.call(func, thisArg, args)`，用于绑定this对象后执行给定函数

一般来说，如果要绑定一个函数的this对象，可以这样写`fn.apply(obj, args)`，但是如果函数定义了自己的apply方法，就只能写成`Function.prototype.apply.call(fn, obj, args)`，采用Reflect对象可以简化这种操作。

```js
const ages = [11, 33, 12, 54, 18, 96];
// 旧写法
const youngest = Math.min.apply(Math, ages);
const oldest = Math.max.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);
// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const oldest = Reflect.apply(Math.max, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```

## 7.Reflect.**preventExtensions**(target)

> `Reflect.preventExtensions`对应`Object.preventExtensions`方法，用于让一个对象变为不可扩展。它返回一个布尔值，表示是否操作成功

```js
var myObject = {};
// 旧写法
Object.preventExtensions(myObject) // Object {}
// 新写法
Reflect.preventExtensions(myObject) // true
```

**如果参数不是对象，Object.preventExtensions在 ES5 环境报错，在 ES6 环境返回传入的参数，而Reflect.preventExtensions会报错**

```js
// ES5 环境
Object.preventExtensions(1) // 报错
// ES6 环境
Object.preventExtensions(1) // 1
// 新写法
Reflect.preventExtensions(1) // 报错
```

## 8.Reflect.**isExtensible**(target)

> `Reflect.isExtensible`方法对应`Object.isExtensible`，返回一个布尔值，表示当前对象是否可扩展

```js
const myObject = {};
// 旧写法
Object.isExtensible(myObject) // true
// 新写法
Reflect.isExtensible(myObject) // true
```

如果**参数不是对象，Object.isExtensible会返回false**，因为非对象本来就是不可扩展的，而**Reflect.isExtensible会报错**。

```js
Object.isExtensible(1) // false
Reflect.isExtensible(1) // 报错
```

## 9.Reflect.**ownKeys**(target)

> `Reflect.ownKeys`方法用于返回对象的所有属性，基本等同于`Object.getOwnPropertyNames`与`Object.getOwnPropertySymbols`之和

```js
var myObject = {
    foo: 1,
    bar: 2,
    [Symbol.for('baz')]: 3,
    [Symbol.for('bing')]: 4,
};
// 旧写法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']
Object.getOwnPropertySymbols(myObject)
//[Symbol(baz), Symbol(bing)]
// 新写法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol(baz), Symbol(bing)]
```

如果`Reflect.ownKeys()`方法的第一个参数不是对象，会报错

## 10.Reflect.**getOwnPropertyDescriptor**(target,  name)

> `Reflect.getOwnPropertyDescriptor`基本等同于`Object.getOwnPropertyDescriptor`，用于得到指定属性的描述对象

```js
var myObject = {};
Object.defineProperty(myObject, 'hidden', {
    value: true,
    enumerable: false,
});
// 旧写法
var theDescriptor = Object.getOwnPropertyDescriptor(myObject, 'hidden');
// 新写法
var theDescriptor = Reflect.getOwnPropertyDescriptor(myObject, 'hidden');
```

**如果第一个参数不是对象，`Object.getOwnPropertyDescriptor(1, 'foo')`不报错，返回`undefined`，而`Reflect.getOwnPropertyDescriptor(1, 'foo')`会抛出错误，表示参数非法**。

## 11.Reflect.**construct**(target, args)

> `Reflect.construct`方法等同于**`new target(...args)`**，这提供了一种不使用new，来调用构造函数的方法

```js
function Greeting(name) {
    this.name = name;
}
// new 的写法
const instance = new Greeting('张三');
// Reflect.construct 的写法
const instance = Reflect.construct(Greeting, ['张三']);
```

## 12.Reflect.**getPrototypeOf**(target)

> `Reflect.getPrototypeOf`方法用于读取对象的__proto__属性，对应`Object.getPrototypeOf(obj)`

```js
const myObj = new FancyThing();
// 旧写法
Object.getPrototypeOf(myObj) === FancyThing.prototype;
// 新写法
Reflect.getPrototypeOf(myObj) === FancyThing.prototype;
```

## 13.Reflect.**setPrototypeOf**(target, prototype)

> `Reflect.setPrototypeOf`方法用于设置目标对象的原型（prototype），对应`Object.setPrototypeOf(obj, newProto)`方法。它返回一个布尔值，表示是否设置成功。

```js
const myObj = {};
// 旧写法
Object.setPrototypeOf(myObj, Array.prototype);
// 新写法
Reflect.setPrototypeOf(myObj, Array.prototype);
myObj.length // 0
```

**如果无法设置目标对象的原型（比如，目标对象禁止扩展），Reflect.setPrototypeOf方法返回false**

```js
Reflect.setPrototypeOf({}, null)
// true
Reflect.setPrototypeOf(Object.freeze({}), null)
// false
```

**如果第一个参数不是对象，Object.setPrototypeOf会返回第一个参数本身，而Reflect.setPrototypeOf会报错**

```js
Object.setPrototypeOf(1, {})
// 1
Reflect.setPrototypeOf(1, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

**如果第一个参数是undefined或null，Object.setPrototypeOf和Reflect.setPrototypeOf都会报错**

```js
Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
Reflect.setPrototypeOf(null, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

## 实例：使用 Proxy 实现观察者模式

> 观察者模式（Observer mode）指的是**函数自动观察数据对象，一旦对象有变化，函数就会自动执行**

```js
const person = observable({ // 数据对象person是观察目标
    name: '张三',
    age: 20
});
function print() { // 函数print是观察者
    console.log(`${person.name}, ${person.age}`)
}
observe(print); // 一旦数据对象发生变化，print就会自动执行
person.name = '李四';
// 输出
// 李四, 20
```

使用 Proxy 写一个观察者模式的最简单实现，即实现observable和observe这两个函数。思路是observable函数返回一个原始对象的 Proxy 代理，拦截赋值操作，触发充当观察者的各个函数。

```js
const queuedObservers = new Set();
const observe = fn => queuedObservers.add(fn);
const observable = obj => new Proxy(obj, {set});
function set(target, key, value, receiver) {
    const result = Reflect.set(target, key, value, receiver);
    queuedObservers.forEach(observer => observer());
    return result;
}
```

上面代码中，先定义了一个Set集合，所有观察者函数都放进这个集合。然后，observable函数返回原始对象的代理，拦截赋值操作。拦截函数set之中，会自动执行所有观察者。