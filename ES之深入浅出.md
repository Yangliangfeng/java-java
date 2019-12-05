###  ElasticSearch 之深入浅出

* Elasticsearch介绍
```
1. Elasticsearch对复杂分布式机制的透明隐藏特性
   Elasticsearch是一套分布式的系统，分布式是为了应对大数据量,隐藏了复杂的分布式机制
   
   1）分片机制（将一些document插入到es集群中，不用关系数据怎么进行分片，数据分到哪个shard中去）
   2）cluster discovery（集群发现机制，当新起一个节点，这个节点会自动发现集群，并且加入了进去，还接受了部分数据）
   3）shard负载均衡（es会自动进行均匀分配，以保持每个节点的均衡的读写负载请求）
   4）shard副本，请求路由，集群扩容，shard重分配
   
2. Elasticsearch的垂直扩容与水平扩容
   1）垂直扩容：购买新的，性能更好的服务器，替换老的服务器
   2）水平扩容：购买普通服务器，加入到es的节点中去（企业一般采取水平扩容的方式）
   
3. 增减或减少节点时的数据rebalance，保持负载均衡
   总有一些节点的承载会重一些，承载的数据和请求量会大一些，当增加节点时，承载中的节点的shard会自动的分配
   到新加入的节点中去，会使shard重新达到平衡rebalance
  
4. master节点
   1）管理es集群的元数据
      创建和删除索引，维护索引的元数据，节点的增加和移除，维护集群的元数据
   2）默认情况下，会自动选择一台节点，作为master节点
   3）master节点不会承载所有的请求

5. 节点平等的分布式架构
   1) 节点对等，每个节点都能接收所有的请求
   2) 自动请求路由
   3) 响应收集
```
* shard&replica机制
```
1. index包含多个shard

2. 每个shard都是一个最小工作单元，承载部分数据，每个shard是一个lucene实例，具有完整的建立索引和处理请求的能力

3. 增减节点时，shard会自动在nodes中负载均衡

4. primary shard和replica shard，每个document肯定只存在于某一个primary shard以及其对应的replica shard中，

   不可能存在于多个primary shard

5. replica shard是primary shard的副本，负责容错，以及承担读请求负载

6. primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改

7. primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard

8. primary shard不能和自己的replica shard放在同一个节点上（否则节点宕机，primary shard和副本都丢失，
   起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上

```
* 横向扩容过程，如何超出扩容极限，如何提升容错性
```
1. primary&replica自动负载均衡，6个shard，3 primary，3 replica

2. 每个node有更少的shard，IO/CPU/Memory资源给每个shard分配更多，每个shard性能更好

3. 扩容的极限，6个shard（3 primary，3 replica），最多扩容到6台机器(此时，每台一个primary shard,
   
   一个replica shard)，每个shard可以占用单台服务器的所有资源，性能最好

4. 超出扩容极限，动态修改replica数量，9个shard（3primary，6 replica），扩容到9台机器，比3台机器时，拥有3倍的读吞吐量

5. 3台机器下，9个shard（3 primary，6 replica），资源更少，但是容错性更好，最多容纳2台机器宕机，6个shard只能容纳1台
   
   机器宕机

解析：6个shard（3 primary shard, 3 replica shard）,分布在三台机器上，为什么能容纳1台机器宕机？
      因为太宕机了，也就是一个primary shard没了，但是，在另外两台机器上，还有宕机的primary shard
      的副本，因此，还能保证数据的完整性。
     
```
* Elasticsearch容错机制
```
master选举 ---->  replica容错  -------> 数据恢复

前提：
   9 shard，3 node

1）master node宕机，其他健康的节点自动master选举，此时，集群的状态（cluster status : red），因为primary shard宕机了

2）replica容错：新master将replica提升为primary shard，此时，集群的状态（cluster status: yellow）,因为replica少了

3）重启宕机node，master copy replica到该node，使用原有的shard并同步宕机后的修改，green

```

* 简单的指令
```
1. 快速检查集群的健康状况
   GET /_cat/health?v
   1）green：每个索引的primary shard和replica shard都是active状态的
   2）yellow：每个索引的primary shard都是active状态的，但是部分replica shard不是active状态，处于不可用的状态
   3）不是所有索引的primary shard都是active状态的，部分索引有数据丢失了
   
说明： 由于默认的配置是给每个index分配5个primary shard和5个replica shard，而且primary shard和replica shard

   不能在同一台机器上（为了容错）

2. 快速查看集群中有哪些索引
   get /_cat/indices?v
   
3. 删除索引
   DELETE /test_index?pretty
   
4. 新增文档，建立索引
   PUT /index/type/id  
   例如：
   PUT /ecommerce/product/1
  {
      "name" : "gaolujie yagao",
      "desc" :  "gaoxiao meibai",
      "price" :  30,
      "producer" :      "gaolujie producer",
      "tags": [ "meibai", "fangzhu" ]
  }

5. 检索文档
   GET /index/type/id
   例如：
   GET /ecommerce/product/1
   
6. 更新文档
   POST /ecommerce/product/1/_update
  {
    "doc": {
      "name": "jiaqiangban gaolujie yagao"
    }
  }
  
7. 删除文档
   DELETE /ecommerce/product/1
```
* ES检索的六种方式
```
1. query string search ----> search参数都是以http请求的query string来附带的
   1) 搜索全部商品
      GET /ecommerce/product/_search
   
   2）搜索商品名称中包含yagao的商品，而且按照售价降序排序
      GET /ecommerce/product/_search?q=name:yagao&sort=price:desc
      
2. query DSL（特定领域的语言）
   1) 搜索全部商品
      GET /ecommerce/product/_search
      {
        "query": { "match_all": {} }
      }
   
   2）查询名称包含yagao的商品，同时按照价格降序排序
      GET /ecommerce/product/_search
      {
          "query" : {
              "match" : {
                  "name" : "yagao"
              }
          },
          "sort": [
              { "price": "desc" }
          ]
      }
      
    3）分页查询商品，总共3条商品，假设每页就显示1条商品，现在显示第2页，所以就查出来第2个商品
      GET /ecommerce/product/_search
      {
        "query": { "match_all": {} },
        "from": 1,
        "size": 1
      }
   
   4）指定要查询出来商品的名称和价格就可以
      GET /ecommerce/product/_search
      {
        "query": { "match_all": {} },
        "_source": ["name", "price"]
      }

3. query filter
   1）搜索商品名称包含yagao，而且售价大于25元的商品
      GET /ecommerce/product/_search
      {
          "query" : {
              "bool" : {
                  "must" : {
                      "match" : {
                          "name" : "yagao" 
                      }
                  },
                  "filter" : {
                      "range" : {
                          "price" : { "gt" : 25 } 
                      }
                  }
              }
          }
      }

4. full-text search（全文检索）
   GET /ecommerce/product/_search
   {
       "query" : {
           "match" : {
               "producer" : "yagao producer"
           }
       }
   }

5. phrase search（短语搜索）
   跟全文检索相对应，相反，全文检索会将输入的搜索串拆解开来，去倒排索引里面去一一匹配，只要能匹配上任意一个
   
   拆解后的单词，就可以作为结果返回phrase search，要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，
   
   才可以算匹配，才能作为结果返回
   GET /ecommerce/product/_search
   {
       "query" : {
           "match_phrase" : {
               "producer" : "yagao producer"
           }
       }
   }

6. highlight search（高亮搜索结果）
   GET /ecommerce/product/_search
   {
       "query" : {
           "match" : {
               "producer" : "producer"
           }
       },
       "highlight": {
           "fields" : {
               "producer" : {}
           }
       }
   }
```
* 嵌套聚合 | 下钻分析 | 聚合分析
```
0. 前提：
   在聚合查询时，将文本field的fielddata属性设置为true
   PUT /ecommerce/_mapping/product
   {
     "properties": {
       "tags": {
         "type": "text",
         "fielddata": true
       }
     }
   }
   
1. 计算每个tag下的商品数量
   GET /ecommerce/product/_search
   {
     "aggs": {
       "group_by_tags": {
         "terms": { "field": "tags"}
       }
     }
   }
   
2. 对名称中包含yagao的商品，计算每个tag下的商品数量
   GET /ecommerce/product/_search
   {
     "size" : 0,
     "query" : {
       "match": {
         "name": "yagao"
       }
     },
     "aggs": {
       "group_by_aggs": {
         "terms": {
           "field": "tags"
         }
       }
     }
   }

3. 先分组，再算每组的平均值，计算每个tag下的商品的平均价格
   GET /ecommerce/product/_search
   {
     "size" : 0,
     "aggs": {
       "group_by_tags": {
         "terms": {
           "field": "tags"
         },
         "aggs" : {
           "avg_price" : {
             "avg": {
               "field": "price"
             }
           }
         }
       }
     }
   }

4. 计算每个tag下的商品的平均价格，并且按照平均价格降序排序
   GET  ecommerce/product/_search
   {
     "size": 0,
     "aggs": {
       "group_by_tags": {
         "terms": {
           "field": "tags",
           "order": {
             "avg_price": "desc"
           }
         },
         "aggs": {
           "avg_price": {
             "avg": {"field": "price"}
           }
         }
       }
     }
   }

5. 按照指定的价格范围区间进行分组，然后在每组内再按照tag进行分组，最后再计算每组的平均价格
   GET /ecommerce/product/_search
   {
     "size":0,
     "aggs": {
       "group_by_price": {
         "range": {
           "field": "price",
           "ranges": [
             {
               "from": 0,
               "to": 20
             },
             {
               "from": 20,
               "to": 40
             },
             {
               "from": 40,
               "to": 50
             }
           ]
         },
         "aggs": {
           "group_by_tags": {
             "terms": {
               "field": "tags"
             },
             "aggs": {
               "avg_price": {
                 "avg": {
                   "field": "price"
                 }
               }
             }
           }
         }
       }
     }
   }
```
