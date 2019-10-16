# 基础概念
索引：含有相同属性的文档信息<br/>
类型：索引可以定义一个或多个类型，文档必须属于一个类型<br/>
文档：文档是可以被索引的基本数据单位<br/>
分片：每个索引都有多个分片，每个分片是一个Lucence索引<br/>
备份：拷贝一份分片就完成了分片备份<br/>

# 基本用法
api基本格式：http://[ip]:[port]/<索引>/<类型>/<文档id><br/>
GET/PUT/POST/DELETE<br/>

# 结构化创建
路径：/people<br/>

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

settings: 设置<br/>
　　number_of_shards: 分片数<br/>
　　number_of_replicas：备份数<br/>
　　mappings: 索引的映射<br/>
　　man: 映射名称，此处的映射名称只能有一个不能出现多个映射（这是一个设计失误，后面的版本将不再支持。官方给出解释是：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html ）<br/>
　　properties：属性的集合 ：（它下面的为各个属性，都是"属性名":{"（类型）type":"对应的类型"}）注：其中Data属性有format设置日期格式<br/>
