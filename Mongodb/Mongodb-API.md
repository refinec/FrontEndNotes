# API

## 插入文档(表记录)

### db.collection.insertOne()

```javascript
db.inventory.insertOne(
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
```



### db.collection.insertMany()

```
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

```powershell
#  插入多条数据
> var res = db.collection.insertMany([{"b": 3}, {'c': 4}])
> res
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("571a22a911a82a1d94c02337"),
                ObjectId("571a22a911a82a1d94c02338")
        ]
}
```

## 更新文档(表记录)

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

**参数说明：**

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

移除 title 为 'MongoDB 教程' 的文档：

```
>db.col.remove({'title':'MongoDB 教程'})
WriteResult({ "nRemoved" : 2 })           # 删除了两条数据
>db.col.find()
……                                        # 没有数据
```

------

如果你只想删除第一条找到的记录可以设置 justOne 为 1，如下所示：

```
>db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)
```

如果你想删除所有数据，可以使用以下方式（类似常规 SQL 的 truncate 命令）：

```
>db.col.remove({})
>db.col.find()
>
```

**remove() 方法已经过时了，现在官方推荐使用 deleteOne() 和 deleteMany() 方法**。

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

>  除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

```
db.collection.find(query, projection)
```

- **query** ：可选，使用查询操作符指定查询条件
- **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

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

