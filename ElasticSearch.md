ElasticSearch的本质是为了提高搜索效率，用到了倒排索引。

先说一下他的存储结构  

ES -> 索引（不同的分区)-> 类型（同种数据的集合） -> 文档（一个数据成员，就是一行）- >字段（属性）

**Index**：只是一个逻辑命名空间，内部还需要分片（shards）





倒排索引：

一个属性的其中一个值叫做一个term，建立倒排索引的时候会将term相同的数据成员的序号保存到一个数组中（Posting List）, 而不同的Term之间也会有被排序，被构造成一个二叉排序树（Term Dictionary）, 而且为了效率更高更节省空间，这颗二叉树的每个节点并不会直接保存一个完整的term而是Term的前缀。

然后对于一个term里的所有序号，也有方法来做压缩

- 增量编码压缩 ：不保存序号而是保存前后序号的差值，那样的话大数字也可以被转换为小数字

- 使用bit来存储，我们先将上面的结果分成若干个块，根据其中最大数字需要占的比特位来为这个块中每一个数分配同样的比特位，然后在开头使用一个字节来保存一下每个数字要用多少比特。这样的好处是不需要每个整数分配四个字节，会有很多冗余

- 就算你有了这个表之后，我还要使用bitmaps来进行修改：就是使用一个最大数对应长度的01串来表示是否有当前序号

  [1,3,4,7,10]

  对应的bitmap就是：

  [1,0,1,1,0,0,1,0,0,1]

- 这样还不够，万一数字还是太大，那么这个01串的长度是线性增长的，我们再使用类似redis的分槽来处理这个bitmap。



### API

- PUT语法：传入数据，但是需要带ID，不带ID的话就是POST方法

  ```json
  PUT /{index}/{type}/{id}
  {
    "field": "value",
    ...
  }
      
  PUT /website/blog/123
  {
    "title": "My first blog entry",
    "text":  "Just trying this out...",
    "date":  "2014/01/01"
  }
  ```
  
- GET 语法： 查询数据`GET /{_index}/{_type}/{_id}?pretty`
  
  


使用ES javaAPI创建查询以及聚类：

- 要创建一个聚类，对于不同类型的聚类要求，可以使用不同的聚类Buider，比如说如果要按照事件聚类，可以用DateHistogramAggregationBuilder（可以指定时间间隔），对于一般的聚类（即按照属性值聚类）可以用TermsAggretionBuilder，然后指定聚类的名字，然后指定聚类字段。
- 创建一个Query对象，使用QueryBuilder指定筛选属性等信息
- 创建一个SearchSourceBuilder，指定好Query和聚类的顺序
- 创建searchRequest对象，传入SearchSourceBuilder
- 使用连接执行searchRequest
- 如果有聚类，首先通过聚类名获得聚类出来的信息
- 然后getBucket，获得所有当中的每一个bucket，类型根据创建AggretionBuilder来
- 然后可以getKey获得这个桶所对应的同样的属性值
- getDocCount可以获得桶里元素的数量
- 然后使用上面两个值创建一个GroupingData对象，获得所有桶里的内容





### ES架构

底层：存储数据的文件系统，可以用多种文件系统去存储数据

往上：lucene框架

再往上：ES模块

传输层：一些传输协议  / JMX java管理框架

API层：RESTful API



#### ES集群：

节点类型，master，data ,coordinate 一个节点可以是一个或者多个节点类型

脑裂问题：因为主节点负载过大，从节点以为他死了，选出了另一个主节点。

- 解决方案：**discovery.zen.minimum_master_nodes:** (有master资格节点数/2) + 1  意思就是超半数具有选举资格的节点在场时才能进行选举。

另外还有分片，副本等保证性能及高可用

- 分片：是一个最小级别工作单位，每个shard就是一个lucene索引，需要单独分配内存，我们与index通信，内部还是去查找shard。文档存储在分片中，然后分片分配到集群中的节点上。当集群扩容或缩小，ES将会自动在节点间迁移分片，以使集群保持平衡。
  - 当一个节点接收到一个请求，他会找到这个文档处于哪个分片，并转发请求到该分片所在的节点中，主分片完成操作后，将请求发送到复制分片所在的节点中，等待复制分片完成操作并成功返回信息给主分片后，才返回成功信息给用户。可以设置后面的这个过程为异步的，但这可能会导致过载



映射（mapping）

> 一句话说完：就是插入数据的时候将某个字段映射为某种类型，不同的类型在进行倒排索引以及查询搜索时的行为是不一样的

查看某个索引的mapping http://192.168.20.114:9250/his_event_bqs/_mapping?pretty 

定义了一个文档（即一条记录）及其所包含的字段（属性）如何被存储和索引的方法

映射类型（mapping type）

每个索引（一张表）都有一个或者多个映射类型来对文档进行逻辑分组（即将文档分为若干个type）









直接看一段公司里的一个索引（表）的mapping

```json
dynamic_templates" : [   //说明这张表使用的是动态映射
          {
              //这张表中被判断属于为地图点这一数据类型的字段是"sys_geo_point"
            "geo_point_fields" : {
              "match" : "sys_geo_point",
              "mapping" : {
                "type" : "geo_point"
              }
            }
          },
          { //这张表中的"sys_date_time"字段被认为是日期，遵循一定的格式
            "date_fields" : {
              "match" : "sys_date_time",
              "mapping" : {
                "format" : "yyyy-MM-dd HH:mm:ss||epoch_millis",
                "type" : "date"
              }
            }
          },
          {
            "time_sort_fields" : {
              "match" : "sys_sort_time",
              "mapping" : {
                "type" : "integer"
              }
            }
          },
          {//其余所有的字段都是string类型，都是可以作为索引的，是keyword(只能精确搜索搜索)
            "string_fields" : {
              "match" : "*",
              "match_mapping_type" : "string",
              "mapping" : {
                "ignore_above" : 256,
                "index" : "true",
                "type" : "keyword"
              }
            }
          }
        ],
```





查看一个ES服务器的情况：curl ip:port/_cat/XXX?v



**搭建集群**

- 参考文档



https://blog.csdn.net/jiao_fuyou/article/details/50509941



#### 复杂API

- 在查找某一个文档时，返回的JSON文件在_source字段中是文档，另外还有版本信息 、ID信息等

```json
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

- 在查找所有文档时GET /megacorp/employee/_search   ，参数不为ID而是 _search，响应内容的`hits`数组中包含了我们所有的三个文档。默认情况下搜索会返回前10个结果

```json
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

- 按照条件查询`GET /megacorp/employee/_search?q=last_name:Smith`，返回结果仍在_hits中

- DSL语句查询：在JSON请求体中添加搜索条件

  ```json
  GET /megacorp/employee/_search
  {
      "query" : {
          "match" : {
              "last_name" : "Smith"
          }
      }
  }
  
  GET /megacorp/employee/_search
  {
      "query" : {
          //得以添加更多条件
          "filtered" : {
              "filter" : {
                  "range" : {
                      "age" : { "gt" : 30 } <1>
                  }
              },
              "query" : {
                  "match" : {
                      "last_name" : "smith" <2>
                  }
              }
          }
      }
  }
  ```



ES javaAPI传入查询条件过程

```json
 {
  "size" : 0,
  "query" : {
    "bool" : {
      "filter" : [
        {
          "range" : {
            "sys_date_time" : {
              "from" : 1568304000000,
              "to" : 1568908799999,
              "include_lower" : true,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        },
        {
          "term" : {
            "appId" : {
              "value" : "credit",
              "boost" : 1.0
            }
          }
        }
      ],
      "must_not" : [
        {
          "exists" : {
            "field" : "deviceId",
            "boost" : 1.0
          }
        }
      ],
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "aggregations" : {
    "SYS_group_field" : {
      "date_histogram" : {
        "field" : "sys_date_time",
        "format" : "yyyy-MM-dd",
        "time_zone" : "Asia/Shanghai",
        "interval" : "1d",
        "offset" : 0,
        "order" : {
          "_key" : "asc"
        },
        "keyed" : false,
        "min_doc_count" : 0
      }
    }
  }
}}
```









