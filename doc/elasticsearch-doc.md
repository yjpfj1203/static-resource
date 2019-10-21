# 基础概念
索引：含有相同属性的文档信息<br/>
类型：索引可以定义一个或多个类型，文档必须属于一个类型<br/>
文档：文档是可以被索引的基本数据单位<br/>
分片：每个索引都有多个分片，每个分片是一个Lucence索引<br/>
备份：拷贝一份分片就完成了分片备份<br/>

# 基本用法
api基本格式：http://[ip]:[port]/<索引>/<类型>/<文档id><br/>
GET/PUT/POST/DELETE<br/>

## 运维命令集合
### cat系列
_cat系统提供集群状态的接口，可以通过执行
GET localhost:9200/_cat获取所有_cat系统的操作。输出结果：
```
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
### cluster系列
1. 查询设置集群状态<br/>
GET http://localhost:9200/_cluster/health?pretty=true<br/>
pretty=true 表示格式化输出<br/>
level=indices 表示显示索引状态<br/>
level=shards 表示显示分片信息<br/>
2. 显示集群系统信息<br/>
GET http://localhost:9200/_cluster/stats?pretty=true
3. 修改集群配置<br/>
例：PUT http://localhost:9200/_cluster/settings
```
{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}
transient 表示临时的，persistent表示永久的
```
4. 对shard的手动控制<br/>
POST http://localhost:9200/_cluster/reroute
5. 关闭节点
```
关闭指定192.168.1.1节点
POST http://192.168.1.1:9200/_cluster/nodes/_local/_shutdown
POST http://localhost:9200/_cluster/nodes/192.168.1.1/_shutdown
关闭主节点
POST http://localhost:9200/_cluster/nodes/_master/_shutdown
关闭整个集群
POST http://localhost:9200/_shutdown?delay=10s
POST http://localhost:9200/_cluster/nodes/_shutdown
POST http://localhost:9200/_cluster/nodes/_all/_shutdown
delay=10s表示延迟10秒关闭
```
### nodes系列
查询节点的状态
```
GET http://localhost:9200/_nodes/stats?pretty=true
GET http://localhost:9200/_nodes/192.168.1.2/stats?pretty=true 
GET http://localhost:9200/_nodes/process
GET http://localhost:9200/_nodes/_all/process
GET http://localhost:9200/_nodes/192.168.1.2/jvm,process
GET http://localhost:9200/_nodes/192.168.1.2,192.168.1.3/info/jvm,process
GET http://localhost:9200/_nodes/192.168.1.2,192.168.1.3/_all
GET http://localhost:9200/_nodes/hot_threads
```
### 索引操作
1. 创建索引<br/>
PUT http://localhost:9200/fvp~oncarsummary/oncarsummary<br/>
PUT http://localhost:9200/fvp~onaviationtasksummary/onaviationtasksummary
2. 查看所有索引<br/>
GET http://localhost:9200/_cat/indices?v
3. 索引数据
```
POST http://localhost:9200/{index}/{type}/{id} 
  body: {"a":"avalue","b":"bvalue"}
PUT http://localhost:9200/{index}/{type}/{id} 
  body: {"a":"avalue","b":"bvalue"}
```
4. 查询索引<br/>
GET http://localhost:9200/fvp~onaviationtasksummary/fvp/_search?q=*&pretty<br/>
GET http://localhost:9200/fvp~oncarsummary/fvp/_search?q=*&pretty
5. 删除索引<br/>
DELETE http://localhost:9200/{index}/{type}/{id}
6. 获取mapping<br/>
GET http://localhost:9200/{index}/{type}/_mapping?pretty
7. 刷新索引<br/>
POST http://localhost:9200/kimchy,elasticsearch/_refresh<br/>
POST http://localhost:9200/_refresh
8. 设置mapping<br/>
POST http://localhost:9200/productindex/product/_mapping?pretty
```
{
    "product": {
            "properties": {
                "title": {
                    "type": "string",
                    "store": "yes"
                },
                "description": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "price": {
                    "type": "double"
                },
                "onSale": {
                    "type": "boolean"
                },
                "type": {
                    "type": "integer"
                },
                "createDate": {
                    "type": "date"
                }
            }
        }
}
```
9. 统计ES某个索引数据量<br/>
GET http://localhost:9200/_cat/count/new-sgs-rbil-core-system-dds-next-tcs-server-core-dcn-2017-06-27
10. 查看模板<br/>
GET http://localhost:9200/_template/fvp_waybillnewstatus_template
11. 设置模板<br/>
PUT http://localhost:9200/_template/fvp_waybillnewstatus_template?pretty
12. 删除模板<br/>
DELETE localhost:9200/_template/template_1

## 结构创建
### 路径： http://localhost:9200/people PUT<br/>

```
{
    "settings":{
        "number_of_shards":3,
        "number_of_replicas":1
    },
    "mappings":{
        "man":{
            "properties":{
                "name":{
                    "type":"text"
                },
                "country":{
                    "type":"keyword"
                },
                "age":{
                    "type":"integer"
                },
                "date":{
                    "type":"date",
                    "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
                }
            }
        }
    }
}
```

### settings: 设置<br/>
　　number_of_shards: 分片数<br/>
　　number_of_replicas：备份数<br/>
　　mappings: 索引的映射<br/>
　　man: 映射名称，此处的映射名称只能有一个不能出现多个映射（这是一个设计失误，后面的版本将不再支持。官方给出解释是：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html ）<br/>
　　properties：属性的集合 ：（它下面的为各个属性，都是"属性名":{"（类型）type":"对应的类型"}）注：其中Data属性有format设置日期格式<br/>

## 数据插入(/索引/类型/id)
### 指定文档id插入： http://127.0.0.1:9200/people/man/1 PUT
```
{
    "name":"晓明",
    "country":"china",
    "age":26,
    "date":"1992-08-08"
}
```

### 自动产生文档id插入:  http://127.0.0.1:9200/people/man POST
```
{
    "name":"Auto晓明",
    "country":"Autochina",
    "age":26,
    "date":"1992-08-08"
}
```

### 修改： http://127.0.0.1:9200/people/man/1/_update PUT
```
直接修改文档
{
    "doc":{
        "name":"修改晓明"
    }
}

脚本修改文档
{
    "script":{
        "lang":"painless",
        "inline":"ctx._source.age += 10"
    }
}

注：script 是脚本标识

　　lang 是脚本语言

　　painless 是内置脚本语言

　　inline 是指定脚本内容

　　ctx 是脚本上下文

　　_source 为当前文档
```
### 修改：127.0.0.1:9200/people/man/1/_update POST
```
{
    "script":{
        "lang":"painless",
        "inline":"ctx._source.age = params.age",
        "params":{
            "age":100
        }
    }
}
```
### 删除文档： http://127.0.0.1:9200/people/man/1 DELETE

### 删除索引： http://127.0.0.1:9200/people DELETE

## 查询
### 准备
#### 创建结构： http://127.0.0.1:9200/people01 PUT
```
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "novel": {
    "properties": {
      "title": {
        "type": "text"
      },
      "author": {
        "type": "keyword"
      },
      "tittle": {
        "type": "text"
      },
      "publish_date": {
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
        "type": "date"
      }
    }
  }
}
```

#### 插入数据： http://127.0.0.1:9200/book/novel/1
```
{
    "name":"晓明1",
    "country":"china1",
    "age":26,
    "date":"1992-08-08"
}
```
### 简单查询
#### http://127.0.0.1:9200/book/novel/1 GET
#### 条件查询：http://127.0.0.1:9200/book/_search POST
```
1、全部查询
{
    "query":{
        "match_all":{}
    },
    "from":1,
    "size":1
}
2、按关键字查询（含有按指定字段排序）
{
    "query":{
        "match":{
            "name":"晓明"
        }
    },
    "sort":[
        {"_id":"desc"}
        ]
}
3.解释请求json串：
match_all 是全部查询
query 是查询关键字
from 是从哪里查
size 是显示多少条数据
sort 是排序设置
```
### 聚合查询
#### http://127.0.0.1:9200/book/_search POST
```
{
  "aggs": {
    "group_by_name": {
      "terms": {
        "field": "name"
      }
    },
    "group_by_country": {
      "terms": {
        "field": "country"
      }
    }
  }
}

如果上面查询报错，可先执行下面的语句
 http://127.0.0.1:9200/book/_mapping/novel/ PUT
{
  "properties": {
    "name": { 
      "type":     "text",
      "fielddata": true
    },
     "country": { 
      "type":     "text",
      "fielddata": true
    }
  }
}

注：解析请求json字符串：
aggs：聚合查询关键字
group_by_word_count ：聚合条件的名字（名字可自定义）
terms 关键词
field 是指定字段
```

#### http://127.0.0.1:9200/book/_search   POST
```
{
  "aggs": {
    "group_by_age": {
      "stats": {
        "field": "age"
      }
    }
  }
}
注：计算年龄的统计  总条数、最大值、最小值、平均值、总和   若：stats改成min则只显示最小值
```




