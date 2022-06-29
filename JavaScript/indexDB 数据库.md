# indexDB 数据库

> indexedDB就是浏览器提供的本地数据库，它可以被网页脚本创建和操作。indexedDB 允许储存大量数据，提供查找接口，还能建立索引。这些都是 LocalStorage 所不具备的。就数据库类型而言，indexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。

## 操作步骤

1. 打开数据库。
2. 在数据库中创建一个对象仓库（object store）。
3. 启动一个事务，并发送一个请求来执行一些数据库操作，像增加或提取数据等。
4. 通过监听正确类型的 DOM 事件以等待操作完成。
5. 在操作结果上进行一些操作（可以在 request 对象中找到）

## 打开数据库

```js
var db = null;
var request = window.indexedDB.open("MyTestDatabase");
request.onerror = function(event) {
  // 错误处理
  console.log(' 打开数据库报错');
};
request.onsuccess = function(event) {
  // 成功处理
  db = event.target.result;
  console.log('打开数据库成功');
};
```

## 创建和更新数据库版本号

如果指定的版本号，大于数据库的实际版本号，就会发生数据库升级事件`upgradeneeded`。这时通过事件对象的`target.result`属性，拿到数据库实例。

```js
var db = null;
request.onupgradeneeded = function (event) {
  db = event.target.result;
}
```

## 新建数据库

新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同的是，后续的操作主要在`upgradeneeded`事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。

新建数据库后，首先是新建对象仓库（即新建表）。

```js
request.onupgradeneeded = function(event) {
  db = event.target.result;
  var objectStore = null;
  if (!db.objectStoreNames.contains('imgLists')) {
    objectStore = db.createObjectStore('imgLists', { keyPath: 'id' });
    // unique name可能会重复
    objectStore.createIndex('name', 'name', { unique: false });
  }
}
```

## 写入数据

新增数据就是向对象仓库写入数据记录。这需要通过事务来完成。

```js
// new 一个blob对象
var obj1 = {hello: "world"};
var blob = new Blob([JSON.stringify(obj1, null, 2)], {type : 'application/json'});

function add() {
  var request = db.transaction(['imgLists'],  'readwrite')
    .objectStore('imgLists')
    .add({ id: 1, name: '图片1', path: '/static/image', blob:  blob});

  request.onsuccess = function (event) {
    console.log('数据写入成功');
  };

  request.onerror = function (event) {
    console.log('数据写入失败');
  }
}
```

## 查询数据

查询数据也通过事物来完成。

```js
function read() {
   var transaction = db.transaction(['imgLists']);
   var objectStore = transaction.objectStore('imgLists');
   // 用户读取数据，参数是主键
   var request = objectStore.get(1);

   request.onerror = function(event) {
     console.log('事务失败');
   };

   request.onsuccess = function( event) {
      if (request.result) {
        console.log(request.result);
      } else {
        console.log('未获得数据记录');
      }
   };
}
```

## 遍历数据

遍历数据表格的所有记录，要使用指针对象 IDBCursor。

```js
function readAll() {
  var objectStore = db.transaction('imgLists').objectStore('imgLists');

   objectStore.openCursor().onsuccess = function (event) {
     var cursor = event.target.result;

     if (cursor) {
       console.log(cursor);
       cursor.continue();
    } else {
      console.log('没有更多数据了！');
    }
  };
}
```

## 更新数据

```js
function update() {
  var request = db.transaction(['imgLists'], 'readwrite')
    .objectStore('imgLists')
    // 主动更新主键为1
    .put({ id: 1, name: '图片2',  path: '/static/image2'});

  request.onsuccess = function (event) {
    console.log('数据更新成功');
  };

  request.onerror = function (event) {
    console.log('数据更新失败');
  }
}
```

## 删除数据

```js
function remove() {
  var request = db.transaction(['imgLists'], 'readwrite')
    .objectStore('imgLists')
    .delete(1);

  request.onsuccess = function (event) {
    console.log('数据删除成功');
  };
}

remove();
```

## 创建/使用索引

索引的意义就是，可以搜索任意字段，也就是说从任意字段拿到数据记录。如果不建立索引的话，只能搜索主键（即从主键取值）。

```js
objectStore.createIndex('name', 'name', { unique: false });
```

```js
function findIndex() {
  var transaction = db.transaction(['imgLists'], 'readonly');
  var store = transaction.objectStore('imgLists');
  var index = store.index('name');
  var request = index.get('图片1');

  request.onsuccess = function (e) {
    var result = e.target.result;
    if (result) {
      console.log(result);
    } else {
      // ...
    }
  }
}
```

