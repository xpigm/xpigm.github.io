---
layout: post
title:  Docker Compose构建ES集群
categories: other 
tags: [other]
no-post-nav: true
comments: true
---

## 1. 安装DOCKER

https://www.cnblogs.com/yufeng218/p/8370670.html

## 2.docker-compose 配置

以下操作均以  *Elasticsearch-5.3.0* 为例，主机IP为10.1.1.1。

最终组建一个包含五个节点端口分别为9201-9205、9301-9305，数据存储在/data/es-data目录的单机集群。

### 2.1. 创建目录

创建数据存储目录

```
mkdir -p /data/es-data
```



docker容器运行ElasticSearch默认以**elasticsearch** 用户（或`1000:1000`）执行，映射到容器的外部资源需要被授予这个用户访问权限

``` shell
chmod +777 ./node
```

### 2.2. docker-compose.yml配置

```yaml
version: '2'
services:
  elasticsearch1:
    build: .    
    image: elasticsearch-plugins
    container_name: elasticsearch1
    environment:
      - node.name=node-1
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - discovery.zen.minimum_master_nodes=3
      - network.publish_host=10.1.1.1
      - transport.tcp.port=9301
      - http.port=9201
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 8g
    volumes:
      - /data/es-data/elasticsearch1/data:/usr/share/elasticsearch/data
    ports:
      - 9201:9201
      - 9301:9301
    networks:
      - esnet
  
  elasticsearch2:
    image: elasticsearch-plugins
    container_name: elasticsearch2
    environment:
      - node.name=node-2
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - "discovery.zen.ping.unicast.hosts=10.1.1.1:9301"
      - discovery.zen.minimum_master_nodes=3
      - network.publish_host=10.1.1.1
      - transport.tcp.port=9302
      - http.port=9202
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 8g
    volumes:
      - /data/es-data/elasticsearch2/data:/usr/share/elasticsearch/data
    ports:
      - 9202:9202
      - 9302:9302
    networks:
      - esnet

  elasticsearch3:
    image: elasticsearch-plugins
    container_name: elasticsearch3
    environment:
      - node.name=node-3
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - "discovery.zen.ping.unicast.hosts=10.1.1.1:9301"
      - discovery.zen.minimum_master_nodes=3
      - network.publish_host=10.1.1.1
      - transport.tcp.port=9303
      - http.port=9203
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 8g
    volumes:
      - /data/es-data/elasticsearch3/data:/usr/share/elasticsearch/data
    ports:
      - 9203:9203
      - 9303:9303
    networks:
      - esnet

  elasticsearch4:
    image: elasticsearch-plugins
    container_name: elasticsearch4
    environment:
      - node.name=node-4
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - "discovery.zen.ping.unicast.hosts=10.1.1.1:9301"
      - discovery.zen.minimum_master_nodes=3
      - network.publish_host=10.1.1.1
      - transport.tcp.port=9304
      - http.port=9204
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 8g
    volumes:
      - /data/es-data/elasticsearch4/data:/usr/share/elasticsearch/data
    ports:
      - 9204:9204
      - 9304:9304
    networks:
      - esnet

  elasticsearch5:
    image: elasticsearch-plugins
    container_name: elasticsearch5
    environment:
      - node.name=node-5
      - cluster.name=es-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - "discovery.zen.ping.unicast.hosts=10.1.1.1:9301"
      - discovery.zen.minimum_master_nodes=3
      - network.publish_host=10.1.1.1
      - transport.tcp.port=9305
      - http.port=9205
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 8g
    volumes:
      - /data/es-data/elasticsearch5/data:/usr/share/elasticsearch/data
    ports:
      - 9205:9205
      - 9305:9305
    networks:
      - esnet

  elasticsearch-head:
        image: mobz/elasticsearch-head:5
        container_name: elasticsearch-head
        ports:
           - "9100:9100"           

  cerebro:
    image: yannart/cerebro:latest
    ports:
      - "9000:9000"
    networks:
      - esnet

networks:
  esnet:
```



- discovery.zen.ping.unicast.hosts

  It is recommended that the unicast hosts list be maintained as the list of master-eligible nodes in the cluster.

- ES_JAVA_OPTS

  一般设置为`mem_limit`的50%，但是最好不要超过32g。

  - oops

    jvm使用名为oops或Ordinary Object Pointers的数据结构来表示对象，这些oops等同于C指针。instanceOops是一种特殊的oop，它表示Java中的对象实例。jvm对instanceOops填充了0，使其32位对齐，在64位系统上则是64位对齐。

  - 内存压缩

    得益于jvm的内存压缩技术，在64位系统中使用了32位内存地址，这样减少了内存地址的空间占用和gc时间，对性能也有一定的提升。由于instanceOops进行了内存对齐，因此在64位系统上它的内存地址后3位始终是0，于是就可以右移三位把35位内存地址压缩位32位。这意味着jvm可以在不使用64位内存地址的情况下使用2<sup>32+3</sup>=2<sup>35</sup>=32GB 的堆空间，当堆内存大于32GB，jvm将关闭oop压缩。
    
  - 超过32GB情况下的内存压缩
  
    instanceOops默认使用8字节（64位）对齐，但是可以通过调整 `-XX：ObjectAlignmentInBytes` 来改变此默认参数以实现更大堆内存下的压缩。指定的值应为2的幂，并且必须在8和256的范围内。**增加对齐的内存值相应的也对增加对象的大小。**



### 2.4. 启动集群

初始化集群（包括构建镜像、创建容器、创建网络，启动容器等相关操作）

``` shell
docker-compose up
```

关闭并移除集群（相对于 `up`）

```shell
docker-compose down
```


启动

```
docker-compose start
```

关闭

```
docker-compose stop
```



验证状态

```shell
curl http://localhost:9201/_cluster/health
```


## 3. 

### 3.1. 物理集群

关于ES物理集群的搭建，主要是要把不同容器的放在同一网络环境下，以内部容器跨主机通信。不同的网络环境采用的互通发案也不太相同。这里采用了最直接的一种方式——利用物理机互联的网络环境，通过物理机的不同端口把服务暴露出去。很蠢也很有效。单机集群本身意义不大，docker-compose本身更适用于复杂运行环境的快速搭建。



### 3.2. 可能遇到的错误
- 执行docker-compose up之后，ssh连接断开，而且ping不通，则可能是docker网段与本机网段冲突导致，解决方案移步[这里](https://www.google.com/)。

