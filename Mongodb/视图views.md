# Views

>  MongoDB视图是可查询的对象，其内容由其他集合或视图上的[聚合管道](https://mongodb.net.cn/manual/core/aggregation-pipeline/#id1)定义 。MongoDB不会将视图内容持久化到磁盘上。客户端[查询](https://mongodb.net.cn/manual/core/views/#views-supported-operations)视图时，将按需计算视图的内容。MongoDB可以要求客户端 [具有](https://mongodb.net.cn/manual/core/authorization/#authorization)查询视图的[权限](https://mongodb.net.cn/manual/core/authorization/#authorization)。MongoDB不支持针对视图的写入操作。

- 在员工数据到[`exclude`](https://mongodb.net.cn/manual/reference/operator/aggregation/project/#pipe._S_project)任何私人或个人信息（PII）的集合上创建一个视图 。应用程序可以在视图中查询不包含任何PII的员工数据。
- 在收集的传感器数据的集合上创建一个视图，以 [`add`](https://mongodb.net.cn/manual/reference/operator/aggregation/addFields/#pipe._S_addFields)计算字段和度量。应用程序可以使用简单的查找操作来查询数据。
- 创建一个视图，其中[`joins`](https://mongodb.net.cn/manual/reference/operator/aggregation/lookup/#pipe._S_lookup)两个集合分别包含库存和订单历史记录。应用程序可以查询联接的数据，而无需管理或了解底层的复杂管道。

当客户端[查询视图时](https://mongodb.net.cn/manual/core/views/#views-supported-operations)，MongoDB会将客户端查询追加到基础管道，并将该组合管道的结果返回给客户端。MongoDB可以将 [聚合管道优化](https://mongodb.net.cn/manual/core/aggregation-pipeline-optimization/)应用于组合管道。

## 创建视图

### db.createCollection()或者create()、db.createView()

```javascript
db.createCollection(
  "<viewName>",
  {
    "viewOn" : "<source>",
    "pipeline" : [<pipeline>],
    "collation" : { <collation> }
  }
)

# 创建集合
db.createCollection( <name>,
   {
     capped: <boolean>,
     autoIndexId: <boolean>,
     size: <number>,
     max: <number>,
     storageEngine: <document>,
     validator: <document>,
     validationLevel: <string>,
     validationAction: <string>,
     indexOptionDefaults: <document>,
     viewOn: <string>,              // Added in MongoDB 3.4
     pipeline: <pipeline>,          // Added in MongoDB 3.4
     collation: <document>,         // Added in MongoDB 3.4
     writeConcern: <document>
   }
)

```

```javascript
db.createView(<view>, <source>, <pipeline>, <options>)

db.createView(
  "<viewName>",
  "<source>",
  [<pipeline>],
  {
    "collation" : { <collation> }
  }
)
```

