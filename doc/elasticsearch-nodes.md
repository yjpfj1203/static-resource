# ES各种节点的分工
1. 客户端节点<br/>
　　当主节点和数据节点配置都设置为false的时候，该节点只能处理路由请求，处理搜索，分发索引操作等，从本质上来说该客户节点表现为智能负载平衡器。独立的客户端节点在一个比较大的集群中是非常有用的，他协调主节点和数据节点，客户端节点加入集群可以得到集群的状态，根据集群的状态可以直接路由请求。

2. 数据节点<br/>
　　数据节点主要是存储索引数据的节点，主要对文档进行增删改查操作，聚合操作等。数据节点对cpu，内存，io要求较高， 在优化的时候需要监控数据节点的状态，当资源不够的时候，需要在集群中添加新的节点。

3. 主节点<br/>
　 主资格节点的主要职责是和集群操作相关的内容，如创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点。稳定的主节点对集群的健康是非常重要的，默认情况下任何一个集群中的节点都有可能被选为主节点，索引数据和搜索查询等操作会占用大量的cpu，内存，io资源，为了确保一个集群的稳定，分离主节点和数据节点是一个比较好的选择。  

4. 建议<br/>
　　在一个生产集群中我们可以对这些节点的职责进行划分，建议集群中设置3台以上的节点作为master节点，这些节点只负责成为主节点，维护整个集群的状态。再根据数据量设置一批data节点，这些节点只负责存储数据，后期提供建立索引和查询索引的服务，这样的话如果用户请求比较频繁，这些节点的压力也会比较大，所以在集群中建议再设置一批client节点(node.master: false node.data: false)，这些节点只负责处理用户请求，实现请求转发，负载均衡等功能。

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
1. 该master-eligible节点的当前状态不是master
2. 该master-eligible节点通过ZenDiscovery模块的ping操作询问其已知的集群其他节点，没有任何节点连接到master
3. 包括本节点在内，当前已有超过minimum_master_nodes个节点没有连接到master
    总结一句话，即当一个节点发现包括自己在内的多数派的master-eligible节点认为集群没有master时，就可以发起master选举
### 2：当需要选举master时，选举谁
  先根据节点的clusterStateVersion比较，clusterStateVersion越大，优先级越高；clusterStateVersion相同时，进入compareNodes，其内部按照节点的Id比较(Id为节点第一次启动时随机生成)
  clusterStateVersion：集群的版本，是由master节点维护。master选举完成后，会将期version广播到其他节点。
1. 当clusterStateVersion越大，优先级越高。这是为了保证新Master拥有最新的clusterState(即集群的meta)，避免已经commit的meta变更丢失。因为Master当选后，就会以这个版本的clusterState为基础进行更新
2. 当clusterStateVersion相同时，节点的Id越小，优先级越高。即总是倾向于选择Id小的Node，这个Id是节点第一次启动时生成的一个随机字符串。之所以这么设计，应该是为了让选举结果尽可能稳定，不要出现都想当master而选不出来的情况
### 3：什么时候选举成功
  当NodaA发起选举时
#### A：NodeA选举NodeB为master时，
1. 如果Node_B已经成为Master，Node_B就会把Node_A加入到集群中，然后发布最新的cluster_state, 最新的cluster_state就会包含Node_A的信息。相当于一次正常情况的新节点加入。对于Node_A，等新的cluster_state发布到Node_A的时候，Node_A也就完成join了
2. 如果Node_B在竞选Master，那么Node_B会把这次join当作一张选票。对于这种情况，Node_A会等待一段时间，看Node_B是否能成为真正的Master，直到超时或者有别的Master选成功
3. 如果Node_B认为自己不是Master(现在不是，将来也选不上)，那么Node_B会拒绝这次join。对于这种情况，Node_A会开启下一轮选举。
#### B：NodeA选自己当Master
- 此时NodeA会等别的node来join，即等待别的node的选票，当收集到超过半数的选票时，认为自己成为master，然后变更cluster_state中的master node为自己，并向集群发布这一消息

## nodes节点的关闭与开启
1. master节点关闭<br/>
假设有多个master候选节点(A，B，C)，如果当前B是master节点，则需要在一分钟后，会重新选举master节点(见master选举方式)。并且，B节点上的数据，会根据分片数量、副本数量重新分配到各个节点。
2. data节点关闭<br/>
假设有多个数据节点(D，E，F)，如果关闭D节点，当master确定节点失效后，会根据分片数量、副本数量重新分配到各个节点。

