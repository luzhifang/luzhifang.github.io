# Elasticsearch 常用命令


## 状态查询

### 获取所有 `_cat` 系列的操作

```Shell
curl http://localhost:9200/_cat
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
```

### 集群状态

```Shell
curl -X GET "localhost:9200/_cluster/health?pretty"
```

### 节点状态

- 节点简要信息

```Shell
curl -X GET "localhost:9200/_cat/nodes?pretty&v"
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.25.64.5           38          97   6    0.18    0.14     0.56 cdfhilmrstw -      es02
172.25.64.3           54          97   6    0.18    0.14     0.56 cdfhilmrstw -      es03
172.25.64.2           52          97   6    0.18    0.14     0.56 cdfhilmrstw *      es01
```

- 节点详细信息

```Shell
curl -X GET "localhost:9200/_nodes/stats/http?pretty"
```

> 后面的 http 是查看的属性，另外还有 indices, fs, http, jvm, os, process, thread_pool, discovery 等，支持组合（如 indices,fs,http）

### 分片状态

```Shell
curl -X GET "localhost:9200/_cat/shards?v&pretty"
index                                                         shard prirep state   docs   store ip          node
elk-demo-2023.01.14                                           0     p      STARTED   98  64.2kb 172.25.64.5 es02
elk-demo-2023.01.14                                           0     r      STARTED   98  31.2kb 172.25.64.3 es03
```

> 分片中如果存在未分配的分片， 可以查看未分片的原因：`_cat/shards?h=index,shard,prirep,state,unassigned.reason&v`

## 索引

### 索引管理

- 索引列表

```Shell
curl -X GET "localhost:9200/_cat/indices?v"
health status index                             uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   elk-demo-2023.01.13               _NX6po1HSs6-z_j0wVqBOw   1   1          6            0     40.2kb         20.1kb
```

条件过滤：`_cat/indices?v&health=yellow`

排序：`_cat/indices?v&health=yellow&s=docs.count:desc`

- 索引详细信息

```Shell
curl -X GET "localhost:9200/elk-demo-2023.01.14/_stats?pretty"
```

- 数据量

```Shell
curl -X GET "localhost:9200/_cat/count/elk-demo-2023.01.14?v&pretty"
```

- 新建索引

```Shell
curl -X PUT -H "content-type:application/json" "localhost:9200/my_index" -d '
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3,
            "number_of_replicas" : 2
        }
    }
}'
```

- 删除索引

```Shell
curl -X DELETE "localhost:9200/my_index"
```

- 分词搜索

```Shell
curl -X POST -H "content-type:application/json" "localhost:9200/elk-demo-2023.01.14/_search" -d '
{
  "query": {
    "match": {
      "message": "测试"
    }
  }
}'
```

- 完全匹配搜索

```Shell
curl -X POST -H "content-type:application/json" "localhost:9200/elk-demo-2023.01.14/_search?pretty" -d '
{
  "query": {
    "match_phrase": {
      "message": "测试"
    }
  }
}'
```

## 别名

- 查看别名

```Shell
curl -X GET "localhost:9200/_alias/elk-demo-2023.01.14?pretty"
```

- 增加别名

```Shell
curl -X PUT "localhost:9200/elk-demo-2023.01.14/_alias/elk-demo-2023.01.14_alias?pretty"
```

- 删除别名

```Shell
curl -X POST -H "content-type:application/json" 'http://localhost:9200/_aliases' -d '
{
    "actions": [
        {"remove": {"index": "elk-demo-2023.01.14", "alias": "elk-demo-2023.01.14_alias"}}
    ]
}'
```

- 别名重新绑定

```Shell
curl -X POST -H "content-type:application/json" 'http://localhost:9200/_aliases' -d '
{
    "actions" : [
        { "remove" : { "index" : "elk-demo-2023.01.14", "alias" : "elk-demo-2023.01.14_alias" } },
        { "add" : { "index" : "elk-demo-2023.01.14_v2", "alias" : "elk-demo-2023.01.14_alias" } }
    ]
}'

```

## 参考

