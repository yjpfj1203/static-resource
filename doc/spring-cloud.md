# spring cloud

* ##config
  * 自定义参数
    ```yaml
      book: 
          name: bookName
          author: bookAuthor
          description: ${bookAuthor} is writing 《${bookName}》 
      ```
    引用方式
    ```java
    @Getter
    @Component
    public class Book {
        @Value(value = "${book.name}")
        private String bookName;
        
        @Value("${bookAuthor}")
        private String bookeAuthor;
        
        @Value("${description}")
        private String description;
    }
    ```
  * 命令行参数
    例：java -jar xxx.jar --server.port = 8888<br>
    命令行参数可以覆盖yml或properties中的参数设置<br>
  * 多环境配置
    多环境配置文件名需要满足application-{profile}.yml的格式<br>
    四种配置文件：application.yml， application-profile.yml, applicationName.yml, applicationName-profile.yml<br>
    从后向前，会逐层覆盖<br>
  * 配置文件加载顺序<br>
    1：命令行参数 <br>
    2：java:comp/env中的JNDI属性<br>
    3：java的系统属性，即可通过System.getProperties()获得的内容<br>
    4：操作系统的环境变量<br>
    5：random.*配置的环境变量<br>
    6：jar外，针对不同{profile}环境配置文件的内容<br>
    7：jar内，针对不同{profile}环境配置文件的内容<br>
    8：jar外，application.yml的配置<br>
    9：jar内，application.yml的配置<br>
    10：在@Configuration注释修改的内容，骑过@PropertySource注解定义的属性<br>
    11：系统默认属性，使用SpringApplication.setDefaultProperties定义的内容<br>
    优先级，数字越小，优先级越高
* ##spring cloud eureka
  * ###搭建eureka服务注册中心
    * 引入依赖
      ```xml
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-eureka-server</artifactId>
      </dependency>
      ```
    * 注解@EnableEurekaService，启动服务中心。
      ```java
      @SpringBootApplication
      @EnableEurekaService
      public class  EurekaServiceApplication {
          public static void main(String[] args) {
              ApplicationContext ctx = SpringApplication.run(EurekaServiceApplication.class, args);
              AppContextUtil.setApplicationContext(ctx);
          }
      }
      ```
    * 配置服务端
      ```yaml
        server:
          port: 1111
        eureka:
          instance:
            hostname: localhost
          client:
            register-with-eureka: false  //不向注册中心注册自己
            fetch-registry: false //不去检索服务
            serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
      ```
    * 向注册中心注册服务<br>
      1：引入pom
        ```xml
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-eureka-server</artifactId>
          </dependency>
        ```
      2：启动类注解@EnableDiscoveryClient，激活DiscoveryClient实现<br>
      3：添加配置，指向服务注册中心
        ```yaml
          spring:
            application:
              name: applicationName //向注册中心注册的服务名称
          eureka.client.serviceUrl.defaultZone: http://localhost:1111/eureka/
        ```
    * 高可用的注册中心<br>
      1：在注册中心添加两个配置文件application-peer1.yml, application-peer2.yml，分别用于启动两个注册中心，内容如下
        ```yaml
          spring:
            application:
              name: eureka-server
          server:
            port: 1111
          eureka:
            instance:
              hostname: peer1
            client:
              register-with-eureka: true
              serviceUrl.defaultZone: http://peer2:1112/eureka/
        ```
        ```yaml
          spring:
            application:
              name: eureka-server
          server:
            port: 1112
          eureka:
            instance:
              hostname: peer2
            client:
              register-with-eureka: true
              serviceUrl.defaultZone: http://peer1:1111/eureka/
        ```
        在/etc/hosts文件中添加peer1，peer2的ip转换
        ```text
          127.0.0.1 peer1
          127.0.0.1 peer2
        ```
        启动peer1与peer2服务命令如下<br>
        java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer1，启动注册中心peer1，激活application-peer1.yml<br>
        java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer2，启动注册中心peer2，激活application-peer2.yml<br>
      2：将业务服务server向两个注册中心注入
        ```yaml
          spring:
            application:
              name: applicationName //向注册中心注册的服务名称
          eureka.client.serviceUrl.defaultZone: http://peer1:1111/eureka/,http://peer2:1111/eureka/
        ```
      3：如果我们也启动两个业务服务server，都向服务注册中心注入，那么就不用担心任何一个服务挂掉了。<br>
        这里有三个重要的概念：服务注册中心、服务提供者、服务消费者。<br>
        服务注册中心即为eureka-server，如果A-server，B-server都注册在服务注册中心，A-server调用了B-server的服务，那么B即为服务提供者，A-server即为服务消费者。<br>
      4：服务提供者的服务续约<br>
        服务提供者会维护一个心跳来持续告诉Eureka Server以防止被剔除。配置如下：
        ```yaml
        eureka:
          instance:
            lease-renewal-interval-in-secends: 30  //服务续约时间间隔
            lease-expiration-duration-in-seconds: 90  //服务失效时间
        ```
        重启或关闭时，会导致服务下线，服务提供者会主要告诉服务注册中心，由它传播出去。<br>
      5：服务消费者的服务发现<br>
        Eureka Server会维护一个只读的服务清单来返回给客户端(服务消费者)<br>
        ```yaml
        eureka:
          client:
            fetch-registry: true  //开启获取服务提供者列表
            registry-fetch-interval-seconds: 30 //30秒获取一次服务清单
        ```
      6：服务注册中心<br>
         在服务注册中心启动时会创建一个定时服务，默认每隔60秒会关当前清单中超时(默认90秒)且没有续约的服务剔除出去。<br>
         自我保护，Eureka会统计心跳失败在15分钟内低于85%的服务，Eureka server会将其注册信息保护起来，让期不会过期。所以在开发的时候，一般使用下面的配置，关闭自我保护。<br>
         eureka.server.enable-self-preservation: false<br>
         com.netflix.appinfo.EurekaInstanceConfig是eureka.instance.后面可以配置的所有属性![eureka.instance](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/elastic-search-doc/image/es_slaves.png)<br>
         com.netflix.discovery.EurekaClientConfig是eureka.client.后面可以配置的所有属性<br>
         com.netflix.eureka.EurekaServerConfig是eureka.server.后面可以配置的所有属性<br>