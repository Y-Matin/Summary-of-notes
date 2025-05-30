<!-- ## ElasticSearch 教程 -->

#### 简介
> Elasticsearch是一个开源的分布式、RESTful 风格的搜索和数据分析引擎，它的底层是开源库Apache Lucene。
> Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库——无论是开源还是私有，但它也仅仅只是一个库。为了充分发挥其功能，你需要使用 Java 并将 Lucene 直接集成到应用程序中。 更糟糕的是，您可能需要获得信息检索学位才能了解其工作原理，因为Lucene 非常复杂。
> 为了解决Lucene使用时的繁复性，于是Elasticsearch便应运而生。它使用 Java 编写，内部采用 Lucene 做索引与搜索，但是它的目标是使全文检索变得更简单，简单来说，就是对Lucene 做了一层封装，它提供了一套简单一致的 RESTful API 来帮助我们实现存储和检索。

> 当然，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确地形容：
    - 一个分布式的实时文档存储，每个字段可以被索引与搜索；
    - 一个分布式实时分析搜索引擎；
    - 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据。


#### 基本概念
- 索引
- type ：  表  <慢慢会被弃用>
- documents ： 行 
- fields： 字段
- elasticsearch集群中可以包含多个索引（数据库），每个索引中可以包含多个类型（表），每个类型下又包含多个文档（行），每个文档中包含多个字段（列）
物理设计：
elasticsearch在后台把每个索引划分成多个分片，每片分片可以在集群中不同服务器间迁移
一个集群至少有一个节点，而一个节点就是一个es进程，节点可以有多个索引，如果你创建索引，那么索引将会有5个分片（primary shard，又称主分片）构成的，每一个主分片会有一个副本（replica shard，又称复制分片）
在多节点的集群中，主分片和复制分片不会在同一个节点中，防止当某个节点挂掉后，其他节点无该节点的数据。导致数据丢失。
实际上，一个分片是一个Lucene索引，一个包含倒排索引的文件目录，倒排索引的结构使得es在不扫描全部文档的情况下，就能告诉你哪些文档包含特定的关键字。
所以一个es索引是由多个Lucene索引组成的。
##### 倒排索引

##### ik分词器

- 增加自己的词。 如：“花果山居士”就是一个词，不能再切割
put     /索引名称/类型名称/文档id   创建文档（指定文档id）
post：  /索引名称/类型名称   创建文档（随机文档id）
post：  /索引名称/类型名称/文档id/_update   修改文档
delete：  /索引名称/类型名称/文档id   删除文档（指定文档id）
get：  /索引名称/类型名称/文档id   查看文档（指定文档id）
post  /索引名称/类型名称/_search   查询所有数据

delete：删除
get：查询

路径顺序：      /索引名称/类型名称/文档id




```cmd
// 分词器
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "花果山居士"
}

GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "花果山居士"
}

```
- 关于索引的操作
```cmd
PUT /test1/type/1
{
  "name":"1231",
  "age":123
}

PUT /test3/_doc/1
{
  "name":"yds",
  "age":123,
  "birth":"2020-12-04"
}

GET /test3/
```

该字段如果是映射的是text，并且是keyword，那么不会被分词。
GET test_user/user/_search?q=name:user02
 查询 索引为：test_user， type为user的结果集中，文档的name字段 是否是 user02. 匹配规则是 精准匹配。

#### 复杂搜索

- 单个查询：match ：会对中文进行分词。 根据score排序。
```json
GET /test_user/user/_search
{
  "query":{
    "match": {
      "name": "花果"
    }
  }
}
```
img1:

- 结果过滤：只查询目标记录的某个字段
_source:[]
```json
GET /test_user/user/_search
{
  "query":{
    "match": {
      "name": "花果"
    }
  }
, "_source": "{name}"
}
```
- 排序：
```json
GET /test_user/user/_search
{
  "query":{
    "match": {
      "name": "花果"
    }
  }
, "_source": "{name}",
"sort": [
  {
    "age": {
      "order": "desc"
    }
  }
]
}
```
- 分页：
```json
"from": 0,
"size": 1
```
- 条件 ： must=>and ; should=>or; must_not=>not 
```json
GET /test_user/user/_search
{
  "query":{
    "bool": {
      "should": [
        {"match": {
          "name": "花果"
          }
        }
        ,
        {"match": {
          "age": "37"
        }}
        
    ]}
  }

}
```

 
- 过滤器  可以有多个过滤器
```json
GET /test_user/user/_search
{
  "query":{
    "bool": {
      "must": [
        {"match": {
          "name": "花果"
          }
        }
    ],
    "filter": [
      {"range": {
        "age": {
          "gte": 110
          
        }
      }}
    ]
    }
  }
}
```

- 匹配多个条件 。 空格隔开 


- 关于分词
1. term ：直接查询精确的
2. match ：模糊查询，会使用分词器解析 ！（先分析文档，才匹配）
  - 字段的两个类型：  
      1. text: 会被分词器分词，与查询条件进行匹配
      2. keyword ： 不会被分词， 直接与查询条件匹配

- 高亮查询 
hightlight : 将 name 高亮 
```json
GET /test_user/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "花果"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "age": {
              "gte": 110
            }
          }
        }
      ]
    }
  },
  "highlight": {"fields": {"name":{}}}
}
```

#### filter与query对比大解密
- filter，仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，对相关度没有任何影响。
- query，会计算每个document相对于搜索条件的相关度，并按照相关度进行排序。
- 一般来说，如果你是在进行搜索，需要将最匹配搜索条件的数据先返回，那么用query；如果你只是要根据一些条件筛选出一部分数据，不关注排序，那么使用filter。

#### filter和query性能对比
- filter，不需要计算相关度分数，不需要按照相关度分数进行排序，同时还有内置的机制，自动cache最常使用filter的数据。
- query，相反，要计算相关度分数，按照分数进行排序，而且无法cache结果。







#### es 索引 改名，迁移
-	es Reindex 数据迁移
- ES在创建好索引后，mapping的properties属性类型是不能更改的，只能添加。如果说需要修改字段就需要重新建立索引然后把旧数据导到新索引。
-	5.X版本后新增_reindex API 。Reindex可以直接在Elasticsearch集群里面对数据进行重建。并且支持跨集群间的数据迁移
	  -	步骤：
	  -	1.原索引 topic
	  -	2.创建新的索引 topic-new
	  -	3.迁移数据:reindex
    ``` 
    POST http://172.16.1.236:9201/_reindex
      {
        "source": {
          "index": "topic"
        },
        "dest": {
          "index": "topic-new"
        }
      }
    ```
	- 4.删除原索引
	- 5.把原索引的名字作为新索引的别名（不用改业务代码）


- 一次查询的过程 
答： query the fetch
1、搜索被执行成一个两阶段过程，我们称之为 Query Then Fetch；
2、在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。
备注：在搜索的时候是会查询 Filesystem Cache 的，但是有部分数据还在 MemoryBuffer，所以搜索是近实时的。
3、每个分片返回各自优先队列中 所有文档的 ID 和排序值 给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。
4、接下来就是 取回阶段，协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并 丰富 文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。
5、补充：Query Then Fetch 的搜索类型在文档相关性打分的时候参考的是本分片的数据，这样在文档数量较少的时候可能不够准确，DFS Query Then Fetch 增加了一个预查询的处理，询问 Term 和 Document frequency，这个评分更准确，但是性能会变差。

2. es的准实时，内部原理
一次update和delete 会写入 memory buffer 和translog  事务日志中
 三级存储：
1.memory buffer   （最新的数据，不在搜索范围中）
2.filesystem Cache （数据都在可搜索范围中， 由1->2同步过程叫refresh，默认1s，或则buffer满了）
3.disk   （2->3过程叫flush，）

- translog 用于 分片恢复数据
所有索引的写入和删除操作在 Luence 内部处理之后确认之前被写入 Translog。这样如果发生崩溃，则在分片恢复时已确认但未提交写入磁盘的数据将从 Translog 中进行恢复。
Elasticsearch 的 flush 操作会执行 commit 提交 和 新的 Translog 生成，刷新是后台自动执行的，以确保 Translog 不会太大而使得恢复需要相当长的时间进行重放。
flush 操作会进行 commit，并清除 translog。
