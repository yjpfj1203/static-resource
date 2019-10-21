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
localhost:9200/_cat获取所有_cat系统的操作
输出结果：
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
## 结构创建
### 路径： http://127.0.0.1:9200/people PUT<br/>

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




