# spring boot + elasticsearch

* elasticsearch的几个关键概念(与关系型数据库做下类比)
    * index索引：类比于mysql的database   
    * type类型：类比与mysql的table
    * document：类比与mysql的row
    * field: 类比与mysql的column
    * shard分片：即你存入的对象或数据分几段存储
    * replicas副本：数据存储几个副本
    * settings与mappings，示例如下：<br>
     ```el
       PUT /my_index
       {
           "settings":{
                  "number_of_shards":3,   
                  "number_of_replicas":0
               },
           "mappings":{
                 "student":{ 
                      "properties":{
                        "name":{
                          "type":"text",
                          "analyzer": "standard"
                        },
                        "age":{
                          "type":"text",
                           "analyzer": "standard"
                        }
                       }
       
                   }
           }
       }
     ```
 * spring boot代码实现
     * yml配置文件
     ```yaml
        spring:
          data:
            elasticsearch:
              cluster-name: liuhan #默认为elasticsearch
              cluster-nodes: 127.0.0.1:9300, 127.0.0.1:9301, 127.0.0.1:9302 #配置es节点信息，逗号分隔，如果没有指定，则启动ClientNode
              repositories.enabled: true
              properties:
                path:
                  logs: ./elasticsearch/log #elasticsearch日志存储目录
                  data: ./elasticsearch/data #elasticsearch数据存储目录
     ```
     这个地方的port并不是eclasticsearch.yml中的端口，而是es-head上看到的。如下图：
     
     * 多个node配置文件
     ```java
        @Configuration
        public class ESConfig {
            @Bean
            public TransportClient client() throws UnknownHostException {
        
                //es集群连接
                TransportAddress master = new InetSocketTransportAddress(
                        InetAddress.getByName("localhost"),
                        9300
                );
                TransportAddress esSlave01 = new InetSocketTransportAddress(
                        InetAddress.getByName("localhost"),
                        9301
                );
                TransportAddress esSlave02 = new InetSocketTransportAddress(
                        InetAddress.getByName("127.0.0.1"),
                        9302
                );
        
                //es集群配置（自定义配置） 连接自己安装的集群名称
                Settings settings = Settings.builder()
                        .put("cluster.name","liuhan")
                        .build();
        
                PreBuiltTransportClient client = new PreBuiltTransportClient(settings);
        
                client.addTransportAddress(master);
                client.addTransportAddress(esSlave01);
                client.addTransportAddress(esSlave02);
        
                return client;
            }
        }
    ```
    * 实体类配置
    ```java
        @Getter
        @Setter
        @NoArgsConstructor
        @Document(indexName = "log", type = "operate_log", shards = 5, replicas = 1, refreshInterval = "-1", createIndex = true)
        public class OptLog {
            @Id
            private String id;
            /**
             * 用户id
             */
            private Long userid;
            /**
             * 用户唯一标识
             */
            private String useruniquecode;
            /**
             * 用户名
             */
            private String username;
            /**
             * 操作类型
             */
            private String operatetype;
            /**
             * 操作时间
             */
            private String operatetime;
            /**
             * 操作备注
             */
            private String operatenote;
            /**
             * 对象类型
             */
            private String domaintype;
            /**
             * 对象id
             */
            private String domainid;
            /**
             * 请求表单
             */
            private String requestmodel;
        
            private Date createddate;
        }
    ```
    * 存储（repository）配置
    ```java
    import com.yjpfj1203.statistics.entity.OptLog;
    import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
    import org.springframework.stereotype.Repository;
    
    @Repository
    public interface OptLogRepository extends ElasticsearchRepository<OptLog, String>{
    }
    ```
    * controller层代码
    ```java
    @RestController
    @RequestMapping("/logs")
    @EnableSwagger2
    public class OptLogController {
        @Autowired
        private OptLogRepository optLogRepository;
           
        /**
         * 1、列表查询
         */
        @GetMapping
        public List<OptLog> findAll() {
            return CollectionUtil.iterableToCollectionJava8(optLogRepository.findAll());
        }
    
        /**
         * 2、通过id查询
         * @param id
         * @return
         */
        @GetMapping("/{id}")
        public OptLog findById(@PathVariable String id) {
            return optLogRepository.findById(id).orElse(null);
        }
   
        /**
         * 3、添加记录
         * @param optLog
         * @return
         */
        @PostMapping
        public OptLog insert(@RequestBody OptLog optLog) {
            optLogRepository.save(optLog);
            return optLog;
        }
    
        /**
         * 4、删 id
         * @param id
         * @return
         */
        @DeleteMapping("/{id}")
        public OptLog delete(@PathVariable String id) {
            OptLog optLog = optLogRepository.findById(id).orElse(null);
            optLogRepository.deleteById(id);
            return optLog;
        }

    }
```