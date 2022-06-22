# 顺序执行(Promise)

> 先getA然后getB执行，最后addAB

#### 1. (方法1)连续使用then链式操作

```javascript
function getA(){
  return new Promise(function(resolve, reject){ 
    setTimeout(function(){     
      resolve(2);
    }, 1000);
  });
}
 
function getB(){
  return new Promise(function(resolve, reject){       
    setTimeout(function(){
      resolve(3);
    }, 1000);
  });
}
 
function addAB(a,b){
    return a+b
}

function getResult(){
    var  obj={};
    Promise.resolve().then(function(){
        return  getA() 
    })
    .then(function(a){
         obj.a=a;
    })
    .then(function(){
        return getB() 
    })
    .then(function(b){
         obj.b=b;
         return obj;
    })
    .then(function(obj){
       return  addAB(obj['a'],obj['b'])
    })
    .then(data=>{
        console.log(data)
    })
    .catch(e => console.log(e));

}
getResult();
```

#### 2. (方法2)使用promise构建队列

```javascript
function getResult(){
    let res=[];
    // 构建队列
    function queue(arr) {
      let sequence = Promise.resolve();
      arr.forEach(function (item) {
        //sequence = sequence是为了在同一个队列中执行,另类的链式调用
        sequence = sequence.then(item).then(data=>{
            res.push(data);
            return res
        })
      })
      return sequence
    }

    // 执行队列
    queue([getA,getB]).then(data=>{
        return addAB(data[0],data[1])
    })
    .then(data => {
        console.log(data)
    })
    .catch(e => console.log(e));

}

getResult();

```

#### 3. (方法3)使用async、await实现类似同步编程

```javascript
function getResult(){
    async function queue(arr) {
        let res = []
        for (let fn of arr) {
            var data= await fn();
            res.push(data);
        }
        return await res
    }

    queue([getA,getB])
        .then(data => {
        return addAB(data[0],data[1])
    }).then(data=>console.log(data))

}

getResult();
```

