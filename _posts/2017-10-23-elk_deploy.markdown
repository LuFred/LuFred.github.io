---
layout: post
title: "ELK安装部署"
subtitle: "Elasticsearch、Logstash、Kinaba、FileBeat 安装和部署"
date: 2017-10-23
author: LuJiangBo
tags: ELK
keyword: Elasticsearch,Logstash,Kinaba,FileBeat 
finished: true
---


## 原因及目的   
实现日志的自动采集、分析、展示   
## 工具介绍  
* Elasticsearch：是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。  
* Logstash：是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。  

* Kibana： 是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。  
* Filebeat：是一个日志文件托运工具，在你的服务器上安装客户端后，filebeat会监控日志目录或者指定的日志文件，追踪读取这些文件（追踪文件的变化，不停的读），并且转发这些信息到elasticsearch或者logstarsh中存放。

## 环境
> CentOS

## 依赖
> java 8及以上

## 安装
> 1.java环境  
> 2.Elasticsearch  
> 3.Kibana  
> 4.Logstash  
> 5.FileBeat  
* ps:安装的优先级并不需要严格分先后。  

## 1.安装java环境  

* 下载的java jdk
    * 这里我就下载当前最新的[9.0.1](http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html)  
这里在后面安装Logstash时会爆出一个错误(不支持java9)
* 解压
    * tar -xzf jdk-9.0.1_linux-x64_bin.tar.gz -C /usr/src
* 设置全局环境变量
    * export JAVA_HOME=/usr/lib/jvm/java**
    * export PATH=$PATH:$JAVA_HOME/bin
* 检查java环境是否安装成功
    *  命令：java -version  
    ```
    java version "9.0.1"  
    Java(TM) SE Runtime Environment (build 9.0.1+11)  
    Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)  
    ```
出现上述内容表示jdk配置成功

## 安装Elasticsearch
* 下载安装包
    * curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.3.tar.gz  
* 解压
    * tar -xvf elasticsearch-5.6.3.tar.gz  
* 跳到安装目录
    * cd elasticsearch-5.6.3/bin
* 运行
    * ./elasticsearch

注意点：  
> 若tar指定目录提示权限拒绝，可在tar前加sudo 用管理员权限执行
> 若执行./elasticsearch提示权限拒绝，可修改elastic内各文件夹的组权限

## 安装Kibana

* 下载安装包
    * wget https://artifacts.elastic.co/downloads/kibana/kibana-5.6.3-linux-x86_64.tar.gz  
* 解压
    * sudo tar -xzf kibana-5.6.3-linux-x86_64.tar.gz -C /usr/src
* 跳到指定目录
    * cd /usr/src/kibana-5.6.3-linux-x86_64/
* 运行
    * ./bin/kibana  

### kibana基本配置  
kibana的配置文件在config/kibana.yml中  
这里就说几个现在涉及到的配置点，其它的点可以后续再去具体了解  
* server.port: 服务的绑定端口号，默认是：5601  
* server.host: 服务绑定的ip地址，默认是localhost
* server.basePath: 如果在代理服务器后面运行，则可以指定安装Kibana的路径。 这仅影响由Kibana生成的URL，您的代理预期在将请求转发给Kibana之前删除basePath值。 此设置无法以斜杠（/）结尾。  
* server.maxPayloadBytes:服务请求的有效载荷，默认是1048567字节
* server.name: kibana的实例名，默认是主机名
* server.defaultRoute: kibana的主页地址,默认为/app/kibana
* server.customResponseHeaders:kibana对所有客户端响应统一添加的response header头部，默认{}
* elasticsearch.url: kabana监视的elasticsearch实例地址，默认http://localhost:9200
* elasticsearch.username: elasticsearch的账户
* elasticsearch.password: elasticsearch的密码
* elasticsearch.preserveHost:默认值：true当此设置的值为真时，Kibana使用server.host设置中指定的主机名。 当此设置的值为false时，Kibana将使用连接到此Kibana实例的主机的主机名。
* kibana.index:默认：“.kibana”Kibana使用Elasticsearch中的索引来存储保存的搜索，可视化和仪表板。 如果索引不存在，Kibana将创建一个新的索引。
* kibana.defaultAppId:默认值：“discover”要加载的默认应用程序。
* tilemap.url:Kibana用于在tilemap可视化中显示地图图块的tile服务的URL。 默认情况下，Kibana从外部元数据服务读取此URL，但用户仍然可以覆盖此参数以使用自己的Tile Map Service。 例如：“https://tiles.elastic.co/v2/default/{z}/{x}/{y}.png?elastic_tile_service_tos=agree&my_app_name=kibana”
* tilemap.options.minZoom: 默认值：1，最小缩放级别。
* tilemap.options.maxZoom: 默认值：10，最大缩放级别。
* tilemap.options.attribution: 默认：“©[弹性地图服务]（https://www.elastic.co/elastic-maps-service）”地图归属字符串。
* tilemap.options.subdomains: tile服务使用的子域数组，用token {s}指定子域的位置。

在这里需要修改的是  
* elasticsearch.url  
* elasticsearch.username  
* elasticsearch.password

访问:http://localhost:5601看到如下图表示kibana安装成功
![kibana]({{ post.url| prepend: site.url  }}/content/images/201710/elk_deploy01.jpg) 


## 安装logstash

* 下载安装包
    * wget https://artifacts.elastic.co/downloads/logstash/logstash-5.6.3.tar.gz
* 解压
    * sudo tar -xzf logstash-5.6.3.tar.gz -C /usr/src
* 跳到文件解压目录
    * cd /usr/src/logstash-5.6.3
* 运行
    * ./bin/logstash -f config/logstash.yml

测试：   
cd logstash-5.6.3  
./bin/logstash -e 'input { stdin { } } output { stdout {} }' -f config/logstash.yml

常见错误：  
```
Java HotSpot(TM) 64-Bit Server VM warning: Option UseParNewGC was deprecated in version 9.0 and will likely be removed in a future release.
Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by jnr.posix.JavaLibCHelper to method sun.nio.ch.SelChImpl.getFD()
WARNING: Please consider reporting this to the maintainers of jnr.posix.JavaLibCHelper
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
io/console on JRuby shells out to stty for most operations
NameError: cannot link Java class org.logstash.common.DeadLetterQueueFactory (java.lang.NoClassDefFoundError: javax/script/ScriptEngineFactory)
  get_proxy_or_package_under_package at org/jruby/javasupport/JavaUtilities.java:54

问题分析：这个问题主要系统中安装的java jdk是9.0.1,而UseParNewGC在9.0版本中已经被弃用，所以logstash找不到。
解决方案：如果可以修改系统中的java jdk版本，切回到1.8.*
这问题在logstash官网其实也有说明，目前logstash需要java8，且不支持java9
```
## 安装Filebeat  
* 下载文件包
    * wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.3-linux-x86_64.tar.gz
* 解压到/usr/src
    * sudo tar -xzf filebeat-5.6.3-linux-x86_64.tar.gz -C /usr/src/
* 运行
    * ./filebeat


### 配置filebeat日志获取路径
```
#=========================== Filebeat prospectors =============================

filebeat.prospectors:

# Each - is a prospector. Most options can be set at the prospector level, so
# you can use different prospectors for various configurations.
# Below are the prospector specific configurations.

- input_type: log

  # filebeat的日志获取路径
  paths:
    - /var/log/vida-service/*.log

#----------------------------- Logstash output --------------------------------
output.logstash:
  # 输出日志到logstash
  hosts: ["localhost:5044"]

```

## elk连通配置

### 1.配置logstash，让它能够接收filebeat的日志数据
* 假设日志格式为：  E1019 02:02:50.154436    5523 main.go:22] error message
* 在logstash目录下创建一个存放输入输出过滤配置文件的文件夹  
    * mkdir conf-pipeline
* 针对不同的日志来源创建独立的配置文件
    * cd conf-pipeline
    * touch vida-golang-service
    * 在文件中添加如下内容 
        ```
        # 定义数据源,这里从filebeat 7000端口获取数据源
        input {
            beats {
                port => "7000"
            }
        }
        # 定义数据格式
        # 属于logstash的一种日志解析器，需要独立安装插件
        # ./bin/logstash-plugin install logstash-filter-grok
        # 具体使用规则看参考官网https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
        # 这里要说的一点是grok是基于正则表达式的解析规则，已内置了上百种的日志解析规则,若你还想自定义解析规则的话
            可将规则写在独立的文件中，通过patterns_dir参数读取规则
        filter {
            grok {
                patterns_dir => ["./grok_patterns"]
                match => { "message" => "%{GOLOGERRORTYPE:err_type}%{GOLOGDATE:date} %{TIME:time}\s{1,}%{GOLOGTHREADID:thread_id} %{GOLOGFILE:file_name}:%{NUMBER:code_line}] %{GREEDYDATA:syslog_message}" }
            }
        }
        # 日志输出目标
        # 这里我们将解析过滤后的日志信息输出到elasticsearch和系统标准输出
        # 若安装了x-pack需要配置elastic的用户名密码
        # index是elastic到时候会创建的索引，便于搜索
        # 
        output {
            elasticsearch {
                hosts => ["localhost:9200"]
                index => "golog_log"
                user => elastic
                password => changeme
            }
            
            stdout { codec => rubydebug }
        }

        ```
    * 基于上面我们使用了自定义的grok解析规则，所以我们需要添加自定义的规则文件
        * 创建文件存放目录：mkdir grop_patterns
        * 进入目录创建一个规则文件:touch golog_format
        * 加入如下正则规则               
            ```
                # /usr/src/logstash-5.6.3/grok_patterns/golog_format 
                # error type
                GOLOGERRORTYPE  [a-zA-Z]{1}
                # date mmdd
                GOLOGDATE  [0-9]{4}
                # thread id
                GOLOGTHREADID  [0-9]{2,}
                #file
                GOLOGFILE  [a-zA-Z]*.[a-zA-Z]*
            ```
* 启动logstash：
    * ./bin/logstash -f conf-pipeline/


### 2.配置filebeat，使其将数据传给logstash
* 修改配置文件  
    ```
    #filebeat.yml文件中取消如下注释

    filebeat.prospectors:

    - input_type: log

    # Paths that should be crawled and fetched. Glob based paths.
    paths:
        - /var/log/*.*


    output.logstash:
    # The Logstash hosts
    hosts: ["localhost:5043"]

    ```
启动：./filebeat

若全部都配置正确，运行之后在logstash命令窗口应该能看到如下内容：
![kibana]({{ post.url| prepend: site.url  }}/content/images/201710/elk_deploy02.jpg)   
同时打开kibana

创建索引后应该能看到如下日志信息：
![kibana]({{ post.url| prepend: site.url  }}/content/images/201710/elk_deploy03.jpg)   
