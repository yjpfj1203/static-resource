```
http.cors.enabled: true
#是否开启跨域访问

http.cors.allow-origin: "*"
#开启跨域访问后的地址限制，*表示无限制

cluster.name: ES-Cluster
#ES集群名称，同一个集群内的所有节点集群名称必须保持一致

node.name: ES-master-10.150.55.94
#ES集群内的节点名称，同一个集群内的节点名称要具备唯一性

node.master: true
#允许节点是否可以成为一个master节点，ES是默认集群中的第一台机器成为master，如果这台机器停止就会重新选举

node.data: false
#允许该节点存储索引数据（默认开启）
#关于Elasticsearch节点的角色功能详解，请看：https://www.dockerc.com/elasticsearch-master-or-data/

path.conf: /path/to/conf
# 设置配置文件的存储路径，默认是es根目录下的config文件夹。

path.data: /path/to/data
# 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：
# path.data: /path/to/data1,/path/to/data2

path.work: /path/to/work
# 设置临时文件的存储路径，默认是es根目录下的work文件夹。

path.logs: /path/to/logs
# 设置日志文件的存储路径，默认是es根目录下的logs文件夹 

path.plugins: /path/to/plugins
# 设置插件的存放路径，默认是es根目录下的plugins文件夹, 插件在es里面普遍使用，用来增强原系统核心功能。

cluster.initial_master_nodes: ["10.150.55.94:9301", "10.150.55.95:9301","10.150.30.246:9301"]
#Elasticsearch7新增参数，写入候选主节点的设备地址，来开启服务时就可以被选为主节点

discovery.zen.ping.unicast.hosts: ["10.150.55.94:9301", "10.150.55.95:9301","10.150.30.246:9301"]   
#在Elasticsearch7.0版本已被移除，配置错误
#写入候选主节点的设备地址，来开启服务时就可以被选为主节点
#默认主机列表只有127.0.0.1和IPV6的本机回环地址
#上面是书写格式，discover意思为发现，zen是判定集群成员的协议，unicast是单播的意思，ES5.0版本之后只支持单播的方式来进行集群间的通信，hosts为主机
#总结下来就是：使用zen协议通过单播方式去发现集群成员主机，在此建议将所有成员的节点名称都写进来，这样就不用仅靠集群名称cluster.name来判别集群关系了

discovery.zen.minimum_master_nodes: 2           
#在Elasticsearch7.0版本已被移除，配置无效
#为了避免脑裂，集群的最少节点数量为，集群的总节点数量除以2加一

discovery.zen.fd.ping_timeout: 120s             
#在Elasticsearch7.0版本已被移除，配置无效
#探测超时时间，默认是3秒，我们这里填120秒是为了防止网络不好的时候ES集群发生脑裂现象

discovery.zen.fd.ping_retries: 6                
#在Elasticsearch7.0版本已被移除，配置无效
#探测次数，如果每次探测90秒，连续探测超过六次，则认为节点该节点已脱离集群，默认为3次

discovery.zen.fd.ping_interval: 15s             
#在Elasticsearch7.0版本已被移除，配置无效
#节点每隔15秒向master发送一次心跳，证明自己和master还存活，默认为1秒太频繁,

discovery.seed_hosts: ["10.150.55.94:9301", "10.150.55.95:9301","10.150.30.246:9301"]
#Elasticsearch7新增参数，写入候选主节点的设备地址，来开启服务时就可以被选为主节点,由discovery.zen.ping.unicast.hosts:参数改变而来

index.number_of_shards: 5
# 设置默认索引分片个数，默认为5片。

index.number_of_replicas: 1
# 设置默认索引副本个数，默认为1个副本。如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配
# （健康度可用 curl 'localhost:9200/_cat/health?v' 查看， 分为绿色、黄色或红色。绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用）。

bootstrap.memory_lock: true
#在ES运行起来后锁定ES所能使用的堆内存大小，锁定内存大小一般为可用内存的一半左右；锁定内存后就不会使用交换分区
#如果不打开此项，当系统物理内存空间不足，ES将使用交换分区，ES如果使用交换分区，那么ES的性能将会变得很差

network.host: 10.150.55.94
#es绑定地址，支持IPv4及IPv6，默认绑定127.0.0.1；es的HTTP端口和集群通信端口就会监听在此地址上

network.tcp.no_delay: true
#是否启用tcp无延迟，true为启用tcp不延迟，默认为false启用tcp延迟

network.tcp.keep_alive: true
#是否启用TCP保持活动状态，默认为true

network.tcp.reuse_address: true
#是否应该重复使用地址。默认true，在Windows机器上默认为false

network.tcp.send_buffer_size: 128mb
#tcp发送缓冲区大小，默认不设置

network.tcp.receive_buffer_size: 128mb
#tcp接收缓冲区大小，默认不设置

transport.tcp.port: 9301
#设置集群节点通信的TCP端口，默认就是9300

transport.tcp.compress: true
#设置是否压缩TCP传输时的数据，默认为false

http.max_content_length: 200mb
#设置http请求内容的最大容量，默认是100mb

gateway.type: local
# gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。

gateway.recover_after_nodes: 1
# 设置集群中N个节点启动时进行数据恢复，默认为1。

gateway.recover_after_time: 5m
# 设置初始化数据恢复进程的超时时间，默认是5分钟。

gateway.expected_nodes: 2
# 设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。

indices.recovery.max_size_per_sec: 0
# 设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。

indices.recovery.concurrent_streams: 5
# 设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。

cluster.fault_detection.leader_check.interval: 15s 
#Elasticsearch7新增参数，设置每个节点在选中的主节点的检查之间等待的时间。默认为1秒

discovery.cluster_formation_warning_timeout: 30s 
#Elasticsearch7新增参数，启动后30秒内，如果集群未形成，那么将会记录一条警告信息，警告信息未master not fount开始，默认为10秒

cluster.join.timeout: 30s
#Elasticsearch7新增参数，节点发送请求加入集群后，在认为请求失败后，再次发送请求的等待时间，默认为60秒

cluster.publish.timeout: 90s 
#Elasticsearch7新增参数，设置主节点等待每个集群状态完全更新后发布到所有节点的时间，默认为30秒

cluster.routing.allocation.cluster_concurrent_rebalance: 32
#集群内同时启动的数据任务个数，默认是2个

cluster.routing.allocation.node_concurrent_recoveries: 32
#添加或删除节点及负载均衡时并发恢复的线程个数，默认4个

cluster.routing.allocation.node_initial_primaries_recoveries: 32
#初始化数据恢复时，并发恢复线程的个数，默认4个
```
