###  ElasticSearch 之深入浅出

* Elasticsearch介绍
```
1. Elasticsearch对复杂分布式机制的透明隐藏特性
   Elasticsearch是一套分布式的系统，分布式是为了应对大数据量,隐藏了复杂的分布式机制
   
   1）分片机制（将一些document插入到es集群中，不用关心数据怎么进行分片，数据分到哪个shard中去）
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
* document_id的生成
```
1. 手动指定document id
   PUT /test_index/test_type/2
   {}
   适合：把数据导入到es中

2. 自动生成document id
   命令： post /index/type {}   ------>  AVp4RN0bhjxldOOnBxaE
   自动生成id的特点：长度为20个字符，URL安全，base64编码，GUID，分布式系统并行生成时不可能会发生冲突
   
```
* 定制返回的结果，指定_source中，返回哪些field
```
1. 返回指定的字段
   GET /test_index/test_type/1?_source=test_field1
   
2. 返回多个字段
   GET /test_index/test_type/1?_source=test_field1,test_field2
```
* document文档删除的原理
```
1. document的全量替换
   语法与创建文档是一样的，如果document id不存在，那么就是创建；如果document id已经存在，那么就是全量替换操作
   es会将老的document标记为deleted，然后新增我们给定的一个document，当我们创建越来越多的document的时候，es会
   在适当的时机在后台自动删除标记为deleted的document
   
2. document的删除
   DELETE /index/type/id
   不会理解物理删除，只会将其标记为deleted，当数据越来越多的时候，在后台自动删除
```
* ES的悲观锁和乐观锁
```
1. 悲观锁
   就是在各种情况下加锁，上锁之后，就只有一个线程可以操作这一条数据了
   
2. 乐观锁
   就是加入版本号的控制
   
3. 乐观锁和悲观锁的优缺点：
   1）悲观锁
      方便，直接加锁，不需要额外的操作
      但是并发能力低，同一个时间段内，只有一个线程能操作数据
   
   2） 乐观锁
      并发能力高，不给数据加锁，能支持大量线程的并发操作
      每次更新，都要先对比版本号，然后，可能需要重新数据，再次修改
```
* 基于_version进行乐观锁并发控制
```
1. document版本号的变化
   每次对这个document执行修改或者删除操作，它不是立即物理删除掉的，而是对这个_version版本号自动加1
   因此，它的一些版本号等信息还是保留着的
   
2. 乐观锁的并发控制
   当有线程来修改数据时，先进行版本号的控制，如果修改的数据和原有的数据版本号不一致，则修改的数据就被丢弃
```
* ES乐观锁并发控制实例
```
1. ES本身提供的version控制
   PUT /test_index/test_type/8?version=1
   {
     "test_field": "test"
   }
   
2. ES支持外部的version控制
   PUT /test_index/test_type/8?version=1&version_type=external
   {
     "test_field": "test"
   }

3. version与external version之间的区别：
   1）内部的version: 只有当你提供的version与es中的_version一模一样的时候，才可以进行修改，只要不一样，就报错
   2) 当version_type=external的时候，只有当你提供的version比es中的_version大的时候，才能完成修改
```
* partial update与全量替换
```
1. 全量替换的流程
   1）应用程序先发起一个get请求，获取到document，展示到前台界面，供用户查看和修改
   2）用户在前台界面修改数据，发送到后台
   3）后台代码，会将用户修改的数据在内存中进行执行，然后封装好修改后的全量数据
   4）发送PUT请求，到es中，进行全量替换
   5）es将老的document标记为deleted，然后重新创建一个新的document
   原理：查询，放界面，用户修改，可能花费的时间比较长，修改完，再写回去，可能es中的数据被修改了
        所以，并发冲突的情况会发生比较多
   
2. partial update
   1）用户发送修改的数据表单到后台
   2）后台代码，会将用户修改的数据在内存中进行执行，然后封装好修改后的partial update
   3）发送POST请求，到es中，进行partial update
   原理：查询，修改和写回都发生在shard内部，一瞬间就可以完成，可能基本就是毫秒级别
        所以，能大大减少并发冲突的情况
   
3.文档修改ES内部的操作
  ES内部对partial update的实际执行，跟传统的全量替换方式，几乎是一样的：
  1）首先获取document
  2）将传递过来的field更新到document的json中
  3）将老的document标记为delete
  4) 将修改的新的document创建出来
  
4. partial update相较于全量替换的优点：
   1）所有的查询，修改和写回操作，都发生在es中的一个shard内部，避免了所有的网络开销（减少2次网络请求）
      和数据传输，大大提升了性能
      
   2）减少了查询和修改中的时间间隔，可以有效的减少并发冲突的情况
```
* ES内置groovy脚本实现各种各样的复杂操作
```
1. 内置脚本
   例子：document中的num字段自增1
   post /test_index/test_type/11/_update
   {
     "script" : "ctx._source.num += 1"
   }
   
2. 外部脚本
   1）修改字段
   新建test-add-tags.groovy脚本
   脚本内容： ctx._source.tags += newtag
   post /test_index/test_type/11/_update
   {
     "script" : {
       "lang": "groovy",
       "file": "test-add-tags", //test-add-tags是外部脚本名称
       "params": {
         "newtag" : "tag1"
       }
     }
   }
   
   2）删除文档
   ctx.op = ctx._source.num == count ? 'delete' : 'none'

   POST /test_index/test_type/11/_update
   {
     "script": {
       "lang": "groovy",
       "file": "test-delete-document",
       "params": {
         "count": 1
       }
     }
   }
   
   3）upsert操作
   下面命令的含义是：
   如果num字段存在就自动加1，如果不存在，就执行upsert命令，新增文档
   post /test_index/test_type/11/_update
   {
     "script": "ctx._source.num += 1",
     "upsert" : {
       "num" : 0,
       "tags" : []
     }
   }
   
```
* mget批量查询
```
1. 情形一
   GET /_mget
   {
      "docs" : [
         {
            "_index" : "test_index",
            "_type" :  "test_type",
            "_id" :    1
         },
         {
            "_index" : "test_index",
            "_type" :  "test_type",
            "_id" :    2
         }
      ]
   }

2. 情形二
   GET /test_index/_mget
   {
      "docs" : [
         {
            "_type" :  "test_type",
            "_id" :    1
         },
         {
            "_type" :  "test_type",
            "_id" :    2
         }
      ]
   }
   
3. 情形三
   GET /test_index/test_type/_mget
   {
      "ids": [1, 2]
   }

特别说明：
   一次性要查询多条数据的话，那么一定要用batch批量操作的api
   尽可能减少网络开销次数，可能可以将性能提升数倍，甚至数十倍，非常非常之重要

```
* bulk语法
```
1. 基本格式
   POST /_bulk
   POST /_bulk
   { "delete": { "_index": "test_index", "_type": "test_type", "_id": "3" }} 
   { "create": { "_index": "test_index", "_type": "test_type", "_id": "12" }}
   { "test_field":    "test12" }
   { "index":  { "_index": "test_index", "_type": "test_type", "_id": "2" }}
   { "test_field":    "replaced test2" }
   { "update": { "_index": "test_index", "_type": "test_type", "_id": "1", "_retry_on_conflict" : 3} }
   { "doc" : {"test_field2" : "bulk test1"} }
   格式说明：
      每一个操作要两个json串，语法如下：
      {"action": {"metadata"}}
      {"data"}

2. bulk操作中，任意一个操作失败，是不会影响其他的操作的，但是在返回结果里，会告诉你异常日志

3. bulk size最佳大小
   bulk request会加载到内存里，如果太大的话，性能反而会下降，因此需要反复尝试一个最佳的bulk size。
   
   一般从1000~5000条数据开始，尝试逐渐增加。另外，如果看大小的话，最好是在5~15MB之间。

```
* 路由原理
```
1. document路由到shard上原理
   一个index的数据会被分成多片，每一片都在一个shard中。所以说，一个document，只能存在于一个shard中。
   当客户端创建document的时候，es此时就需要决定说，这个document是放在这个index的哪个shard上。这个过程
   称为document routing，数据路由。
   
2. 路由算法
   shard = hash(routing) % number_of_primary_shard

3. 是选择_id 还是 custom routing value
   默认routing 的值是 _id
   也可以在发送请求的时候，手动指定一个routing value，比如说 put /index/type/id?routing=user_id
   手动指定routing value是很有用的，可以说，某一类document一定呗路由到一个shard上去，那么在后续
   进行应用级别的负载均衡以及提升批量读取的性能的时候，是很有帮助的。
   
4. primary shard 数量不可变原理
   原理是：document被分配到哪个shard中，是根据hash(routing)来计算的；如果primary shard发生变化，
   则会导致查询的数据丢失。所以，primary shard一旦index建立，是不允许修改的。但是，replica shard是
   可以随时修改的。
```
* document增删改的内部
```
步骤：
   1）客户端选择一个node发送请求过去，这个node就是coordinating node（协调节点）
   
   2）coordinating node，对document进行路由，将请求转发给对应的node的primary shard进行处理
      replica shard不能处理数据
   
   3）实际的node上的primary shard处理请求，然后将数据同步到replica node
   
   4）coordinating node，如果发现primary node和所有replica node都搞定之后，就返回响应结果给客户端
```
* 写一致性原理和quorum机制
```
0. put /index/type/id?consistency=quorum

1. consistency一致性的三种状态
   1）one（primary shard）
      写操作，只要有一个primary shard是active活跃可用的，就可以执行
      
   2）all（all shard）
      写操作，必须所有的primary shard和replica shard都是活跃的，才可以执行这个写操作
      
   3）quorum（default）
      默认的值，要求所有的shard中，必须是大部分的shard都是活跃的，可用的，才可以执行这个写操作

2. quorum机制
   原理：
   写之前必须确保大多数shard都可用，int( (primary + number_of_replicas) /2) + 1，当number_of_replicas>1时才生效
   number_of_replicas = 1 是不生效的。
   
3. 如果节点数少于quorum数量，可能导致quorum不齐全，进而导致无法执行任何写操作
  
4. quorum不齐全时，wait，默认1分钟，timeout，100，30s
   在写操作的时候，加一个timeout参数，比如说put /index/type/id?timeout=30
```
* document内部原理
```
1. 客户端发送请求到任意一个node，成为coordinate node

2. coordinate node对document进行路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，

   在primary shard以及其所有replica中随机选择一个，让读请求负载均衡
   
3. 接收请求的node返回document给coordinate node

4. coordinate node返回document给客户端

5. 特殊情况：document如果还在建立索引过程中，可能只有primary shard有，任何一个replica shard都没有，

   此时可能会导致无法读取到document，但是document完成索引建立之后，primary shard和replica shard就都有了
```
* deep paging性能问题
```
1. 概念
   deep paing简单来说，就是搜索的特别深。比如说，有60000条数据，每个shard上有20000条，每页10条，这个时候，
   
   你要搜索第1000页。实际上是把每个shard内部的20000条数据中的10001-10010条数据拿出来，不是10条，而是
   
   10010条数据。所以，3个shard，每个shard返回10010条数据给coordinate node节点，总共30030条数据，然后，按照
   
   _score分数，对这些数据进行排序
   
2. 性能问题
   搜索的过深，就需要在coordinate node节点上保存大量的数据，还要进行大量的数据排序，排序之后，再取出
   
   那一页，所以，这个过程耗费网络带宽，耗费内存，还耗费cpu，所以，deep paging会导致性能问题
```
* query 分词
```
1. 分词原理
   1）query string必须以和index建立时相同的analyzer进行分词
   2）query string对exact value和full text的区别对待
      date：exact value  ----> 精确匹配
      _all：full text    ----> 全文搜索
      
2. 不同类型的field，可能有的就是full text，有的就是exact value
   post_date，date：exact value
   _all：full text，分词，normalization
   
3. GET /_search?q=2017 搜索的是_all field，document所有的field都会拼接成一个大串，进行分词

4. 测试分词器，查看是怎么分词的
   GET /_analyze
   {
     "analyzer": "standard",
     "text": "Text to analyze"
   }
```
* mapping 的原理
```
1. 往es里面直接插入数据，es会自动建立索引，同时建立type以及对应的mapping

2. mapping中就自动定义了每个field的数据类型

3. 不同的数据类型（比如说text和date），可能有的是exact value，有的是full text

4. exact value，在建立倒排索引的时候，分词的时候，是将整个值一起作为一个关键词建立到倒排索引中的；

   full text，会经历各种各样的处理，分词，normaliztion（时态转换，同义词转换，大小写转换），才会建立到倒排索引中
   
5. exact value和full text类型的field就决定了，在一个搜索过来的时候，对exact value field或者是full text field进行

   搜索的行为也是不一样的，会跟建立倒排索引的行为保持一致；比如说exact value搜索的时候，就是直接按照整个值进行匹配，
   
   full text query string，也会进行分词和normalization再去倒排索引中去搜索
   
6. dynamic mapping 
   1) true or false	-->	boolean  转义
   2）123		-->	long
   3）123.45		-->	double
   4）2017-01-01	-->	date
   5）"hello world"	-->	string/text
   
7. 查看某个索引的mapping
   GET /index/_mapping/type
```
* filter与query对比
```
1. filter，仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，对相关度没有任何影响

2. query，会去计算每个document相对于搜索条件的相关度，并按照相关度进行排序

3. 一般来说，如果你是在进行搜索，需要将最匹配搜索条件的数据先返回，那么用query；如果你只是要根据一些条件筛

   选出一部分数据，不关注其排序，那么用filter除非是你的这些搜索条件，你希望越符合这些搜索条件的document越排
   
   在前面返回，那么这些搜索条件要放在query中；如果你不希望一些搜索条件来影响你的document排序，那么就放在filter
   
   中即可

4. filter与query性能
   1) filter，不需要计算相关度分数，不需要按照相关度分数进行排序，同时还有内置的自动cache最常使用filter的数据
   2) query，相反，要计算相关度分数，按照分数进行排序，而且无法cache结果
```
* 多条件组合查询
```
   bool的子条件组合下面可以有：
   must， must_not， should， filter
   must: 必须匹配
   must_not: 必须不匹配
   should: 可以匹配其中任意一个
   term: 精确匹配
   terms: 匹配多个；相当于mysql中的in
   每个子查询都会计算一个document针对它的相关度分数，然后bool综合所有分数，合并为一个分数，当然filter是不会计算分数的
   
   query term: 精确匹配exact value
   query match: 全文检索 full text
   
   全文检索：
     进行多个值的检索，有两种做法：
       1. match query；
       2. should
     控制搜索结果精准度：
         and operator，minimum_should_match
```
* 普通match底层为term的过程
```
1. 普通match如何转换为term+should

{
    "match": { "title": "java elasticsearch"}
}

使用诸如上面的match query进行多值搜索的时候，es会在底层自动将这个match query转换为bool的语法
bool should，指定多个搜索词，同时使用term query
{
  "bool": {
    "should": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }}
    ]
  }
}

2. and match如何转换为term+must
   {
    "match": {
        "title": {
            "query":    "java elasticsearch",
            "operator": "and"
        }
    }
}

{
  "bool": {
    "must": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }}
    ]
  }
}

3. minimum_should_match如何转换
   {
    "match": {
        "title": {
            "query":                "java elasticsearch hadoop spark",
            "minimum_should_match": "75%"
        }
    }
}

{
  "bool": {
    "should": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }},
      { "term": { "title": "hadoop" }},
      { "term": { "title": "spark" }}
    ],
    "minimum_should_match": 3 
  }
}
```
* 搜索条件的权重之boost
```
搜索条件的权重，boost，可以将某个搜索条件的权重加大，此时当匹配这个搜索条件和匹配另一个搜索条件的document，

计算relevance score时，匹配权重更大的搜索条件的document，relevance score会更高，当然也就会优先被返回回来

默认情况下，搜索条件的权重都是一样的，都是1
```
* 多shard场景下relevance score不准确问题大揭秘
```
1. 问题：
   在多个shard场景下，有时候导致出现的搜索结果，似乎不是你太想要的结果，也许会出现相关度很高的doc排在了后面
   ，分数不搞。而相关度很低的doc排在了前面，分数很高。

2. 出现的原因分析：
   在某一个shard中，有很多的document，包含了title中有java这个关键字，比如说10个doc包含了java；当一个搜索title
   包含java的请求，到这个shard的时候，应该会这么计算relevance score分数，TF/IDF算法：
  （1）在一个doc的title中java出现了几次
  （2）在所有的doc的title的java出现了几次
  （3）这个doc的title的长度
   
   shard中只是一部分doc，默认，就在shard local本地计算IDF
   
   在另外一个shard中，只有1个document title包含java，此时，计算shard local IDF，就会分数很高，相关度很高
   
3.解决措施：
  （1）在生产环境中，数据量很大，尽可能实现均匀分配
  （2）测试环境下，将索引的primary shard设置为1个，number_of_shards=1，index settings
      如果说只有一个shard，那么当然，所有的document都在这个shard里面，就没有这个问题了
      
   （3）测试环境下，搜索附带search_type=dfs_query_then_fetch参数，会将local IDF取出来计算global IDF
      计算一个doc的相关度分数的时候，就会将所有shard对的local IDF计算一下，获取出来，在本地进行global IDF
      分数的计算，会将所有shard的doc作为上下文来进行计算，也能确保准确性。但是production生产环境下，不推荐
      这个参数，因为性能很差。

```
* best fields策略
```
1. best fields策略，就是说，搜索到的结果，应该是某一个field中匹配到了尽可能多的关键词，被排在前面；而不是尽可能多

的field匹配到了少数的关键词，排在了前面

dis_max语法，直接取多个query中，分数最高的那一个query的分数即可

2. tie_breaker  将其他query的分数，乘以tie_breaker，然后综合与最高分数的那个query的分数，综合在一起进行计算

除了取最高分以外，还会考虑其他的query的分数

tie_breaker的值，在0~1之间，是个小数

3. multi_match
   如：
   GET /forum/article/_search
   {
     "query": {
       "multi_match": {
           "query":                "java solution", //匹配的内容
           "type":                 "best_fields", //搜索的策略
           "fields":               [ "title^2", "content" ], //"test^2" 表示权重boost的值为2
           "tie_breaker":          0.3, //表示考虑其他query的分数的比重
           "minimum_should_match": "50%"  //控制搜索结果的精准度，只有匹配一定数量的关键词的数据，才能返回
       }
     } 
   }
```
* most-fields 策略
```
1. 定义
   most-fields策略，主要是说尽可能返回更多field匹配到某个关键词的doc，优先返回
   
   没办法用minimum_should_match去掉长尾数据，就是匹配的特别少的结果
   
2. enligsh analyzer分词器
   会将单词还原为其最基本的形态，stemmer，去掉单词所有的形态变化，回到单词本身
   
3. best-fields与most-fields的区别
   1）best-fields：
      （1）是对多个field进行搜索，挑选某个field匹配度最高的那个分数，同时在多个query最高分相同的情况下，在一定程
           度上考虑其他query的分数
      （2）优点：通过best_fields策略，以及综合考虑其他field，还有minimum_should_match支持，可以尽可能精准地将匹
           配的结果推送到最前面
      （3）缺点：除了那些精准匹配的结果，其他差不多大的结果，排序结果不是太均匀，没有什么区分度了
   
   2）most_fields：
      （1）综合多个field一起进行搜索，尽可能多地让所有field的query参与到总分数的计算中来
      （2）优点：将尽可能匹配更多field的结果推送到最前面，整个排序结果是比较均匀的
      （3）缺点：可能那些精准匹配的结果，无法推送到最前面
```
* copy-to
```
用copy_to，将多个field组合成一个field,就可以将多个字段的值拷贝到一个字段中，并建立倒排索引

PUT /forum/_mapping/article
{
  "properties": {
      "new_author_first_name": {
          "type":     "string",
          "copy_to":  "new_author_full_name" 
      },
      "new_author_last_name": {
          "type":     "string",
          "copy_to":  "new_author_full_name" 
      },
      "new_author_full_name": {
          "type":     "string"
      }
  }
}

原生的muti-match支持cross-fields
GET /forum/article/_search
{
  "query": {
    "multi_match": {
      "query": "Peter Smith",
      "type": "cross_fields",
      "operator": "and",
      "fields": ["author_first_name", "author_last_name"]
    }
  }
}
```
* 几个比较重要的概念
```
0. 近似匹配
   phrase match加上 slop就是proximity match 近似匹配
0-1. slop的含义
   query string，搜索文本中的几个term，要经过几次移动才能与一个document匹配，这个移动的次数，就是slop。
   
1. 召回率：
   比如你搜索一个java spark，总共有100个doc，能返回多少个doc作为结果，就是召回率，recall
   
2. 精准度
   比如你搜索一个java spark，能不能尽可能让包含java spark，或者是java和spark离的很近的doc，排在最前面，precision
   
3. 优先满足召回率
   意思是比如搜索java spark，包含java的能返回，包含spark也能返回，包含java和spark的也返回；
   同时兼顾精准度，就是包含java和spark，同时java和spark离的越近的doc排在前面
   此时可用match query和match phrase query一起使用
   GET /forum/article/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "title": {
                 "query":"java spark"
               }
             }
           }
         ],
          "should" : {
         "match_phrase": {
           "title" : {
             "query" : "java spark",
             "slop" : 50
           }
         }
       }

       }
     }
   }
```
* 定位不合法的搜索原因
```
1. 使用validate的API
   GET /test_index/test_type/_validate/query?explain
   {}
   一般用在那种特别复杂庞大的搜索下，比如你一下子写了上百行的搜索，这个时候可以先用validate api去验证一下，搜索是否合法
```
* TF/IDF算法
```
0. Elasticsearch使用的是 term frequency/inverse document frequency算法，简称为TF/IDF算法

1. Term frequency
   搜索文本中的各个词条在field文本中出现了多少次，出现次数越多，就越相关
   
2. Inverse document frequency
   搜索文本中的各个词条在整个索引的所有文档中出现了多少次，出现的次数越多，就越不相关

```
* doc values
```
0. 搜索的时候，要依靠倒排索引,排序的时候，需要依靠正排索引，看到每个document的每个field，然后进行排序，

   所谓的正排索引，其实就是doc values

1. 在建立索引的时候，一方面会建立倒排索引，以供搜索用;一方面会建立正排索引，也就是doc values，以供排序，

   聚合，过滤等操作使用

2. doc values是被保存在磁盘上的，此时如果内存足够，os会自动将其缓存在内存中，性能还是会很高；如果内存不足够，

   os会将其写入磁盘上
```
* scoll技术滚动搜索大量数据
```
   如果一次性要查出来比如10万条数据，那么性能会很差，此时一般会采取用scoll滚动查询，一批一批的查，直到所有数据都查询

完处理完。使用scoll滚动搜索，可以先搜索一批数据，然后下次再搜索一批数据，以此类推，直到搜索出全部的数据来scoll搜索会在

第一次搜索的时候，保存一个当时的视图快照，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的

采用基于_doc进行排序的方式，性能较高每次发送scroll请求，我们还需要指定一个scoll参数，指定一个时间窗口，每次搜索请求只要

在这个时间窗口内能完成就可以了。

GET /test_index/test_type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": [ "_doc" ],
  "size": 3
}
下一次搜索：
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "xxxxxx"
}
```
* bouncing results问题
```
1. preference
   决定了哪些shard会被用来执行搜索操作
   _primary, _primary_first, _local, _only_node:xyz, _prefer_node:xyz, _shards:2,3
   
2. bouncing results问题
   两个document排序，field值相同；不同的shard上，可能排序不同；每次请求轮询打到不同的replica shard上；每次页面上看
   
   到的搜索结果的排序都不一样。这就是bouncing result，也就是跳跃的结果。搜索的时候，是轮询将搜索请求发送到每一个
   
   replica shard（primary shard），但是在不同的shard上，可能document的排序不同
   
3. 解决方案
   将preference设置为一个字符串，比如说user_id，让每个user每次搜索的时候，都使用同一个replica shard去执行，
   
   就不会看到bouncing results了
```
* 倒排索引的结构
```
1. 倒排索引，是适合用于进行搜索的

2. 倒排索引的结构
   1) 包含这个关键词的document list
   2) 包含这个关键词的所有document的数量：IDF（inverse document frequency）
   3) 这个关键词在每个document中出现的次数：TF（term frequency）
   4) 这个关键词在这个document中的次序
   5) 每个document的长度：length norm
   6) 包含这个关键词的所有document的平均长度
   
3. 倒排索引不可变的好处
   1) 不需要锁，提升并发能力，避免锁的问题
   2) 数据不变，一直保存在os cache中，只要cache内存足够
   3) filter cache一直驻留在内存，因为数据不变
   4) 可以压缩，节省cpu和io开销
   
4. 倒排索引不可变的坏处：每次都要重新构建整个索引
```
* document写入原理
```
1. 步骤：
   1） document会首先被写入内存buffer缓冲

   2） 每隔一定时间间隔，执行commit point操作

   3） buffer中的数据写入新的index segment

   4） index segment会被写入os cache缓冲中，等待在os cache中的index segment被fsync强制刷到磁盘上

   5） 新的index sgement被打开，供search使用

   6） buffer被清空

2. 每次commit point时，会有一个.del文件，标记了哪些segment中的哪些document被标记为deleted了

3. 搜索的时候，会依次查询所有的segment，从旧的到新的，比如被修改过的document，在旧的segment中，

   会标记为deleted，在新的segment中会有其新的数据
```
* ES数据写入流程
```
1. 将document写入buffer缓冲，同时，将document写入translog日志文件

2. 每隔1秒钟，数据从buffer写入index segment file文件中；然后，segment写入os cache，写入os cache后，
   内存的buffer就被清空。进入os cache后，就被打开供search使用。search请求就可以搜索到这个os cache中的
   index segment file。
   
3. 随着时间的推移，translog文件会不断的变大，当大到一定程度，就会触发flush操作

4. 执行commit；将buffer现有的数据全部写入segment file,并刷入os cache，打开供search使用

5. 写一个commit point 到磁盘中，标明有哪些Index segment

6. 用fsync将数据强行刷到os disk磁盘上去，所有os cache的数据都会被刷入磁盘

```
* 数据的恢复
```
问题：
   os cache中囤积了一些数据，但是，此时，不巧，宕机了，os cache中的数据全部丢失，那么，怎么恢复数据
   
解决：
   丢失之前，translog就存储了上一次flush（commit point）直到现在最近的数据的变更记录；os disk上存放了
   上一次commit point为止，所有的segment file都fsync到了磁盘上；
   
   机器被重启，disk上的数据并没有丢失，此时，就会将translog文件中的变更记录进行回放，重新执行之前的各种
   操作，在buffer中执行，再重新刷新一个一个的segment到os cache中，等待下一次的commit发生即可。
```
* 对相关度评分调节和优化的常见4种方法
```
1. query-time boost
   GET /forum/article/_search
   {
     "query": {
       "bool": {
         "should": [
           {
             "match": {
               "title": {
                 "query": "java spark",
                 "boost": 2  //对某个查询内容增加权重
               }
             }
           },
           {
             "match": {
               "content": "java spark"
             }
           }
         ]
       }
     }
   }
   
2. 重构查询结构
3. negative boost
   搜索包含java，不包含spark的doc，但是这样子很死板搜索包含java，尽量不包含spark的doc，如果包含了spark，
   
   不会说排除掉这个doc，而是说将这个doc的分数降低包含了negative term的doc，分数乘以negative boost，分数降低

   GET /forum/article/_search
   {
     "query": {
         "boosting": {
           "positive": {
             "match": {
               "content": "java"
             }
           },
           "negative": {
             "match": {
               "content": "spark"
             }
           },
           "negative_boost": 0.2
         }
     }
   }

4. constant_score
   如果你压根儿不需要相关度评分，直接走constant_score加filter，所有的doc分数都是1，没有评分的概念了
   
```
* IK中文分词器
```
1. ik_max_word: 会将文本做最细粒度的拆分

2. ik_smart: 会做最粗粒度的拆分
   
   一般选用ik_max_word
```
* 聚合分析的概念解释
```
1. bucket
   一个数据分组，按照某个字段进行bucket划分，那个字段的值相同的那么数据，就会被划分到一个bucket中。
   
2. metric
   就是对一个bucket执行的某种聚合分析的操作，比如说求平均值，求最大值，最小值
   
3. histogram
   类似于terms，也是进行bucket分组操作，接收一个field，按照这个field的值的各个范围区间，进行bucket分组操作
   interval 划分的间隔
4. date_histogram
   按照我们指定的某个date类型的日期field，以及日期interval，按照一定的日期间隔，去划分bucket
   {
     "size": 0,
     "aggs": {
       "group_by_date": {
         "date_histogram": {
           "field": "sold_date",
           "interval": "month",
           "format": "yyyy-MM-dd",
           "min_doc_count": 0, //即使某个区间一个数据也没有，这个区间也会返回，不然默认是会过滤掉这个区间的
           "extended_bounds":{ //划分bucket的时候，会限定这个起始日期和截止日期
             "min":"2016-01-01",
             "max":"2017-12-31"
           }
         }
       }
     }
   }
 5. aggregation scope 一个聚合操作，必须在query的搜索结果范围内执行
    global：就是global bucket，就是将所有数据纳入聚合的scope，而不管之前的query
    GET /tvs/sales/_search
   {
     "size": 0, 
     "query": {//搜索
       "term": {
         "brand": {
           "value": "长虹"
         }
       }
     },
     "aggs": {
       "single_brand_avg_price": {
         "avg": {
           "field": "price"
         }
       },
       "all":{
         "global": {},
         "aggs": {
           "all_brand_avg_price": {
             "avg": {
               "field": "price"
             }
           }
         }
       }
     }
   }
   
6. 聚合的组合
   1）搜索 + 聚合：例子如上
   2）过滤 + 聚合
      GET /tvs/sales/_search
      {
        "size": 0,
        "query": {//过滤
          "constant_score": {
            "filter": {
              "range": {
                "price": {
                  "gte": 1200
                }
              }
            }
          }
        },
        "aggs": {
          "avg_price": {
            "avg": {
              "field": "price"
            }
          }
        }
   }
   
7. aggs.filter  针对的是聚合去做的
   如果放query里面的filter，是全局的，会对所有的数据都有影响
   比如说，你要统计，长虹电视，最近1个月的平均值; 最近3个月的平均值; 最近6个月的平均值
   
   bucket filter：对不同的bucket下的aggs，进行filter
```
* 三角选择原则
```
1. 精准+实时+大数据 --> 选择2个
（1）精准+实时: 没有大数据，数据量很小，那么一般就是单机
（2）精准+大数据：hadoop，批处理，非实时，可以处理海量数据，保证精准，可能会跑几个小时
（3）大数据+实时：es，不精准，近似估计，可能会有百分之几的错误率

2. 近似聚合算法
   如果采取近似估计的算法：延时在100ms左右，0.5%错误
   
3. 并行聚合算法：max
   1）有些聚合分析的算法，是很容易就可以并行的，比如说max
   2）有些聚合分析的算法，是不好并行的，比如说，count(distinct)
      es会采取近似聚合的方式，就是采用在每个node上进行近估计的方式，得到最终的结论
      近似估计后的结果，不完全准确，但是速度会很快，一般会达到完全精准的算法的性能的数十倍
```
* cartinality的优化
```
1. es，去重，cartinality metric，对每个bucket中的指定的field进行去重，取去重后的count，类似于count(distcint)

2. precision_threshold优化准确率和内存开销
   GET /tvs/sales/_search
   {
       "size" : 0,
       "aggs" : {
           "distinct_brand" : {
               "cardinality" : {
                 "field" : "brand",
                 "precision_threshold" : 100  //在多少个unique value以内，cardinality，几乎保证100%准确
               }
           }
       }
   }
   1) cardinality算法占用的内存
      内存消耗： precision_threshold * 8 byte 
      precision_threshold，值设置的越大，占用内存越大
      数百万的unique value，错误率在5%之内
      1KB=1024B（byte B）= 1000B(近似)
      
   2）HyperLogLog++ (HLL)算法性能优化
      cardinality底层算法：HLL算法
      算法的内容是：会对所有的uqniue value取hash值，通过hash值近似去求distcint count
      优化：
      默认情况下，发送一个cardinality请求的时候，会动态地对所有的field value，取hash值; 
      将取hash值的操作，前移到建立索引的时候
      PUT /tvs/
      {
        "mappings": {
          "sales": {
            "properties": {
              "brand": {
                "type": "text",
                "fields": {
                  "hash": {
                    "type": "murmur3" 
                  }
                }
              }
            }
          }
        }
      }
      GET /tvs/sales/_search
      {
          "size" : 0,
          "aggs" : {
              "distinct_brand" : {
                  "cardinality" : {
                    "field" : "brand.hash",
                    "precision_threshold" : 100 
                  }
              }
          }
      }
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
