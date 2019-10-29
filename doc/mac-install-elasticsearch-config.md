# mac环境下，安装与配置

* 安装(确认java版本在1.8以上)
    登录官网：http://www.elastic.co，下载elasticsearch<br>
    进入bin目录，执行 ./elasticsearch，即可运行elasticsearch<br>
    验证方式，浏览器输入：localhost:9200（或127.0.0.1:9200）
    
* head插件(如果需要可以安装kibana)
    点击下载elasticsearch-head : （页面地址：https://github.com/mobz/elasticsearch-head）<br>
    安装node.js和npm，查看node版本：node -v<br>
    进入elasticsearch-head的目录下，启动elasticsearch-head：npm run start<br>
    当显示成功后，验证方式：localhost:9100<br>
    
* 由于head插件与ElasticSearch是两个独立的进程，它们之间的访问有跨域问题。
    需要对ElasticSearch的配置进行相应的修改(.../elasticsearch/config/elasticsearch.yml)。<br>
    在尾部添加<br>
    http.cors.enabled: true<br>
    http.cors.allow-origin: "*"<br>
    重新启动elasticsearch<br>
* 安装分布式elasticsearch
    * 配置master节点，修改elasticsearch.yml。尾部添加：<br>
        \#集群名称 <br>
        cluster.name: xiaoming<br>
        \#master名称<br>
        node.name: master<br>
        node.master: true<br>
        
        network.host: 127.0.0.1<br>
        退出es并重新启动。<br>
    * 配置两个子节点，创建es_slave文件夹，存入子节点服务。<br>
        copy两份es的原文件，到es_slave中，并分别命令为：es_slave1, es_slave2；如果在copy的data文件夹下有数据，将其下的数据全部删除<br>
        在es_slave1/config/elasticsearch.yml尾部添加<br>
        cluster.name: xiaoming<br>
        node.name: slave1<br>
        
        network.host: 127.0.0.1<br>
        http.port: 8200<br>
        discovery.zen.ping.unicast.hosts: ["127.0.0.1"]<br>
        
        在es_slave2/config/elasticsearch.yml尾部添加<br>
        cluster.name: xiaoming<br>
        node.name: slave2<br>
        
        network.host: 127.0.0.1<br>
        http.port: 8300<br>
        discovery.zen.ping.unicast.hosts: ["127.0.0.1"]<br>
        如果配置成功那么在es_head中查看到的内容如下：
        ![es分布式安装](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/doc/image/es_slaves.png)
参考网站内容：https://www.cnblogs.com/liuxiaoming123/p/8081883.html
