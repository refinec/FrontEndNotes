# API

## 插入文档(表记录)

### db.collection.insertOne()

> 插入一条数据

**语法：**

```
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)
```

```javascript
const result = await db.inventory.insertOne(
   { 
       item: "canvas", 
       qty: 100, 
       tags: ["cotton"], 
       size: { h: 28, w: 35.5, uom: "cm" } 
   }
)

# 返回结果
{
        "acknowledged" : true,
        "insertedId" : ObjectId("571a218011a82a1d94c02333")
}

console.dir(result.insertedCount); // 插入数量1条
```

### db.collection.insertMany()

> 插入多条数据

**语法：**

```
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

```javascript
const result = await db.inventory.insertMany([
    { 
        item: "journal", 
        qty: 25, status: "A", 
        size: { h: 14, w: 21, uom: "cm" }, 
        tags: [ "blank", "red" ] 
    },
    { 
        item: "notebook", 
        qty: 50, status: "A", 
        size: { h: 8.5, w: 11, uom: "in" }, 
        tags: [ "red", "blank" ] 
    }
])
# 返回结果
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("571a22a911a82a1d94c02337"),
                ObjectId("571a22a911a82a1d94c02338")
        ]
}
console.dir(result.insertedCount); // 插入数量2条
```

**插入多个指定`_id`字段的文档**

值`_id`在集合中必须唯一，以避免重复的键错误。

```javascript
try {
   db.products.insertMany( [
      { _id: 10, item: "large box", qty: 20 },
      { _id: 11, item: "small box", qty: 55 },
      { _id: 12, item: "medium box", qty: 30 }
   ] );
} catch (e) {
   print (e);
}
# 该操作返回以下文档
{ "acknowledged" : true, "insertedIds" : [ 10, 11, 12 ] }

## 插入具有_id已经存在的值的文档将报错
try {
   db.products.insertMany( [
      { _id: 13, item: "envelopes", qty: 60 },
      { _id: 13, item: "stamps", qty: 110 },
      { _id: 14, item: "packing tape", qty: 38 }
   ] );
} catch (e) {
   print (e);
}
# 报错
BulkWriteError({
   "writeErrors" : [
      {
         "index" : 0,
         "code" : 11000,
         "errmsg" : "E11000 duplicate key error collection: inventory.products index: _id_ dup key: { : 13.0 }",
         "op" : {
            "_id" : 13,
            "item" : "stamps",
            "qty" : 110
         }
      }
   ],
   "writeConcernErrors" : [ ],
   "nInserted" : 1,
   "nUpserted" : 0,
   "nMatched" : 0,
   "nModified" : 0,
   "nRemoved" : 0,
   "upserted" : [ ]
})
```

请注意，第一个文档将成功插入，但第二个插入将失败。这也将阻止插入队列中剩余的其他文档。`_id: 13`

使用`ordered`to `false`，插入操作将继续处理所有剩余文档。

### db.collection.insert()

> 将一个或多个文档插入集合中。

```
db.collection.insert(
   <document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

| 参数           | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| `document`     | 要插入集合的文档或文档数组。                                 |
| `writeConcern` | 可选的。表达[书面关切的](https://mongodb.net.cn/manual/reference/write-concern/)文件。省略使用默认的写关注。请参阅[写关注点](https://mongodb.net.cn/manual/reference/method/db.collection.insert/#insert-wc)。如果在事务中运行，则不要为操作明确设置写关注点。要对事务使用写关注，请参见 [事务和写关注](https://mongodb.net.cn/manual/core/transactions/#transactions-write-concern)。 |
| `ordered`      | 可选的。如果为`true`，请按顺序在数组中插入文档，并且如果其中一个文档发生错误，则MongoDB将返回而不处理数组中的其余文档。如果为`false`，请执行无序插入，并且如果其中一个文档发生错误，请继续处理数组中的其余文档。默认为`true`。 |

## 更新文档(表记录)

### db.collection.updateOne（***filter***，***update***，***options* **）

```
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>        // Available starting in MongoDB 4.2.1
   }
)
```

**返回值**

- `matchedCount` 包含匹配文件的数量
- `modifiedCount` 包含修改后的文件数
- `upsertedId`包含`_id`上色文档的。
- 一个布尔值`acknowledged`，就`true`好像该操作在运行时带有 [写关注关系](https://mongodb.net.cn/manual/reference/glossary/#term-write-concern)或`false`是否禁用了写关注关系

### db.collection.updateMany（***filter***，***update***，***options* **）

```
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>        // Available starting in MongoDB 4.2.1
   }
)
```

```
db.collection.updateMany(
   <query>,
   [
      { $set: { status: "Modified", comments: [ "$misc1", "$misc2" ] } },
      { $unset: [ "misc1", "misc2" ] }
   ]
   ...
)
```

### db.collection.replaceOne（***filter***，***replace***，***options* **）

> 根据过滤器替换集合中的单个文档.替换匹配的集合中的第一个匹配的文件`filter`，使用该`replacement` 文件

```
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     hint: <document|string>                   // Available starting in 4.2.1
   }
)
```



### update() 方法

> update() 方法用于更新已存在的文档

**参数说明：**

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

```powershell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

```powershell
# 更新标题
>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })   # 输出信息
> db.col.find().pretty()
{
        "_id" : ObjectId("56064f89ade2f21f36b03136"),
        "title" : "MongoDB",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
>
```

以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true。

```powershell
>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

数据存在则更新：

```
db.collection.update({'title':'MongoDB 教程'}, {$set: {'title':'MongoDB'}})
```

数据存在时不进行操作：

```
db.collection.update({'title':'MongoDB 教程'}, {$setOnInsert: {'title':'MongoDB'}})
```

以上可扩展为多个字段(查询，更新)，如果数据不存在需要插入，设置 **upsert:true** 即可：

```
db.collection.update({'title':'MongoDB 教程'}, {$set: {'title':'MongoDB'}},{upsert:true})
```

* 移除集合中的键值对，使用的 **$unset** 操作符：`{ $unset: { <field1>: "", ... } }`

  ```powershell
  # 如果指定的字段不存在则操作不做任何处理。
  db.col.update({"_id":"56064f89ade2f21f36b03136"}, {$set:{ "test2" : "OK"}})
  db.col.find()
  
  db.col.update({"_id":"56064f89ade2f21f36b03136"}, {$unset:{ "test2" : "OK"}})
  db.col.find()
  ```

* **db.collection.updateOne() **向指定集合更新单个文档

* **db.collection.updateMany()** 向指定集合更新多个文档

  ```powershell
  # 1.在test集合里插入测试数据
  use test
  db.test_collection.insert( [
  {"name":"abc","age":"25","status":"zxc"},
  {"name":"dec","age":"19","status":"qwe"},
  {"name":"asd","age":"30","status":"nmn"},
  ] )
  
  # 2.更新单个文档
  > db.test_collection.updateOne({"name":"abc"},{$set:{"age":"28"}})
  { "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
  > db.test_collection.find()
  { "_id" : ObjectId("59c8ba673b92ae498a5716af"), "name" : "abc", "age" : "28", "status" : "zxc" }
  { "_id" : ObjectId("59c8ba673b92ae498a5716b0"), "name" : "dec", "age" : "19", "status" : "qwe" }
  { "_id" : ObjectId("59c8ba673b92ae498a5716b1"), "name" : "asd", "age" : "30", "status" : "nmn" }
  >
  
  # 3.更新多个文档
  > db.test_collection.updateMany({"age":{$gt:"10"}},{$set:{"status":"xyz"}})
  { "acknowledged" : true, "matchedCount" : 3, "modifiedCount" : 3 }
  > db.test_collection.find()
  { "_id" : ObjectId("59c8ba673b92ae498a5716af"), "name" : "abc", "age" : "28", "status" : "xyz" }
  { "_id" : ObjectId("59c8ba673b92ae498a5716b0"), "name" : "dec", "age" : "19", "status" : "xyz" }
  { "_id" : ObjectId("59c8ba673b92ae498a5716b1"), "name" : "asd", "age" : "30", "status" : "xyz" }
  >
  ```

  **异常：**

  ```
  WriteConcern.NONE:没有异常抛出
  WriteConcern.NORMAL:仅抛出网络错误异常，没有服务器错误异常
  WriteConcern.SAFE:抛出网络错误异常、服务器错误异常；并等待服务器完成写操作。
  WriteConcern.MAJORITY: 抛出网络错误异常、服务器错误异常；并等待一个主服务器完成写操作。
  WriteConcern.FSYNC_SAFE: 抛出网络错误异常、服务器错误异常；写操作等待服务器将数据刷新到磁盘。
  WriteConcern.JOURNAL_SAFE:抛出网络错误异常、服务器错误异常；写操作等待服务器提交到磁盘的日志文件。
  WriteConcern.REPLICAS_SAFE:抛出网络错误异常、服务器错误异常；等待至少2台服务器完成写操作。
  ```

  

### save() 方法

>  save() 方法通过传入的文档来替换已有文档，_id 主键存在就更新，不存在就插入。语法格式如下：

**参数说明：**

- **document** : 文档数据。
- **writeConcern** :可选，抛出异常的级别。

```
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

```powershell
# 替换了 _id 为 56064f89ade2f21f36b03136 的文档数据
>db.col.save({
    "_id" : ObjectId("56064f89ade2f21f36b03136"),
    "title" : "MongoDB",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Runoob",
    "url" : "http://www.runoob.com",
    "tags" : [
            "mongodb",
            "NoSQL"
    ],
    "likes" : 110
})

# 替换成功后，我们可以通过 find() 命令来查看替换后的数据
>db.col.find().pretty()
{
        "_id" : ObjectId("56064f89ade2f21f36b03136"),
        "title" : "MongoDB",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "Runoob",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "NoSQL"
        ],
        "likes" : 110
}
> 
```

### 更多实例

- 只更新第一条记录：

  `db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );`

- 全部更新：

  `db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );`

- 只添加第一条：

  `db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );`

- 全部添加进去:

  `db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );`

- 全部更新：

  `db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );`

- 只更新第一条记录：

  `db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );`

## 删除文档(表记录)

### db.inventory.deleteOne()

> 删除与过滤器匹配的第一个文档。使用作为[唯一索引](https://mongodb.net.cn/manual/reference/glossary/#term-unique-index)一部分的字段，例如`_id` 进行精确删除

```
db.collection.deleteOne(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)
```

| 参数           | 类型 | 描述                                                         |
| :------------- | :--- | :----------------------------------------------------------- |
| `filter`       | 文献 | 使用[查询运算符](https://mongodb.net.cn/manual/reference/operator/)指定删除条件。指定一个空文档以删除集合中返回的第一个文档。`{ }` |
| `writeConcern` | 文献 | 可选的。表达[书面关切的](https://mongodb.net.cn/manual/reference/write-concern/)文件。省略使用默认的写关注。如果在事务中运行，则不要为操作明确设置写关注点。要对事务使用写关注，请参见 [事务和写关注](https://mongodb.net.cn/manual/core/transactions/#transactions-write-concern)。 |
| `collation`    | 文献 | 可选的。指定 用于操作的[排序规则](https://mongodb.net.cn/manual/reference/bson-type-comparison-order/#collation)。[归类](https://mongodb.net.cn/manual/reference/collation/)允许用户为字符串比较指定特定于语言的规则，例如字母大写和重音符号的规则。排序规则选项具有以下语法：`collation: {   locale: <string>,   caseLevel: <boolean>,   caseFirst: <string>,   strength: <int>,   numericOrdering: <boolean>,   alternate: <string>,   maxVariable: <string>,   backwards: <boolean> } `指定排序规则时，该`locale`字段为必填字段；所有其他排序规则字段都是可选的。有关字段的说明，请参见[整理文档](https://mongodb.net.cn/manual/reference/collation/#collation-document-fields)。如果未指定排序规则，但是集合具有默认排序规则（请参阅参考资料[`db.createCollection()`](https://mongodb.net.cn/manual/reference/method/db.createCollection/#db.createCollection)），则该操作将使用为集合指定的排序规则。如果没有为集合或操作指定排序规则，则MongoDB使用先前版本中使用的简单二进制比较进行字符串比较。您不能为一个操作指定多个排序规则。例如，您不能为每个字段指定不同的排序规则，或者如果对排序执行查找，则不能对查找使用一种排序规则，而对排序使用另一种排序规则。3.4版的新功能。 |

**返回值：**

* 一个布尔值acknowledged，就true好像该操作在运行时带有 写关注关系或false是否禁用了写关注关系
* deletedCount 包含已删除文件的数量

### db.collection.deleteMany()

> 从集合中删除所有与匹配的文档

```
db.collection.deleteMany(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)
```

```javascript
try {
   db.orders.deleteMany(
       { "client" : "Crude Traders Inc." },
       { w : "majority", wtimeout : 100 }
   );
} catch (e) {
   print (e);
}
# 如果确认花费的时间超过`wtimeout`限制，则会引发以下异常：
WriteConcernError({
   "code" : 64,
   "errInfo" : {
      "wtimeout" : true
   },
   "errmsg" : "waiting for replication timed out"
})
```

如删除集合下全部文档：

```
db.inventory.deleteMany({})
```

删除 status 等于 A 的全部文档：

```
db.inventory.deleteMany({ status : "A" })
```

删除 status 等于 D 的一个文档：

```
db.inventory.deleteOne( { status: "D" } )
```

**remove() 方法 并不会真正释放空间**。

需要继续执行 db.repairDatabase() 来回收磁盘空间。

```
> db.repairDatabase()
或者
> db.runCommand({ repairDatabase: 1 })
```

## 查询文档(表记录)

### db.collection.findOne(query, projection)

### db.collection.find(query, projection)

- **query** ：可选，使用查询操作符指定查询条件。要返回集合中的所有文档，忽略此参数或传递一个空文档`{}`

- **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

  **该格式：**`{ field1: <value>, field2: <value> ... }`

  `<value>`可以是任何如下：

  - `1`或`true`将该字段包括在返回值中。
  - `0`或`false`排除该字段。

**如果你需要以易读的方式来读取数据，可以使用 pretty() 方法,pretty() 方法以格式化的方式来显示所有文档:**

```powershell
> db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}

db.inventory.find( { status: "D" } ); //返回其中statusfield等于"D"的文档
db.inventory.find( { qty: 0, status: "D" } ); //返回qtyfield等于0并且statusfield等于"D"
db.inventory.find( { "size.uom": "in" } ); //uom嵌套在size,并等于"in"
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" }}); //size字段等于文档：{ h: 14, w: 21, uom: "cm" }
db.inventory.find( { tags: "red" } ) // tags数组包含"red"为其元素之一
db.inventory.find( { tags: [ "red", "blank" ] } ) //返回该tags字段与指定数组完全匹配的文档，包括顺序
```

###  projection 参数的使用方法

```
// 其中projection中，入参格式为{columnA : 0/1,columnB : 0/1}
db.collection.find(query, projection)
```

columnA 和 columnB 表示你要查询的表中的字段，0/1 表示取或不取。

如：**{title:1}**，表示查询出的每条记录中只显示 title 字段内容。{description:0}，表示查询出的每条记录中不显示 description 字段内容(其他字段都展示)。

注：_id(主键)字段默认为 1，可指定 {_id:0} 来不输出 _id 字段值。

若不指定 projection，则默认返回所有键，指定 projection 格式如下，有两种模式

```
db.collection.find(query, {title: 1, by: 1}) // inclusion模式 指定返回的键，不返回其他键
db.collection.find(query, {title: 0, by: 0}) // exclusion模式 指定不返回的键,返回其他键
```

_id 键默认返回，需要主动指定 _id:0 才会隐藏

两种模式不可混用（因为这样的话无法推断其他键是否应返回）

```
db.collection.find(query, {title: 1, by: 0}) // 错误
```

只能全1或全0，除了在inclusion模式时可以指定_id为0

```
db.collection.find(query, {_id:0, title: 1, by: 1}) // 正确
```

若不想指定查询条件参数 **query** 可以 用 **{}** 代替，但是需要指定 **projection** 参数：

```
querydb.collection.find({}, {title: 1})
```

### MongoDB 与 RDBMS Where 语句比较

| 操作       | 格式                     | 范例                                        | RDBMS中的类似语句       |
| :--------- | :----------------------- | :------------------------------------------ | :---------------------- |
| 等于       | `{<key>:<value>`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`      |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`     |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`      |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`     |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50`     |

**qty 大于 50 小于 80 :**

```
db.posts.find( {  qty: { $gt: 50 ,$lt: 80}} )
```

### AND 条件

**MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件:**

```powershell
>db.col.find({key1:value1, key2:value2}).pretty()
# 类似于 WHERE 语句：WHERE by='菜鸟教程' AND title='MongoDB 教程'
> db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

### OR 条件

**OR 条件语句使用了关键字 $or **:

```powershell
>db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()

# 查询键 by 值为 菜鸟教程 或键 title 值为 MongoDB 教程 的文档
>db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
>
```

### AND 和 OR 联合使用

```powershell
# 类似常规 SQL 语句为： 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

