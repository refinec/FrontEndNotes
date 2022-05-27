## 1. 注解描述接口

> 优点：程序员可以有一个参考
>
> 缺点：还是属于文档的范畴  ，这种方式太松散了 没有检查接口的方法是否完全被实现

```js
/**
* 	interface Composite {
* 		function add(obj);
* 		function remove(obj);
*		  function uopdate(obj);
*  }
*/


// CompositeImpl implements Composite
var CompositeImpl = function(){

};

CompositeImpl.prototype.add = function(obj){
  // do something ...
}
CompositeImpl.prototype.remove = function(obj){
  // do something ...
}			
CompositeImpl.prototype.update = function(obj){
  // do something ...
}

var c1 = new CompositeImpl();
var c2 = new CompositeImpl();
alert(c1.add == c2.add);

```

## 2. 属性检测接口

```js
/**
			 * 	interface Composite {
			 * 		function add(obj);
			 * 		function remove(obj);
			 *		function uopdate(obj);
			 *  }
			 *  
			 *  interface FormItem {
			 *  	function select(obj);
			 *  }
			 *  
			 */		
			 
// CompositeImpl implements Composite , FormItem
var CompositeImpl = function(){
  // 显示的再类的内部 接受所实现的接口
  // 一般来说是一个规范 我们项目经理:在类的内部定义一个数组(名字要固定)
  this.implementsInterfaces = ['Composite' ,'FormItem' ];
}	


CompositeImpl.prototype.add = function(obj){
  // do something ...
  alert('add...');
}
CompositeImpl.prototype.remove = function(obj){
  // do something ...
}			
CompositeImpl.prototype.update = function(obj){
  // do something ...
}			
CompositeImpl.prototype.select = function(obj){
  // do something ...
}		


// 检测CompositeImpl类的对象的
function CheckCompositeImpl(instance){
  //判断当前对象是否实现了所有的接口
  if(!IsImplements(instance,'Composite','FormItem')){
    throw new Error('Object does not implement a required interface!');
  }
}
			
// 公用的具体的检测方法(核心方法) 返回值类型 boolean
// 这个方法的主要目地：就是判断 实例对象 有没有实现相关的接口
function IsImplements(object){
  // arguments 对象 获得函数的实际参数
  for(var i = 1 ; i < arguments.length;i++){
    //接受所实现的每一个接口的名字
    var interfaceName = arguments[i];
    //判断此方法到底成功 还是失败啊
    var interfaceFound = false ;
    for(var j = 0 ; j <object.implementsInterfaces.length;j++){
      if(object.implementsInterfaces[j] == interfaceName)	{
        interfaceFound = true ;
        break;
      }
    }
    if(!interfaceFound){
      return false ; 
    }
  }
  return true ; 
}


var c1 = new CompositeImpl();
CheckCompositeImpl(c1);
c1.add();
```

## 3. 鸭式辩型接口

> 鸭式辨型法实现的核心：一个类实现接口的主要目的：把接口里的方法都实现（检测方法）
>
> 完全面向对象 代码也实现统一  也解耦了

一、接口类 Class Interface ==> 实例化N个接口

```js
/**
 * 接口类需要2个参数
 * 参数1: 接口的名字 (string)
 * 参数2: 接受方法名称的集合(数组) (array)
 */
var Interface = function(name, methods){
  //判断接口的参数个数
  if(arguments.length != 2){
    throw new Error('this instance interface constructor arguments must be 2 length!');
  }
  this.name = name ; 
  this.methods = [] ; //定义一个内置的空数组对象 等待接受methods里的元素(方法名字)
  for(var i = 0,len = methods.length ; i <len ; i++){
    if( typeof methods[i] !== 'string'){
      throw new Error('the Interface method name is error!');
    }
    this.methods.push(methods[i]);
  }
}
```

二、 准备工作

```js
// 1 实例化接口对象
var CompositeInterface = new Interface('CompositeInterface' , ['add' , 'remove']);
var FormItemInterface  = new Interface('FormItemInterface' , ['update','select']);
			
//  CompositeImpl implements CompositeInterface , FormItemInterface

// 2 具体的实现类 
var CompositeImpl = function(){

} 

// 3 实现接口的方法implements methods 			
CompositeImpl.prototype.add = function(obj){
  alert('add');
  // do something ...
}
CompositeImpl.prototype.remove = function(obj){
  alert('remove');
  // do something ...
}			
CompositeImpl.prototype.update = function(obj){
  alert('update');
  // do something ...
}	
CompositeImpl.prototype.select = function(obj){
  alert('select');
  // do something ...
}	
```

三、检验接口里的方法

```js
// 如果检验通过 不做任何操作 不通过：浏览器抛出error
// 这个方法的目的 就是检测方法的
Interface.ensureImplements = function(object){
  // 如果检测方法接受的参数小于2个 参数传递失败!
  if(arguments.length < 2 ){
    throw new Error('Interface.ensureImplements method constructor arguments must be  >= 2!');
  }

  // 获得接口实例对象 
  for(var i = 1 , len = arguments.length; i<len; i++ ){
    var instanceInterface = arguments[i];
    // 判断参数是否是接口类的类型
    if(instanceInterface.constructor !== Interface){
      throw new Error('the arguments constructor not be Interface Class');
    }
    // 循环接口实例对象里面的每一个方法
    for(var j = 0 ; j < instanceInterface.methods.length; j++){
      // 用一个临时变量 接受每一个方法的名字(注意是字符串)
      var methodName = instanceInterface.methods[j];
      // object[key] 就是方法
      if( !object[methodName] || typeof object[methodName] !== 'function' ){
        throw new Error("the method name '" + methodName + "' is not found !");
      }
    }
  }
}

var c1 = new CompositeImpl();
Interface.ensureImplements(c1,CompositeInterface,FormItemInterface);
c1.add();
```

四、继承方法

```js
const extend = function(sub, sup) {
  // 目的：实现只继承父类的原型对象
  const F = new Function(); // 1.创建一个空函数。 目的：空函数进行中转
  F.prototype = sup.prototype; // 2. 实现空函数的原型对象和超类的原型对象转换
  sub.prototype = new F(); // 3.原型继承
  sub.prototype.constructor = sub; // 4.还原子类的构造器
  sub.superClass = sup.prototype; // 自定义一个子类的的静态属性，接受父类的原型对象
  // 判断父类的原型对象的构造器（加保险）
  if(sup.prototype.constructor == Object.prototype.constructor) {
    sup.prototype.constructor = sup; // 手动设置父类原型对象的构造器
  }
}
```

