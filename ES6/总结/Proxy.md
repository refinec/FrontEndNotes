# Proxy

> Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。
>
> Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”
>

```javascript
var obj = new Proxy({}, {
    get: function (target, propKey, receiver) {
        console.log(`getting ${propKey}!`);
        return Reflect.get(target, propKey, receiver);
    },
    set: function (target, propKey, value, receiver) {
        console.log(`setting ${propKey}!`);
        return Reflect.set(target, propKey, value, receiver);
    }
});

obj.count = 1
//  setting count!
    ++obj.count
//  getting count!
//  setting count!
//  2
```

**注意：**<u>new Proxy()表示生成一个Proxy实例，第一个参数表示所要拦截的目标对象。第二个参数是一个配置对象，用来定制拦截行为。</u>

* 要使得Proxy起作用，必须针对Proxy实例（上例是proxy对象）进行操作，而不是针对目标对象（上例是空对象）进行操作

* **Proxy 实例也可以作为其他对象的原型对象 **

  ```javascript
  
  var proxy = new Proxy({}, {
      get: function(target, propKey) {
          return 35;
      }
  });
  // proxy对象是obj对象的原型，obj对象本身并没有time属性，所以根据原型链，会在proxy对象上读取该属性，导致被拦截
  let obj = Object.create(proxy);
  obj.time // 35
  ```

* 同一个拦截器函数，可以设置拦截多个操作。对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。

  ```javascript
  var handler = {
      get: function(target, name) {
          if (name === 'prototype') {
              return Object.prototype;
          }
          return 'Hello, ' + name;
      },
      apply: function(target, thisBinding, args) {
          return args[0];
      },
      construct: function(target, args) {
          return {value: args[1]};
      }
  };
  var fproxy = new Proxy(function(x, y) {
      return x + y;
  }, handler);
  fproxy(1, 2) // 1
  new fproxy(1, 2) // {value: 2}
  fproxy.prototype === Object.prototype // true
  fproxy.foo === "Hello, foo" // true
  ```

## Proxy.revocable()

> Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以取消Proxy实例。

**Proxy.revocable的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。**

```javascript
let target = {};
let handler = {};
let {proxy, revoke} = Proxy.revocable(target, handler);
proxy.foo = 123;
proxy.foo // 123
revoke();
proxy.foo // TypeError: Revoked
```

## Proxy 支持的拦截操作方法一览，一共 13 种

1. **`get(target, propKey, receiver)`**：拦截对象**属性的读取**，比如`proxy.foo ` 和 `proxy['foo']`。
2. **`set(target, propKey, value, receiver)`**：拦截对象**属性的设置**，比如`proxy.foo = v`或`proxy['foo'] = v`，**返回一个布尔值**。
3. **`has(target,propKey)`**：拦截`propKey in proxy`的操作，**返回一个布尔值.**
4. **`deleteProperty(target,propKey)`**：拦截`delete proxy[propKey]`的操作，**返回一个布尔值。**
5. **`ownKeys(target)`**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，**返回一个数组**。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
6. **`getOwnPropertyDescriptor(target,propKey)`**：拦截`Object.getOwnPropertyDescriptor(proxy,     propKey)`，**返回属性的描述对象**。
7. **`defineProperty(target, propKey, propDesc)`**：拦截`Object.defineProperty(proxy,     propKey, propDesc）、Object.defineProperties(proxy, propDescs)`，**返回一个布尔值**。
8. **`preventExtensions(target)`**：拦截`Object.preventExtensions(proxy)`，**返回一个布尔值**。
9. **`getPrototypeOf(target)`**：拦截`Object.getPrototypeOf(proxy)`，**返回一个对象**。
10. **`isExtensible(target)`**：拦截`Object.isExtensible(proxy)`，**返回一个布尔值**。
11. **`setPrototypeOf(target,proto)`**：拦截`Object.setPrototypeOf(proxy, proto)`，**返回一个布尔值**。<u>如果目标对象是函数，那么还有两种额外操作可以拦截。</u>
12. **`apply(target, object, args)`**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)、proxy.call(object,   ...args)、proxy.apply(...)`。
13. **`construct(target, args, newTarget)`**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

## 拦截操作方法详细介绍

### 1. get(target, propKey, receiver)

> get方法用于拦截某个属性的读取操作，接受三个参数，依次为 目标对象、属性名、proxy 实例本身（严格地说，是操作行为所针对的对象。该参数可选）。

* get方法可以继承

  ```javascript
  let proto = new Proxy({}, {
      get(target, propertyKey, receiver) {
          console.log('GET ' + propertyKey);
          return target[propertyKey];
      }
  });
  let obj = Object.create(proto); // 拦截操作定义在Prototype对象上面，所以如果读取obj对象继承的属性时，拦截会生效
  obj.foo // "GET foo"
  ```

* get方法的第三个参数的例子,它总是指向原始的读操作所在的那个对象，一般情况下就是 Proxy 实例

  ```javascript
  const proxy = new Proxy({}, {
      get: function(target, key, receiver) {
          return receiver;
      }
  });
  proxy.getReceiver === proxy // true
  
  const d = Object.create(proxy);
  d.a === d // true
  ```

* 如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错

  ```javascript
  const target = Object.defineProperties({}, {
      foo: {
          value: 123,
          writable: false,
          configurable: false
      },
  });
  const handler = {
      get(target, propKey) {
          return 'abc';
      }
  };
  const proxy = new Proxy(target, handler);
  proxy.foo
  // TypeError: Invariant check failed
  ```

#### 例子：

1. 利用get拦截，实现一个生成各种 DOM 节点的通用函数dom

   ```javascript
   const dom = new Proxy({}, {
       get(target, property) {
           return function(attrs = {}, ...children) {
               const el = document.createElement(property);
               for (let prop of Object.keys(attrs)) {
                   el.setAttribute(prop, attrs[prop]);
               }
               for (let child of children) {
                   if (typeof child === 'string') {
                       child = document.createTextNode(child);
                   }
                   el.appendChild(child);
               }
               return el;
           }
       }
   });
   const el = dom.div({},
                      'Hello, my name is ',
                      dom.a({href: '//example.com'}, 'Mark'),
                      '. I like:',
                      dom.ul({},
                             dom.li({}, 'The web'),
                             dom.li({}, 'Food'),
                             dom.li({}, '…actually that\'s it')
                            )
                     );
   document.body.appendChild(el);
   ```

2. 将读取属性的操作（get），转变为执行某个函数，从而实现属性的链式操作

   ```javascript
   var pipe = function (value) {
       var funcStack = [];
       var oproxy = new Proxy({} , {
           get : function (pipeObject, fnName) {
               if (fnName === 'get') {
                   return funcStack.reduce(function (val, fn) {
                       return fn(val);
                   },value);
               }
               funcStack.push(window[fnName]);
               return oproxy;
           }
       });
       return oproxy;
   }
   var double = n => n * 2;
   var pow    = n => n * n;
   var reverseInt = n => n.toString().split("").reverse().join("") | 0;
   pipe(3).double.pow.reverseInt.get; // 63
   ```

   

### 2. set(target, propKey, value, receiver)

> set方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身(指向原始赋值行为所在的对象)，其中最后一个参数可选。

* 利用set方法，还可以数据绑定，即每当对象发生变化时，会自动更新 DOM。

* 结合get和set方法，就可以做到防止这些内部属性被外部读写。

  ```javascript
  const handler = {
      get (target, key) {
          invariant(key, 'get');
          return target[key];
      },
      set (target, key, value) {
          invariant(key, 'set');
          target[key] = value;
          return true;
      }
  };
  function invariant (key, action) {
      if (key[0] === '_') {
          throw new Error(`Invalid attempt to ${action} private "${key}" property`);
      }
  }
  const target = {};
  const proxy = new Proxy(target, handler);
  proxy._prop
  // Error: Invalid attempt to get private "_prop" property
  proxy._prop = 'c'
  // Error: Invalid attempt to set private "_prop" property
  ```

* 如果目标对象自身的某个属性，不可写且不可配置，那么set方法将不起作用

* 严格模式下，set代理如果没有返回true，就会报错

  ```javascript
  'use strict';
  const handler = {
      set: function(obj, prop, value, receiver) {
          obj[prop] = receiver;
          // 无论有没有下面这一行，都会报错.
          // 严格模式下，set代理返回false或者undefined，都会报错。
          return false;
      }
  };
  const proxy = new Proxy({}, handler);
  proxy.foo = 'bar';
  // TypeError: 'set' on proxy: trap returned falsish for property 'foo'
  ```

### 3.has(target,propKey)

> has方法用来拦截HasProperty操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是in运算符
>
> has方法可以接受两个参数，分别是目标对象、需查询的属性名。

* 使用has方法隐藏某些属性，不被in运算符发现

  ```javascript
  var handler = {
      has (target, key) {
          if (key[0] === '_') {
              return false;
          }
          return key in target;
      }
  };
  var target = { _prop: 'foo', prop: 'foo' };
  var proxy = new Proxy(target, handler);
  '_prop' in proxy // false
  ```

* 如果原对象不可配置或者不可扩展，这时has拦截会报错

  ```javascript
  var obj = { a: 10 };
  Object.preventExtensions(obj);
  var p = new Proxy(obj, {
      has: function(target, prop) {
          return false;
      }
  });
  'a' in p // TypeError is thrown
  ```

  **注意：** has方法拦截的是hasProperty操作，而不是hasOwnProperty操作，即has方法不判断一个属性是对象自身的属性，还是继承的属性.

* 虽然for...in循环也用到了in运算符，但has拦截只对in运算符生效，对for...in循环不生效，导致不符合要求的属性没有被for...in循环所排除

  ```javascript
  let stu1 = {name: '张三', score: 59};
  let stu2 = {name: '李四', score: 99};
  let handler = {
      has(target, prop) {
          if (prop === 'score' && target[prop] < 60) {
              console.log(`${target.name} 不及格`);
              return false;
          }
          return prop in target;
      }
  }
  let oproxy1 = new Proxy(stu1, handler);
  let oproxy2 = new Proxy(stu2, handler);
  'score' in oproxy1
  // 张三 不及格
  // false
  'score' in oproxy2
  // true
  for (let a in oproxy1) {
      console.log(oproxy1[a]);
  }
  // 张三
  // 59
  for (let b in oproxy2) {
      console.log(oproxy2[b]);
  }
  // 李四
  // 99
  ```

### 4.deleteProperty(target,propKey)

> deleteProperty方法用于拦截delete操作

* 如果这个方法抛出错误或者返回false，当前属性就无法被delete命令删除。另外，如果目标对象自身的不可配置（configurable）的属性，不能被deleteProperty方法删除，否则报错。

  ```javascript
  var handler = {
      deleteProperty (target, key) {
          invariant(key, 'delete');
          delete target[key];
          return true;
      }
  };
  function invariant (key, action) {
      if (key[0] === '_') {
          throw new Error(`Invalid attempt to ${action} private "${key}" property`);
      }
  }
  var target = { _prop: 'foo' };
  var proxy = new Proxy(target, handler);
  delete proxy._prop
  // Error: Invalid attempt to delete private "_prop" property
  ```

### 5.ownKeys(target)

> ownKeys方法用来拦截对象自身属性的读取操作。具体来说，拦截以下操作。
>
> - Object.getOwnPropertyNames()
> - Object.getOwnPropertySymbols()
> - Object.keys()
> - for...in循环

```javascript
let target = {
    _bar: 'foo',
    _prop: 'bar',
    prop: 'baz'
};
let handler = {
    ownKeys (target) {
        return Reflect.ownKeys(target).filter(key => key[0] !== '_');
    }
};
let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
    console.log(target[key]);
}
// "baz"
```

* 注意，使用`Object.keys`方法时，有三类属性会被`ownKeys`方法自动过滤，不会返回

  - 目标对象上不存在的属性
  - 属性名为 Symbol 值
  - 不可遍历（enumerable）的属性

  ```javascript
  let target = {
      a: 1,
      b: 2,
      c: 3,
      [Symbol.for('secret')]: '4',
  };
  Object.defineProperty(target, 'key', {
      enumerable: false,
      configurable: true,
      writable: true,
      value: 'static'
  });
  let handler = {
      ownKeys(target) {
          return ['a', 'd', Symbol.for('secret'), 'key'];
      }
  };
  let proxy = new Proxy(target, handler);
  Object.keys(proxy)
  // ['a']
  ```

* `ownKeys`方法还可以拦截`Object.getOwnPropertyNames()`、`for...in`循环。

*  ownKeys方法返回的数组成员，只能是字符串或 Symbol 值。如果有其他类型的值，或者返回的根本不是数组，就会报错.

  ```javascript
  var obj = {};
  var p = new Proxy(obj, {
      ownKeys: function(target) {
          return [123, true, undefined, null, {}, []];
      }
  });
  Object.getOwnPropertyNames(p)
  // Uncaught TypeError: 123 is not a valid property name
  ```

* 如果目标对象自身包含不可配置的属性，则该属性必须被ownKeys方法返回，否则报错

  ```javascript
  var obj = {};
  Object.defineProperty(obj, 'a', {
      configurable: false,
      enumerable: true,
      value: 10 }
                       );
  var p = new Proxy(obj, {
      ownKeys: function(target) {
          return ['b'];
      }
  });
  Object.getOwnPropertyNames(p)
  // Uncaught TypeError: 'ownKeys' on proxy: trap result did not include 'a'
  ```

* 如果目标对象是不可扩展的（non-extensible），这时ownKeys方法返回的数组之中，必须包含原对象的所有属性，且不能包含多余的属性，否则报错。

  ```javascript
  var obj = {
      a: 1
  };
  Object.preventExtensions(obj);
  var p = new Proxy(obj, {
      ownKeys: function(target) {
          return ['a', 'b'];
      }
  });
  Object.getOwnPropertyNames(p)
  // Uncaught TypeError: 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
  ```

### 6.getOwnPropertyDescriptor(target,propKey)

> getOwnPropertyDescriptor方法拦截Object.getOwnPropertyDescriptor()，返回一个属性描述对象或者undefined。

```javascript
var handler = {
    getOwnPropertyDescriptor (target, key) {
        if (key[0] === '_') {
            return;
        }
        return Object.getOwnPropertyDescriptor(target, key);
    }
};
var target = { _foo: 'bar', baz: 'tar' };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, 'wat')
// undefined
Object.getOwnPropertyDescriptor(proxy, '_foo')
// undefined
Object.getOwnPropertyDescriptor(proxy, 'baz')
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

### 7.defineProperty(target, propKey, propDesc)

> defineProperty方法拦截了Object.defineProperty操作

* 如果defineProperty方法返回false，导致添加新属性总是无效。

  ```javascript
  var handler = {
      defineProperty (target, key, descriptor) {
          return false;
      }
  };
  var target = {};
  var proxy = new Proxy(target, handler);
  proxy.foo = 'bar' // 不会生效
  ```

* 如果目标对象不可扩展（non-extensible），则defineProperty不能增加目标对象上不存在的属性，否则会报错。
* 如果目标对象的某个属性不可写（writable）或不可配置（configurable），则defineProperty方法不得改变这两个设置。

### 8.preventExtensions(target)

> preventExtensions方法拦截Object.preventExtensions()。该方法必须返回一个布尔值，否则会被自动转为布尔值。

* 这个方法有一个限制，只有目标对象不可扩展时（即Object.isExtensible(proxy)为false），proxy.preventExtensions才能返回true，否则会报错。为了防止出现这个问题，通常要在proxy.preventExtensions方法里面，调用一次Object.preventExtensions。

```javascript
var proxy = new Proxy({}, {
    preventExtensions: function(target) {
        return true;
    }
});
Object.preventExtensions(proxy)
// Uncaught TypeError: 'preventExtensions' on proxy: trap returned truish but the proxy target is extensible

var proxy = new Proxy({}, {
    preventExtensions: function(target) {
        console.log('called');
        Object.preventExtensions(target);
        return true;
    }
});
Object.preventExtensions(proxy)
// "called"
// Proxy {}
```



### 9.getPrototypeOf(target)

> getPrototypeOf方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作。
>
> - Object.prototype.__proto__
> - Object.prototype.isPrototypeOf()
> - Object.getPrototypeOf()
> - Reflect.getPrototypeOf()
> - instanceof

```javascript
var proto = {};
var p = new Proxy({}, {
    getPrototypeOf(target) {
        return proto;
    }
});
Object.getPrototypeOf(p) === proto // true
```

* getPrototypeOf方法的返回值必须是对象或者null，否则报错。
* 如果目标对象不可扩展（non-extensible）， getPrototypeOf方法必须返回目标对象的原型对象。

### 10.isExtensible(target)

> isExtensible方法拦截Object.isExtensible操作,该方法只能返回布尔值，否则返回值会被自动转为布尔值。

```javascript
var p = new Proxy({}, {
    isExtensible: function(target) {
        console.log("called");
        return true;
    }
});
Object.isExtensible(p)
// "called"
// true
```

* 注意这个方法有一个强限制，它的返回值必须与目标对象的`isExtensible`属性保持一致，否则就会抛出错误

  ```javascript
  Object.isExtensible(proxy) === Object.isExtensible(target)
  
  var p = new Proxy({}, {
      isExtensible: function(target) {
          return false;
      }
  });
  Object.isExtensible(p)
  // Uncaught TypeError: 'isExtensible' on proxy: trap result does not reflect extensibility of proxy target (which is 'true')
  ```

### 11.setPrototypeOf(target,proto)

> setPrototypeOf方法主要用来拦截Object.setPrototypeOf方法。

```javascript
var handler = {
    setPrototypeOf (target, proto) {
        throw new Error('Changing the prototype is forbidden');
    }
};
var proto = {};
var target = function () {};
var proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

* 注意，该方法只能返回布尔值，否则会被自动转为布尔值。
* 如果目标对象不可扩展（non-extensible），`setPrototypeOf`方法不得改变目标对象的原型。

### 12.apply(target, ctx, args)

> apply方法拦截函数的调用、call和apply操作
>
> apply方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组

```javascript
var twice = {
    apply (target, ctx, args) {
        return Reflect.apply(...arguments) * 2;
    }
};
function sum (left, right) {
    return left + right;
};
var proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30

Reflect.apply(proxy, null, [9, 10]) // 38
```

### 13.construct(target, args, newTarget)

> construct方法用于拦截new命令
>
> construct方法可以接受三个参数:目标对象、构造函数的参数对象、创造实例对象时，new命令作用的构造函数

```javascript
var p = new Proxy(function () {}, {
    construct: function(target, args) {
        console.log('called: ' + args.join(', '));
        return { value: args[0] * 10 };
    }
});
(new p(1)).value
// "called: 1"
// 10

```

* construct方法返回的必须是一个对象，否则会报错。

  ```javascript
  var p = new Proxy(function() {}, {
      construct: function(target, argumentsList) {
          return 1;
      }
  });
  new p() // 报错
  // Uncaught TypeError: 'construct' on proxy: trap returned non-object ('1')
  ```

## this问题

虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是**在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理 **：

```javascript
const target = {
    m: function () {
        console.log(this === proxy);
    }
};
const handler = {};
// 一旦proxy代理target.m，后者内部的this就是指向proxy，而不是target。
const proxy = new Proxy(target, handler);
target.m() // false
proxy.m()  // true
```

```javascript
const _name = new WeakMap();
class Person {
    constructor(name) {
        _name.set(this, name);
    }
    get name() {
        return _name.get(this);
    }
}
const jane = new Person('Jane');
jane.name // 'Jane'

//目标对象jane的name属性，实际保存在外部WeakMap对象_name上面，通过this键区分。由于通过proxy.name访问时，this指向proxy，导致无法取到值，所以返回undefined。
const proxy = new Proxy(jane, {}); //由于this指向的变化，导致 Proxy 无法代理目标对象
proxy.name // undefined
```

```javascript
// 有些原生对象的内部属性，只有通过正确的this才能拿到，所以 Proxy 也无法代理这些原生对象的属性。
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);
proxy.getDate();
// TypeError: this is not a Date object.
```

### 解决方法：

​	this绑定原始对象，就可以解决这个问题

```javascript
const target = new Date('2015-01-01');
const handler = {
    get(target, prop) {
        if (prop === 'getDate') {
            return target.getDate.bind(target);
        }
        return Reflect.get(target, prop);
    }
};
const proxy = new Proxy(target, handler);
proxy.getDate() // 1
```

## Proxy默认只代理一层对象的属性问题

> 解决办法是，在Reflect返回的时候，判断是否是一个对象，如果是对象的话，**再次用Proxy代理，返回代理对象**。

```js
const obj = {
    name: '张三',
    age: 20,
    xxx: { a:1 }
}
const handler = {
  get(target, key, receiver) {
    const ret = Reflect.get(target, key, receiver);
    
    return type ret === "object" ? new Proxy(ret, handler) : ret;
	}
}
new Proxy(obj, handler);
```

