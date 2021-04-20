## 1. app.set(name,value) 、app.get(name)

> **app.set(name,value)**把名字为name的项的值设为value，用于设置参数
>
> **app.get(name)** 获取名为name的项的值

```js
//设定端口
app.set('port', process.env.PORT || 3000);
//设定视图路径主要清楚__dirname的意思就可以了，它是node.js中的全局变量，表示取当前执行文件的路径     
app.set('views', path.join(__dirname,  'views' ));   
//设定视图引擎模板，还可以设定其他模板
app.set('view engine',  'jade' );
```

```js
if(app.get( 'env' ) ===  'development' ) {  
    app.use( function (err, req, res, next) { 
        res.status(err.status|| 500);       
        res.render( 'error' , {           
            message: err.message,  
            error: err        
        });   
    });
}
```

## 2. 使用app.use([path], function)和express.static公开指定目录

> 公开指定目录，可通过 /public/xx 的方式访问  public 目录中的所有资源

```javascript
app.use('/public/',express.static('./public/'))
app.use('/static/',express.static('./static/'))
app.use('/static',express.static(path.join(__dirname, 'public')))
```

```js
//下面代码表示当用户使用/访问时，调用routes，即routes目录下的index.js文件，
//其中.js后缀省略，用/users访问时，调用routes目录下users.js文件
var  routes = require( './routes/index' );
var  users = require( './routes/users' );
app.use( '/' , routes);
app.use( '/users' , users);
```

## 3. 文件操作路径和模块路径

> 文件操作路径:

```javascript
./data/a.txt //相对于当前目录
data/a.txt   //相对于当前目录
/data/a.txt  //绝对路径，当前文件模块所处的磁盘根目录
```

> 模块操作路径:

```javascript
// 这里如果忽略了，则也是磁盘根目录
require('/data/foo.js');

//相对路径
require('./data/foo.js');
```

## 4. 使用模板引擎时render函数的使用

> render 方法默认不可使用，但如果配置了模板引擎则可使用

```javascript
// 第一个参数不能写路径，默认去项目的views 目录查找该模板文件，Express约定所有视图文件都放到 views 目录中
res.render('html模板名',{模板数据});
res.render('404.html');
//修改默认的 views 目录
app.set('views', 'render函数的默径');
```

## 5. 中间件使用细节

> 同一个请求所经过的中间件都是同一个请求对象和响应对象。
>
> 中间件本身就是一个方法，该方法接收三个参数：
>
> * request 请求参数
> * response 响应对象
> * next 下一个中间件

* ### 中间件类型

  * 万能匹配(不关心任何请求路径和请方法)

  ```javascript
  app.use(function (req,res, next){
      ....
      next();
  })
  ```

  * 只是以 **'/xxx'**开头的路由中间件

  ```javascript
  app.use('/a',function(req,res,next){
  	...
  	next();
  })
  ```

* ### 错误中间件

  ```javascript
  app.get('/', function(req, res, next){
      fs.readFile('./1.txt', function(err, data){
          if(err){
              // 当调用 next 的时候，若传递了参数，则直接往后找到带有 4 个参数的应用程
              // 序级别中间件。当发生错误的的时候，可以调用 next 传递错误对象，
              // 然后就会被全局错误处理中间件匹配到并处理之。
              next(err);
          }
      })
  })
  
  // 配置错误处理中间件
  app.use(function (err, req, res, next){
      res.status(500).send(err.message);
  })
  
  app.listen(3000,()=>{
      console.log("running at port 3000.")
  })
  ```

  

