### 非关系型数据库

优点:

- \- 高可扩展性
- \- 分布式计算
- \- 低成本
- \- 架构的灵活性，半结构化数据
- \- 没有复杂的关系

缺点:

- \- 没有标准化
- \- 有限的查询功能（到目前为止）
- \- 最终一致是不直观的程序

#### BASE是NoSQL数据库通常对可用性及一致性的弱要求原则:

- Basically Availble --基本可用
- Soft-state --软状态/柔性事务。 "Soft state" 可以理解为"无连接"的, 而 "Hard state" 是"面向连接"的
- Eventual Consistency -- 最终一致性， 也是 ACID 的最终目的。

#### 分类

- 列存储：**按列存储数据的。最大的特点是方便存储结构化和半结构化数据**
  - **Hbase**
  - Cassandra
  - Hypertable
- 文档存储：文档型内容
  - **MongoDB**
  - CouchDB 
- Key-Value型：
  - **Redis**
  - MemcacheDB
- 图存储
  - Neo4j
- 对象存储
- xml数据库



# MongoDB

#### 概念

- database：数据库
- collection (table)：数据库表/集合
  - 集合中的文档一般会有一定的逻辑联系
- document（row ）：数据行/文档，，文档是一组键值(key-value)对(即 BSON)。MongoDB 的文档**不需要设置相同的字段，并且相同的字段不需要相同的数据类型**
  - 文档中的键/值对是有序的
  - 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)
  - MongoDB的文档不能有重复的键
  - 文档的键是字符串
  - 文档键命名规范：
    - 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
    - .和$有特别的意义，只有在特定环境下才能使用。
    - 以下划线"_"开头的键是保留的(不是严格要求的)。
- field (column ):  数据字段/域
- index (index):  索引
- primary key：_id被自动设置为主键



**capped collections**

固定大小的集合，不是文档的数量固定，是总的大小空间固定，在磁盘中按照插入顺序存储，如果超过容量了就会删除最先插入的文档，此外，修改其中一个文档时，不能比原来的文档大。

```db.createCollection("mycoll", {capped:true, size:100000})```

然后也不能删除一个其中的文档，只能删除全部文档，capped collections最大为1e9个字节



#### **数据类型**

- String
- Integer
- Boolean
- Double
- Min/Max keys
- Array
- Timestamp
- Object 内嵌文档
- Null
- Symbol
- Date
  - 表示当前距离 Unix新纪元（1970年1月1日）的毫秒
- Object ID：创建文档的ID
  - ObjectId 类似唯一主键，可以很快的去生成和排序，包含 **12 bytes**，含义是：
    - 前 4 个字节表示创建 **unix** 时间戳,格林尼治时间 **UTC** 时间，比北京时间晚了 8 个小时,可以利用这个时间戳去获取创建时间
    - 接下来的 3 个字节是机器标识码
    - 紧接的两个字节由进程 id 组成 PID
    - 最后三个字节是随机数
- Binary Data
- Code：可以保存JS代码
- Regular expression



#### **一些基础命令**

```
#插入
db.col.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})

#查找
db.col.find()
{ "_id" : ObjectId("56064886ade2f21f36b03134"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }

#更新 前面是条件，后面是结果，也可以加上multi参数，表示全部更新
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})

#替换 
db.col.save({
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


#只更新第一条记录：
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
#全部更新：
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
#只添加第一条：
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
#全部添加进去:
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
#全部更新：
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
#只更新第一条记录：
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );

#删除文档，还可以加上justOne选项
db.col.remove({'title':'MongoDB 教程'})

#删除全部文档
db.col.remove({})

#查询 条件，返回的键
db.collection.find(query，projection)

#条件查询
db.col.find({"likes":{$lt:50}}).pretty()  
#AND条件查询
db.col.find({key1:value1, key2:value2}).pretty()
#OR条件查询
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
#复合使用查询条件
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()

#指定值类型的查询  {$type : 2}表示数据类型为string
db.col.find({"title" : {$type : 2}})

#指定返回数据记录文档数,以及跳过前N条记录
db.col.find().limit(NUMBER)
db.col.find().skip(NUMBER)
#排序 key表示键名，1表示升序排序，-1表示降序排序
db.COLLECTION_NAME.find().sort({KEY:1})
#skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 sort(), 然后是 skip()，最后是显示的 limit()。

#创建索引 1表示升序，-1表示降序，options可选是否唯一索引、默认语言、索引名等信息
db.collection.createIndex(keys, options)
db.col.createIndex({"title":1,"description":-1})

#聚合函数  这里num_tutorial相当于新建统计列的别名，它的意义是 {$sum : 1}，对相同的by_user求和
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
#还有很多的聚合函数
#{$avg : "$likes"}     {$min : "$likes"}      {$first : "$url"}


```



#### MongoDB的主从复制

一主一从、一主多从，主节点的每一个操作会被记录到oplog中，然后从节点定期轮询获取操作日志然后执行达到复制的作用。任意一个从节点都可以作为主节点。