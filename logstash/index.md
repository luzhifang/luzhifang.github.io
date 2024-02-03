# Logstash


## 简介

Logstash 可以收集日志数据并对数据进行处理后传递给 ES，但是现阶段通常使用它做数据分析和过滤工作，采集工作交由 filebeat 完成。

因为使用它采集日志的话需要在所有需要收集日志的服务器上安装 Logstash，由于 Logstash 偏重量级，现在通常不会这样做，而是交给各种 beat 去完成这个工作，然后最后交给 Logstash 处理后再传递给 ES。

如果数据量太大可以在 agent 与 server 端之前部署消息队列，一般使用 Redis 或者 Kafka。

## 安装运行

1. 二进制方式安装

最新包到官网[下载](https://www.elastic.co/cn/downloads/logstash)

运行以下命令：

```Shell
cd /usr/local
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.17.6-linux-x86_64.tar.gz
tar zxvf logstash-7.17.6-linux-x86_64.tar.gz
chown -R elastic. /usr/local/logstash-7.17.6/
su elastic
/usr/local/logstash-7.17.6/bin/logstash -f /usr/local/logstash-7.17.6/config/logstash-sample.conf
```

2. docker 方式安装

```
docker run -it docker.elastic.co/logstash/logstash:7.17.6
```

## 工作原理

logstash 处理流程有三步：inputs -> filters -> outputs。

- inputs：负责生产数据
- filters：修改数据
- outputs：搬运数据到其他地方

### inputs

inputs 负责获取数据到 logstash，以下是一些通用的 inputs：

- file：从文件系统上的文件读取，很像 UNIX 命令 `tail -0F`
- syslog：监听知名端口 514 上的 syslog 消息，并根据 RFC3164 格式进行解析
- redis：读取 redis channels 或者 lists
- beats：由 Beats 发送过来，比如 Filebeat
- kafka：从 kafka topic 读取

更多 inputs，请访问[链接](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)

### filters

filters 是 Logstash 管道中的中间处理设备。如果事件符合某些条件，可以将筛选器与条件组合在一起对其执行操作。常用的 filters 包括：

- grok: 转换和构造任意文本。Grok 是目前 Logstash 中将非结构化日志数据解析为结构化可查询数据的最佳方法。Logstash 内置了 120 种模式，您很可能会找到满足您需求的模式! 文档见[链接](https://www.elastic.co/guide/en/logstash/8.5/plugins-filters-grok.html)
- mutate: 对事件字段执行通用转换。您可以重命名、删除、替换和修改事件中的字段。
- drop: 完全删除事件，如 debug 事件。
- clone: 复制一个事件，可能添加或删除字段。
- geoip: 添加 IP 地址的地理位置信息(也可以在 Kibana 中显示惊人的图表!)
- json: 处理 json 格式的文本，文档见[链接](https://www.elastic.co/guide/en/logstash/8.5/plugins-filters-json.html)

更多 filters，请访问[链接](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

### outputs

outputs 是 Logstash 管道的最后阶段。一个事件可以通过多个输出，但是一旦所有输出处理完成，事件就完成了它的执行。常用的 outputs 包括:

- elasticsearch: 将事件数据发送到 elasticsearch。如果你计划将你的数据以一种高效、方便、易于查询的格式保存，Elasticsearch 就是你的选择。
- file: 将事件数据写入磁盘上的文件。
- Graphite: 将事件数据发送到 Graphite，这是一种流行的用于存储和绘制指标的开源工具。http://graphite.readthedocs.io/en/latest/
- statsd: 将事件数据发送到 statsd，该服务“侦听统计数据，如计数器和计时器，通过 UDP 发送，并将聚合发送到一个或多个可插入的后端服务”。如果您已经在使用 statsd，这可能对您很有用!

更多 outputs，请访问[链接](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)

## 配置文件

在正式的生产环境中要启动 Logstash 需要配置一系列复杂的规则以满足需求，所以就需要通过配置文件来自定义这些规则。

Logstash 和 ElasticSearch 一样有 2 个配置文件需要调整，其中 jvm.options 和 Elasticsearch 中的一样的用于优化堆栈内存，另一个 config/logstash.yml，通过该配置文件来进行采集和过滤数据等操作。

自定义配置文件(logstash-sample.conf)时主要配置的内容包括 input（用于指定输入源，包含本地日志、redis、stdin 标准输入等多种方式）、filter（进行规则过滤和处理）、output（指定输出目标）三个区。在 Logstash 中输入的数据可以通过很多种方式获取，比如本地日志文件、filebeat、数据库等。而输出也可以指定到需要的容器中，如 Elasticsearch

配置示例如下：

```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
}
```

## 参考

https://www.elastic.co/guide/en/logstash/current/pipeline.html

https://www.linuxe.cn/post-309.html

