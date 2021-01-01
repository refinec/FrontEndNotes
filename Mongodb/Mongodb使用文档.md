# Mongodb

#### 概念解析

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| :----------- | :--------------- | :---------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档(JSON 对象)          |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。

MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

MongoDB提供高性能的数据持久性。特别是，

- 对嵌入式数据模型的支持减少了数据库系统上的I / O活动。
- **索引支持更快的查询**，并且可以包括来自嵌入式文档和数组的键。

#### 1. 启动和关闭数据库

```javascript
# 启动
// mongodb 默认使用执行 mongod 命令所处盘符根目录下的 /data/db 作为自己的数据存储目录
// 在第一次执行该命令之前先自己手动新建一个 /data/db
mongod 

# 修改默认的数据存储目录
mongod --dbpath=数据存储目录路径

# 停止
ctrl+C 
```

#### 2. 连接和退出数据库

```javascript
# 连接本机的 MongoDB 服务
mongo

# 在连接状态输入 exit 退出连接
exit
```

#### 3. 基本命令

> 在控制台使用以下命令

 * `show dbs`

   *查看显示所有数据库*

* `db`

  *查看当前操作的数据库*
* `use 数据库名`

  *如果数据库不存在，则创建数据库，否则切换到指定数据库*

* `db.dropDatabase()`

  使用`use`命令切换到要删除的数据库，再使用该命令删除当前数据库

* `db.createCollection(name, options)`

  创建集合，类似数据库中的表。

  - name: 要创建的集合名称

  - options: 可选参数, 指定有关内存大小及索引的选项

    | 字段        | 类型 | 描述                                                         |
    | :---------- | :--- | :----------------------------------------------------------- |
    | capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
    | autoIndexId | 布尔 | 3.2 之后不再支持该参数。（可选）如为 true，自动在 _id 字段创建索引。默认为 false。 |
    | size        | 数值 | （可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段。** |
    | max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

    在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

    在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。

    ```powershell
    > db.mycol2.insert({"name" : "菜鸟教程"})
    > show collections
    mycol2
    ```

* `db.collection.drop()`

  删除集合(表)。如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。

  ```powershell
  > use runoob
  switched to db runoob
  > db.createCollection("runoob")     # 先创建集合，类似数据库中的表
  > show tables             # show collections 命令会更加准确点
  runoob
  > db.runoob.drop()
  true
  > show tables
  > 
  ```

#### 4. MongoDB 在 bin 目录下提供了一系列有用的工具，这些工具提供了 MongoDB 在运维管理上 的方便。

| 工具                                                       | 描述                                                         |
| :--------------------------------------------------------- | :----------------------------------------------------------- |
| [mongosniff](https://www.mongodb.org.cn/manual/201.html)   | mongodb监测工具，作用类似于 tcpdump                          |
| [mongodump](https://www.mongodb.org.cn/manual/193.html)    | MongoDB数据备份工具                                          |
| [mongoimport](https://www.mongodb.org.cn/manual/197.html)  | Mongodb数据导入工具                                          |
| [mongoexport](https://www.mongodb.org.cn/manual/198.html)  | Mongodb数据导出工具                                          |
| [bsondump](https://www.mongodb.org.cn/manual/195.html)     | 将 bson 格式的文件转储为 json 格式的数据                     |
| [mongoperf](https://www.mongodb.org.cn/manual/202.html)    |                                                              |
| [mongorestore](https://www.mongodb.org.cn/manual/194.html) | MongoDB数据恢复工具                                          |
| [mongod.exe](https://www.mongodb.org.cn/manual/188.html)   | MongoDB服务启动工具                                          |
| [mongostat](https://www.mongodb.org.cn/manual/199.html)    | mongodb自带的状态检测工具                                    |
| [mongofiles](https://www.mongodb.org.cn/manual/203.html)   | GridFS 管理工具，可实现二制文件的存取                        |
| [mongooplog](https://www.mongodb.org.cn/manual/196.html)   |                                                              |
| [mongotop](https://www.mongodb.org.cn/manual/200.html)     | 跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据 |
| [mongos](https://www.mongodb.org.cn/manual/189.html)       | 分片路由，如果使用了 sharding 功能，则应用程序连接的是 mongos 而不是 mongod |
| [mongo](https://www.mongodb.org.cn/manual/190.html)        | 客户端命令行工具，其实也是一个 js 解释器，支持 js 语法       |