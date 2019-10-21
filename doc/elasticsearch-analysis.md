# master选举
```
conf/elasticsearch.yml:
    node.master: true/false
    node.data: true/false
```
node.master为true时，表示这个node是一个master的候选节点，可以参与选举，在es文档中被称为master-eligible node，类似于masterCandidate。<br/>
node.data为true时，这个节点作为一个数据节点，会存储分配在该node上的shard的数据并负责这些shard的写入、查询等。<br/>
node.master和node.data都为false时，这个节点可以作为一个类似proxy的节点，接受请求并进行转发、结果聚合等。<br/>
为了防止脑裂，需要把候选master节点最小值设置(minimumMasterNodes)为可以成为master节点数n/2+1。脑裂是指有两个master<br/>


### 1：master选举是谁发起的，什么时候发起
  master选举当然是由master-eligible节点发起，当一个master-eligible节点发现满足以下条件时发起选举：
    (1)：该master-eligible节点的当前状态不是master
    (2)：该master-eligible节点通过ZenDiscovery模块的ping操作询问其已知的集群其他节点，没有任何节点连接到master
    (3)：包括本节点在内，当前已有超过minimum_master_nodes个节点没有连接到master
    总结一句话，即当一个节点发现包括自己在内的多数派的master-eligible节点认为集群没有master时，就可以发起master选举
### 2：当需要选举master时，选举谁<br/>
  先根据节点的clusterStateVersion比较，clusterStateVersion越大，优先级越高。
  clusterStateVersion相同时，进入compareNodes，其内部按照节点的Id比较(Id为节点第一次启动时随机生成)
    (1)：当clusterStateVersion越大，优先级越高。这是为了保证新Master拥有最新的clusterState(即集群的meta)，避免已经commit的meta变更丢失。
         因为Master当选后，就会以这个版本的clusterState为基础进行更新。
    (2)：当clusterStateVersion相同时，节点的Id越小，优先级越高。即总是倾向于选择Id小的Node，这个Id是节点第一次启动时生成的一个随机字符串。
         之所以这么设计，应该是为了让选举结果尽可能稳定，不要出现都想当master而选不出来的情况


