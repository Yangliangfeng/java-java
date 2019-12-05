###  ElasticSearch 之深入浅出

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
