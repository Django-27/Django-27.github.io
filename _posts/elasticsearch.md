# 1 elasticsearch 
- brew info elasticsearch
- brew services list
- brew services start elasticsearch # restart , stop

在elasticsearch中，一个节点(node)就是一个elasticsearch实例，而一个集群(cluster)由一个或多个节点组成，它们具有相同的cluster.name，并且协同工作，分享数据和负载。当加入新的节点或者删除一个节点时，集群就会感知到并平衡数据
```
http://127.0.0.1:9200/_cluster/health?pretty
```
## 广播
当es实例启动的时候，它发送了广播的ping请求到地址224.2.2.4:54328。而其他的es实例使用同样的集群名称响应了这个请求；（但是，广播也有不好之处，过程不可控
## 单播
告诉es集群其他节点的ip及（可选的）端口及端口范围。我们在elasticsearch.yml配置文件中设置
```
discovery.zen.ping.unicast.hosts: ["10.0.0.1", "10.0.0.3:9300", "10.0.0.6[9300-9400]"]
```
## 主节点选取
当集群中节点发生变化，会协商谁成为主节点（es认为所有节点都有资格成为主节点）；
如果只有一个节点，该节点等待一段时间，没有发现其他节点，就任命自己为主节点；
### 脑裂
当网络存在问题，一个或多个节点失去与主节点的通信，即开始选举新的主节点；可能出现，两个不同的集群在相互运行；
解决办法：设置集群节点总数，规则：节点总数除以2加一（保证一半以上）
discovery.zen.minimum_master_nodes: 3   # 3=5/2+1
如果不满足上面的条件，则没有办法选举，只能一直处于寻找集群状态；
如果都不满足条件，也可以设定node_master，让其中一个有资格成为主节点
### 错误识别
就是当主节点确定后，建立内部ping机制来确保每个节点在集群中保持活跃和健康；（也是一个互相ping的过程）
```
discovery.zen.fd.ping_interval: 1  # 默认1秒发送一个ping请求
discovery.zen.fd.ping_timeout: 30  # 等待的时间，默认30秒
discovery_zen.fd.ping_retries: 3   # 默认尝试3次，无果的话，宣布节点失联，并且在需要的时候进行新的分片和主节点选举
```
# elasticsearch 使用小记
- ES=elaticsearch, 是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据 
- Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单
- 主要解决问题：检索相关数据、返回统计结果、速度要快
- 启动方式 C:\elasticsearch\bin elasticsearch  # -d 后台启动
- 启动 elasticsearch-head ： npm run start

```
# command line url viewer
# -L 自动跳转 -i 显示header头信息 -v 显示通信过程  -X POST --data "data=xxx"  发送post表单数据  -X 指定接口动词
# --user-agent "[User Agent]"    或   --cookie "name=xxx"   或   --header "Content-Type:application/json"
# --user name:password 进行http认证
curl -XGET http://localhost:9200/goods/dealergoods/_search?q=*    # pretty=true
{"_index":"goods","_type":"dealergoods","_id":"_serach","found":false}

server {  # nginx 
    server_name es.dev2.youmiyou.cn;
    location / {
        proxy_pass  http://localhost:9200;
    }
}

浏览器中执行：http://es.dev2.youmiyou.cn/goods/dealergoods/_search?q=*

git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start

curl -XPUT 'http://localhost:9200/dept/employee/32' -d '{ "empname": "emp32"}'
```
- chrome 中也可以安装 插件，直接访问就是：http://es.dev2.youmiyou.cn/, 这个地址是在nginx中配置的
- 开发机中es是后台启动的，所以可以省去上面关于es-head的安装，直接访问就好
- 下面是浏览器中打开的效果
![image 2018-03-23 003](https://github.com/Django-27/workspace/blob/master/pic/elasticsearch_chrome.png)

MySQL | ES | MySQL | ES
---|---|---|--- 
Database | Index | Index | Everything is indexed
Table | Type | SQL | Query DSL
Row | Field | select * from * | GET http://..
Scheme | Mapping  | update table set | PUT http://..

- 在一个关系型数据库里面，schema定义了表、每个表的字段，还有表和字段之间的关系
- 与之对应的，在ES中：Mapping定义索引下的Type的字段处理规则，即索引如何建立、索引类型、
- 是否保存原始索引JSON文档、是否压缩原始JSON文档、是否需要分词处理、如何进行分词处理等
- 在数据库中的增insert、删delete、改update、查search操作
- 等价于ES中的增PUT/POST、删Delete、改_update、查GET
## (1)查看当前节点的所有 Index
- _cat命令，它帮助开发者快速查询Elasticsearch的相关信息
- v 表示显示详细的信息
- [中文社区](https://elasticsearch.cn/)
- _cat?help  或 _cat/allocation?help  #  通过help，显示每条命令的帮助信息 
- _cluster/health 集群状态
- status一共有3个状态：green表示所有主分片及备份分片都分配好了，集群100%健康
- yellow表示所有主分片都分配好了，但至少有一个备份分片未分配。数据没有丢失，搜索结果是完整的。但高可用性会缺失，存在丢失数据风险，以黄色表示警告
- red表示至少有一个主分片未分配，并且这个主分片的所有分片都丢失了。数据已经丢失了，搜索结果不完整，并且往这个分片索引数据会发生异常
- active_primary_shards，集群中所有index的主分片数量的总和
- active_shards，集群中所有index的分片数量的总和，包括备份分片
- relocating_shards，正在被移动的分片数量。通常为0，在集群rebalance的时候会有变化
- number_of_nodes和number_of_data_nodes，是节点数和数据节点数
- _nodes/ 节点状态
```
  curl -X GET 'http://localhost:9200/_cat/indices?v'
```
## (2)列出每个 Index 所包含的 Type
```
  curl 'localhost:9200/_mapping?pretty=true'
```
## (3)新建一个名叫weather的 Index
```
  curl -X PUT 'localhost:9200/weather'
```
## (4)删除这个 Index
```
  curl -X DELETE 'localhost:9200/weather'
```
## (5)向/accounts/person发送请求，就可以新增一条人员记录
```
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}' 
```
## (6)新增记录的时候，也可以不指定 Id，这时要改成 POST 请求;index 写错了也不报错，而是直接生成了一个新的
```
curl -X POST 'localhost:9200/accounts/person' -d '
{
  "user": "李四",
  "title": "工程师",
  "desc": "系统管理"
}'
```
## (7)发出 GET 请求，就可以查看这条记录;如果 Id 不正确，就查不到数据，found字段就是false
```
curl 'localhost:9200/accounts/person/1?pretty=true'
```
## (8)删除记录就是发出 DELETE 请求
```
curl -X DELETE 'localhost:9200/accounts/person/1'
```
## (9)更新记录就是使用 PUT 请求，重新发送一次数据
- 当记录的 Id 没变，但是版本（version）从1变成2，操作类型（result）
- 从created变成updated，created字段变成false，因为这次不是新建记录
```
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}' 
```
## (10)返回所有记录
- 返回结果的 took字段表示该操作的耗时（单位为毫秒)
- timed_out字段表示是否超时，hits字段表示命中的记录
- 默认一次返回10条，size字段指定大小，from指定偏移、位移
- 逻辑运算：如果有多个搜索关键字， Elastic 认为它们是or关系
```
curl 'localhost:9200/accounts/person/_search'
```
## (11)Elastic 的查询非常特别，使用自己的查询语法，要求 GET 请求带有数据体
```
curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { 
        "match" : {
            "desc" : "管理 软件" }},   # 搜索的是软件 or 系统
  "from": 1,
  "size": 1
}'
```
## (12)如果要执行多个关键词的and搜索，必须使用[布尔查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-bool-query.html)
```
curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "软件" } },
        { "match": { "desc": "系统" } }
      ]
    }
  }
}'

```
## (13)ES中的聚合被分为两大类：Metric度量和bucket桶
- metric很像SQL中的avg、max、min等方法；按返回类型可以分为两种：单值聚合 和 多值聚合
- 而bucket就有点类似group by
- 其中intraday_return是聚合的名字，同时也会作为请求返回的id值
- min, max, avg, percentiles 求百分比
- percentiles 求百分比, stats 统计，会直接显示多种聚合结果，包括 min/max/sum/count/avg
- extend stats 扩展统计
```
{  # 单值聚合
    "aggs" : {
        "min_price" : { "min" : { "field" : "price" } }
    }
}

{  # 多值聚合
    "aggs" : {
        "load_time_outlier" : {
            "percentile_ranks" : {
                "field" : "load_time", 
                "values" : [15, 30]
            }
        }
    }
}
```
![image 2020-05-9](https://github.com/Django-27/workspace/blob/master/pic/elasticsearch.png)
# 正向索引（forward index），反向索引（inverted index）即倒排索引
- 正向索引：以docID为key存储，需遍历一遍，找到所有有关键词的文档，再sort、rank等，但新增易维护(document to word)
- 反向索引：以word为key存储，表项记录了出现这个词的所有docID、位置偏移、出现评率等，但维护耗时(word to document)
```
curl -X PUT "localhost:9200/user/_doc/1" -H 'Content-Type: application/json' -d'
{
    "name" : "Jack",
    "gender" : 1,
    "age" : 20
}
'
```
id | name | age | address
:-:| :-: | :-: | :-: |
1  | jack | 1 | 20 | PEK
2  | qiyuan   | 2 | 23 | BAV
3  | sunny | 1 | 22 | BAV
假设user索引，有四个字段：name、gender、age、address
- Term（单词）：一段文本分词后，一个一个的就是Term，即单词
- Term Dictionary（单词字典）：Term的集合
- Term Index（单词索引）：为了更快找到某个单词，为单词建立索引
- Posting List（倒排索引）：一条记录是一个倒排项，记录出现某个单词的所有文档列表即在文档中的位置（还可以有
- 词频、偏移量等信息
ES 为每个字段都建立索引，如MySQL MyISAM存储引擎使用的B树，索引和数据分开，通过索引找到记录的地址，再找到记录；
类比MyISAM的话，Term Index 相当于索引文件，Term Dictionary 相当于数据文件，可以将他们看成一步，就是找Term；
所以：倒排索引就是通过单词找到倒排列表，再根据其中的倒排项找到文档记录；

集群状态 | 内容
:-:| :-:
Green | every thing is good （所有功能正常）
Yellow  | 所有数据可用，但有些副本没有分配（所有功能正常）
Red  |有些数据不可用（部分功能正常）

- 查看集群状态： curl -X GET "localhost:9200/_cat/health?v"
- 查看节点状态： curl -X GET "localhost:9200/_cat/nodes?v"

- 查看所有索引： curl -X GET "localhost:9200/_cat/indices?v"
- 创建一个索引： curl -X PUT "localhost:9200/customer?pretty"（（pretty的意思是响应（如果有的话）以JSON格式返回）
- 索引并查询：curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'{"name": "John Doe"}'
- 删除一个索引： curl -X DELETE "localhost:9200/customer?pretty"

- 更新文档：curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'{ "doc": { "name": "Jane Doe", "age": 20 }}'
- 更新文档（通过脚本）：curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'{ "script" : "ctx._source.age += 5"}'
- 删除文档：curl -X DELETE "localhost:9200/customer/_doc/2?pretty"

- 批处理 _bulk API: curl -X POST "localhost:9200/customer/_doc/_bulk?pretty" -H 'Content-Type: application/json' -d'{"index":{"_id":"1"}}{"name": "John Doe" }{"index":{"_id":"2"}}{"name": "Jane Doe" }'
- eg: curl -X POST "localhost:9200/customer/_doc/_bulk?pretty" -H 'Content-Type: application/json' -d'{"update":{"_id":"1"}}{"doc": { "name": "John Doe becomes Jane Doe" } }{"delete":{"_id":"2"}} '

- 加载数据 accounts.json : curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"

``` search API
- 1 通过REST请求URI发送检索参数，参数放在URI后面
curl -X GET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
- 2 通过REST请求体发送检索参数，参数放到请求体里面
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
- 3 通过DSL查询语言(Domain-specific language, 领域特定语言)
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "account_number": 20 }  # 查询条件
             "bool": {                          # bool must 即 where address like '%mill%lane%'
                   "must": [                    # must 相当于 and，should相当于or，must_not相当于！
                     { "match": { "address": "mill" } },
                     { "match": { "address": "lane" } }
                   ]
                   "filter": { # bool 支持过滤筛选
                           "range": {
                             "balance": {
                               "gte": 20000,
                               "lte": 30000
                             }
                           }
                         }
                 }
    ”size": 0, # 0 表示，只显示聚合结果，不显示搜索结果
    "aggs": { # 支持聚集函数，如分组、求和、求求平均等
        "group_by_state": {
          "terms": {
            "field": "state.keyword"
            “ranges": [
                {
                    "from": 20,
                    "to":30
                },
                {
                    "from": 30,
                    "to":50
                }
            ]
          }
        }，
        "aggs": { # 相当于 SELECT state, COUNT(*), AVG(balance) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;
            "group_by_gender": {
                      "terms": {
                        "field": "gender.keyword"
                      },
            "average_balance": {
                "avg": {
                    "field": "balance"
                  }
                }
              }
      }
  }
  "sort": { "balance": { "order": "desc" } }
  "_source": ["account_number", "balance"]  # 返回部分字段
}
'
```
# Query DSL
查询语句有两种类型子句组成，叶子查询子句 和 组合查询子句
叶子查询子句：查询特定字段中的特定值，如 match、term或range查询
组合查询子句：包装其他叶子或复合查询，用逻辑方式组合多个查询（bool、dis_max），或更改行为（如constant_socore)

查询子句的行为取决于它用在查询上下文（query context）、还是过滤上下文（filter context）
查询上下文：文档与查询子句的匹配程度，并计算 _score 表示文档与其他文档的相关程度（匹配所有文档，_socore为1.0）
过滤上下文：文档是否匹配查询子句（Yes或No），用于过滤结构化数据，不计算评分（频繁使用的过滤器将被ES自动缓存）
```
curl -X GET "localhost:9200/_search" -H 'Content-Type: application/json' -d'
{
    "query": {                                           # query 表示这是一个查询上下文
        "bool": {                           # bool和match子句用于查询上下文，表明他们参与每条文档的打分
            "must": [
                { "match": { "title":   "Search"        }}, 
                { "match": { "content": "Elasticsearch" }}  
            ],                              # 类比sql，match模糊查询，term精确查询，range范围查询
            "filter": [                                  # filter 表示这个一个过滤上下文
                { "term":  { "status": "published" }},   # term和range用在过滤上下文，过滤不匹配的文档
                { "range": { "publish_date": { "gte": "2015-01-01" }}} ]}}}'
```
- match 查询接受文本、数值、日期，先对提供的文本进行分析，并再分析过程为提供的文本构造一个布尔查询
- operator 选项可以设置为 or 或 and 来控制布尔子句（默认是or）
```
curl -X GET "localhost:9200/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match" : {                             # match_phrase 用于精确匹配或单词接近匹配
            "message" : {
                "query": "this is a test"，     # 查询子句以 query 开头 ,这里 message 是字段名
                "operator" : "and"，            # match 是模糊查询，添加参数 and
                "analyzer" : "my_analyzer"，    # 精确匹配是，设置用哪个分析器分析文本
                                                # match_phrase_prefix 允许文本最后一个单词前缀匹配
                "max_expansions": 10,           # match_phrase_prefix可接受一个max_expansions参数，它是用来控制最后一个单词可以扩展多少后缀（默认50）
            }
        }
        "multi_match" : { # match 只能针对一个字段，multi_match 可以指定多个字段
            "query":    "this is a test", 
            "fields": [ "subject", "message","title"] # 字段还可以指定通配符，如title,first_name,last_name,*_name等
                          # "subject^3" 字段重要性也可以进行提升，如例子中提升三倍
        }
        "query_string" : { # query_string 查询解析输入并围绕操作符进行拆分文本，每个文本都独立分析
            "default_field" : "content",
            "query" : "this AND that OR thus" # 支持 Lucene 查询字符串语法，允许指定 AND OR NOT,并在单个查询字符串中进行多字段查询
            ”default_operator": "AND" # 默认是 OR，可以指定
        }
        "simple_query_string" : { # 更简单、健壮、面向用户，不会抛出异常，并丢弃查询的无效部分
                "query": "\"fried eggs\" +(eggplant | potato) -frittata", # 支持 +、-、|、"、*、(、)、~N 操作符
                "fields": ["title^5", "body"],
                "default_operator": "and"
            }
    }
}
'
```
## Term level queries 单词级别查询
全文本查询会在执行之前对查询字符串进行分词分析，而单词级别查询直接在相应的反向索引中精确查找
```
curl -X POST "localhost:9200/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "term" : { "user" : "Kimchy" } # 在user字段的倒排表中精确查找Kimchy的文档
    "terms" : { "user" : ["kimchy", "elasticsearch"]} # 结果是 OR
    "range" : { "age" : {"gte" : 10,"lte" : 20,"boost" : 2.0}} #查范围，boost指重要程度（默认1.0）
    "range" : { "date" : {"gte" : "now-1d/d","lt" :  "now/d"} } # date类型字段，范围可以用Date Math
    "range" : { "born" : { "gte": "01/01/2012", "lte": "now", "format": "dd/MM/yyyy||yyyy","time_zone": "+01:00"}}
    "exists" : { "field" : "user" }  # 在特定字段中查找非空的文档
    "prefix" : { "user" : "ki" }     # 查找包含指定前缀的term的文档
    "wildcard" : { "user" : "ki*y" } # 支持通配符，*表示任意字符，？表示任意单个字符
    "regexp":{ "name.first": "s.*y"} # 支持使用正则
    "ids" : { "type" : "_doc", "values" : ["1", "4", "100"] }} # 用 _uid 查询
  }
}'
```
## 复合查询
复合查询包装其他复合查询或叶子查询，以组合他们的结果和得分，更改他们的行为，或从查询切换到筛选上下文
```
curl -X GET "localhost:9200/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "constant_score" : { # 固定分数查询
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
        "bool" : {
                    "must" : {
                        "term" : { "user" : "kimchy" }
                    },
                    "filter": {
                        "term" : { "tag" : "tech" }
                    },
                    "must_not" : {
                        "range" : {
                            "age" : { "gte" : 10, "lte" : 20 }
                        }
                    },
                    "should" : [
                        { "term" : { "tag" : "wow" } },
                        { "term" : { "tag" : "elasticsearch" } }
                    ],
                    "minimum_should_match" : 1, # should需要满足的最小条件
                    "boost" : 1.0 }}}'
```
# 深刻理解为何单节点集群健康状态为 Yellow
```
    1: 在配置文件中修改，/usr/local/etc/elasticsearch/elasticsearch.yml
    （Since elasticsearch 5.x index level settings can NOT be set on the nodes configuration like the elasticsearch.yaml）
    
    index.number_of_shards: 5 # 设置数据分片个数，默认为5个
    index.number_of_replicas: 0 # 设置索引副本个数（即数据备份书),如果一台机器就设置为0，yellow->green 
    
    默认情况下5个分片，0,1,2,3,4是分片
    默认设置1个副本，所以每一个分片都有一个备知份，并且备份一定是在不同的主机上，
    当es集群有一个主机出现意外数据丢失时，其他主机的副本备份会自动同步到丢失数据的主机上
    
    但如果是单节点（如自己电脑上创建一个es实例），在同一个节点上既保存原始数据又保存副本是没有意义的，因
    为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据(这也就是按照默认创建的结点，集群状态为黄色的原因)
    则单节点无论有多少个副本分片（number_of_replicas 的设置）都是 unassigned —— 它们都没有被分配到任何节点
    
    2: 通过 -d 参数 在 curl 是追加设定(5.x版本以后，上面的设定已经取消，必须通过/${index}/_settings API )
     curl -XPUT 'http://localhost:9200/bank' -H 'Content-Type: application/json' -d'
     {
       "settings":{
                "number_of_shards":3,     
                "number_of_replicas":0 } }'
      
    
    加载数据 accounts.json : curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
    
    3： Can't update non dynamic settings 
    - 索引一旦建立，主分片数量不可改变: 先删除索引 curl -X DELETE "localhost:9200/bank?pretty"
    - 手动关停索引： curl -XPOST 'localhost:9200/you_index_name/_close'
    - 更改配置，如2中的内容, 建索引并指定分片和备份
    - 加载数据到上面的索引
    - 开启索引：curl -XPOST 'localhost:9200/bank/_open'
    - Yellow 变为 Green (同理修改分词器参数的时候，也要先关闭索引，在打开索引)
```
### aaa
1.身份证原件及正反面复印件 2 份；(可以提供)

2.上一家单位离职证明及收入证明原件（半年以上的银行对账单，需加盖银行红章，月薪超过 1 万元
以上员工须提交）；（不是银行转账，走的微信支付宝提供截图复印件可以吗）

3.毕业证，学位证书原件及复印件各 1 份；（只提供复印件可以吗，原件在老家没有带在身边）

4.近三个月体检证明原件（一般项目，心电图，谷丙转氨酶检测）；（可以提供）

5.银行借记卡复印件（手写注明卡号及开户行名称）（可以提供）

6. 职称证书、职业资格证书或其他相关培训证书原件及复印件 1 份；（没有可以吗）

7. 如果您未曾缴纳过各项社会保险，请携带户口本首页和本人页（含变更页）复印件 1 份，一寸彩色
白底免冠照片电子版；（缴纳过，中间有断档可以吗）