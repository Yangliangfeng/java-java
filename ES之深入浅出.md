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
   每个子查询都会计算一个document针对它的相关度分数，然后bool综合所有分数，合并为一个分数，当然filter是不会计算分数的
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
