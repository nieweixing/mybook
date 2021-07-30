# docker-compose搭建Elasticsearch集群

在主节点创建一个目录es,并创建docker-compose.yaml，主节点yaml文件如下

```
version: '2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.2
    privileged: true
    environment:
      - cluster.name=docker-cluster
      - xpack.security.enabled=false
      - bootstrap.memory_lock=true
      - node.master=true
      - node.data=true                # store data on master
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"  #根据机器实际来分配
      - "discovery.zen.ping.unicast.hosts=192.168.30.30"    # put your master_ip here! make sure `telnet MASTER_IP 9300` is oK
      - "transport.host=0.0.0.0"
      - "network.host=0.0.0.0"
      - node.name=esmaster
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK
    volumes:
      - /data/elasticsearch-service/data:/usr/share/elasticsearch/data
    ports:
      - 9208:9200
      - 9308:9300
network_mode: "host"
```

执行命令docker-compose up -d启动es，一般会报错max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]，可以通过sudo sysctl -w vm.max_map_count=262144设置下值。

在node节点上同样操作创建es目录，创建docker-compose.yaml，node节点yaml文件如下

```
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.2
    privileged: true
    environment:
      - cluster.name=docker-cluster
      - xpack.security.enabled=false
      - bootstrap.memory_lock=true
      - node.master=false
      - node.data=true
      - "ES_JAVA_OPTS=-Xms12g -Xmx12g"
      - "discovery.zen.ping.unicast.hosts=192.168.30.30"    # put your master_ip here! make sure `telnet MASTER_IP 9300` is oK
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK
    volumes:
      - /data/elasticsearch-service/data:/usr/share/elasticsearch/data
    ports:
      - 9208:9200
      - 9308:9300
network_mode: "host"
```

执行命令docker-compose up -d启动es，一般会报错max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]，可以通过sudo sysctl -w vm.max_map_count=262144设置下值

# helm部署es集群到k8s上

```
# helm repo add elastic https://helm.elastic.co
# helm install --name elasticsearch elastic/elasticsearch
```

# elasticsearch命令操作大全

## 命令格式

```
curl -X<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>
```

* <REST Verb>：REST风格的语法谓词
* <Node>:节点ip
* <port>:节点端口号，默认9200
* <Index>:索引名
* <Type>:索引类型
* <ID>:操作对象的ID号

## 访问带鉴权的es

```
curl -u 'elastic:Je2pW2SOa'  http://192.168.30.32:9200/
```

## 查看es信息

```
curl http://192.168.30.32:9200/
```

## 查看集群状态

```
curl http://192.168.30.32:9200/_cat/nodes 
```

## 查看所有索引

```
curl http://192.168.30.32:9200/_cat/indices
```

## 创建索引nwx

```
curl -XPUT '192.168.30.32:9200/nwx?pretty'
```
## 查看customer索引状态:

```
curl -XGET http://192.168.30.32:9200/_cat/indices/customer/?pretty 
```

## 往索引customer中插入数据，类型为external，id为1

```
curl -XPUT '192.168.30.32:9200/customer/external/1?pretty' -d '
{
    "name": "nie wei xing"
}'
```

## 往索引customer中插入数据，类型为aaa，id为1

```
curl -XPUT '192.168.30.32:9200/customer/aaa/1?pretty' -d '
{
    "name": "nie wei xing"
}'
```

## 往索引customer中批量插入数据

```
curl -XPOST '192.168.30.32:9200/customer/external/_bulk?pretty' -d '
{"index":{"_id":"3"}}
{"name": "John Doe" }
{"index":{"_id":"4"}}
{"name": "Jane Doe" }
'
```

## 导入数据集

```
curl -H 'Content-Type: application/x-ndjson' -XPOST 
'192.168.30.32:9200/bank/account/_bulk?pretty' --data-binary @accounts.json

curl -H 'Content-Type: application/x-ndjson' -XPOST 
'192.168.30.32:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json

curl -H 'Content-Type: application/x-ndjson' -XPOST '192.168.30.32:9200/_bulk?pretty' 
--data-binary @logs.jsonl
```

## 查询插入的数据

```
curl -XGET '192.168.30.32:9200/customer/external/1?pretty'

curl -XGET '192.168.30.32:9200/customer/aaa/1?pretty'
```

## 查询某个索引的所有数据

```
curl '192.168.30.32:9200/bank/_search?q=*&pretty'

curl '192.168.30.32:9200/customer/_search?q=*&pretty'

curl '192.168.30.32:9200/shakespeare/_search?q=*&pretty'

curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
	"query": { "match_all": {} }
}'
```

## 匹配所有数据，但只返回1个

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
	"query": { "match_all": {} },
	"size": 1
}'
```

注意：如果siez不指定，则默认返回10条数据。


## 返回从11到20的数据。（索引下标从0开始）

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from" : 10,
  "size" : 10
}'
```

## 匹配所有数据返回前10条

匹配所有的索引中的数据，按照balance字段降序排序，并且返回前10条

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'
```


## 下面例子展示如何返回两个字段（account_number balance）

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'
```

## 返回account_number 为20 的数据

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
 　"query": { "match": { "account_number": 20 } }
}'
```

## 返回address中包含mill的所有数据

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
 　"query": { "match": { "address": "mill" } }
}'
```
　
## 返回地址中包含mill或者lane的所有数据

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
   "query": { "match": { "address": "mill lane" } }
}'
```

## 这个例子是多匹配（match_phrase短语匹配）

返回地址中包含短语 “mill lane”的所有数据：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": { "match_phrase": { "address": "mill lane" } }
}'
```

## must布尔查询

布尔查询允许我们将多个简单的查询组合成一个更复杂的布尔逻辑查询。

这个例子将两个查询组合，返回地址中含有mill和lane的所有记录数据：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

## Should布尔查询

上述例子中，must表示所有查询必须都为真才被认为匹配。

相反, 这个例子组合两个查询，返回地址中含有mill或者lane的所有记录数据：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

## must_not布尔查询

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

## 多级逻辑查询

上述例子中,must_not表示查询列表中没有为真的（也就是全为假）时则认为匹配。

我们可以组合must、should、must_not来实现更加复杂的多级逻辑查询。

下面这个例子返回年龄大于40岁、不居住在ID的所有数据：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
      }
  }
}'
```

## 过滤filter(查询条件设置)

下面这个例子使用了布尔查询返回balance在20000到30000之间的所有数据。

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
        "filter": {
          "range": {
            "balance": {
               "gte": 20000,
               "lte": 30000
            }
        }
      }
     }
   }
}'
```

## 聚合 Aggregations

下面这个例子： 将所有的数据按照state分组（group），然后按照分组记录数从大到小排序，返回前十条（默认）：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    } 
  }
}'
```

下面这个实例按照state分组，降序排序，返回balance的平均值：

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance.keyword"
          }
        }
      }
    }
  }
}'
```

## 带条件查询

```
curl -XPOST '192.168.30.32:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "account_number": 20 } }
}'
```

## 索引数据的修改

```
curl -XPUT '192.168.30.32:9200/customer/external/1?pretty' -d '
{
    "name": "Jane Doe"
}'
```

## 索引数据的更新

```
curl -XPOST '192.168.30.32:9200/customer/external/1/_update?pretty' -d '
{
    "doc": {"name": "Jane Doe1"}
}'
```

## 索引数据的删除

```
curl -XDELETE '192.168.30.32:9200/customer/external/2?pretty'
```

## 索引的删除

```
curl -XDELETE '192.168.30.32:9200/nwx?pretty'
```

## 更新id为1的内容并删除id为2的

```
curl -XPOST '192.168.30.32:9200/customer/external/_bulk?pretty' -d '
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}'
```

## 设置全局的慢日志级别

```
curl -XPUT '192.168.30.32:9200/_cluster/settings/?pretty' -H 'Content-Type: application/json' -d '{
  "transient": {
    "logger.index.search.slowlog":"DEBUG",
    "logger.index.indexing.slowlog":"DEBUG"
  }
}'
```

## 设置customer索引的慢日志超时时间

```
curl -XPUT '192.168.30.32:9200/customer/_settings/?pretty' -H 'Content-Type: application/json' -d '{
    "index.search.slowlog.threshold.query.debug" : "1ms", 
    "index.search.slowlog.threshold.fetch.debug": "1ms", 
    "index.indexing.slowlog.threshold.index.debug": "1ms" 
}'

curl -XPUT '192.168.30.32:9200/people/_settings/?pretty' -H 'Content-Type: application/json' -d '{
    "index.search.slowlog.threshold.query.debug" : "1ms", 
    "index.search.slowlog.threshold.fetch.debug": "1ms", 
    "index.indexing.slowlog.threshold.index.debug": "1ms" 
}'

curl -XPUT '192.168.30.32:9200/bank/_settings/?pretty' -H 'Content-Type: application/json' -d '{
    "index.search.slowlog.threshold.query.debug" : "1ms", 
    "index.search.slowlog.threshold.fetch.debug": "1ms", 
    "index.indexing.slowlog.threshold.index.debug": "1ms" 
}'
```