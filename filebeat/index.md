# Filebeat


## 简介

首先 filebeat 是 Beats 中的一员。
　　 Beats 在是一个轻量级日志采集器，其实 Beats 家族有 6 个成员，早期的 ELK 架构中使用 Logstash 收集、解析日志，但是 Logstash 对内存、cpu、io 等资源消耗比较高。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。
目前 Beats 包含六种工具：

- Packetbeat：网络数据（收集网络流量数据）
- Metricbeat：指标（收集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）
- Filebeat：日志文件（收集文件数据）
- Winlogbeat：windows 事件日志（收集 Windows 事件日志数据）
- Auditbeat：审计数据（收集审计日志）
- Heartbeat：运行时间监控（收集系统运行时的数据）

## 工作原理

Filebeat 是用于转发和集中日志数据的轻量级传送工具。Filebeat 监视您指定的日志文件或位置，收集日志事件，并将它们转发到 Elasticsearch 或 Logstash 进行索引。

Filebeat 的工作方式如下：启动 Filebeat 时，它将启动一个或多个输入，这些输入将在为日志数据指定的位置中查找。对于 Filebeat 所找到的每个日志，Filebeat 都会启动收集器。每个收集器都读取单个日志以获取新内容，并将新日志数据发送到 libbeat，libbeat 将聚集事件，并将聚集的数据发送到为 Filebeat 配置的输出。

工作的流程图如下：

<img src="/images/elk/filebeat-architecture.png" alt="" width="80%" />

### Filebeat 的构成

Filebeat 由两个组件构成，分别是 input（输入）和 harvester（收集器）

- input(输入)：负责管理 harvester 并找到所有读取源。如果 input 类型是 log，则 input 将查找驱动器上与定义的路径匹配的所有文件，并为每个文件启动一个 harvester。
- harvester(收集器)：负责读取单个文件内容，每个文件启动一个

### Filebeat 如何保存文件的状态

- Filebeat 保留每个文件的状态，并经常将状态刷新到磁盘中的注册表文件中（默认在/var/lib/filebeat/registry）。
- 该状态用于记住 harvester 读取的最后一个偏移量，并确保发送所有日志行。如果无法访问输出（如 Elasticsearch 或 Logstash），Filebeat 将跟踪最后发送的行，并在输出再次可用时继续读取文件。
- 当 Filebeat 运行时，每个输入的状态信息也保存在内存中。当 Filebeat 重新启动时，来自注册表文件的数据用于重建状态，Filebeat 在最后一个已知位置继续每个 harvester。
- 对于每个输入，Filebeat 都会保留它找到的每个文件的状态。由于文件可以重命名或移动，文件名和路径不足以标识文件。对于每个文件，Filebeat 存储唯一的标识符，以检测文件是否以前被捕获。

### Filebeat 何如保证至少一次数据消费

- Filebeat 保证事件将至少传递到配置的输出一次，并且不会丢失数据。是因为它将每个事件的传递状态存储在注册表文件中。
- 在已定义的输出被阻止且未确认所有事件的情况下，Filebeat 将继续尝试发送事件，直到输出确认已接收到事件为止。
- 如果 Filebeat 在发送事件的过程中关闭，它不会等待输出确认所有事件后再关闭。当 Filebeat 重新启动时，将再次将 Filebeat 关闭前未确认的所有事件发送到输出。
- 这样可以确保每个事件至少发送一次，但最终可能会有重复的事件发送到输出。通过设置 shutdown_timeout 选项，可以将 Filebeat 配置为在关机前等待特定时间

## 安装与运行

1. 压缩包方式安装，点击[链接](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)查看最新版本，运行以下命令

```Shell
cd /usr/local
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-xxx-linux-x86_64.tar.gz
tar xzvf filebeat-xxx-linux-x86_64.tar.gz
```

2. [docker](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html) 方式安装

```Shell
docker run \
docker.elastic.co/beats/filebeat:8.5.3 \
setup -E setup.kibana.host=kibana:5601 \
-E output.elasticsearch.hosts=["elasticsearch:9200"]
```

编辑配置文件 filebeat.yml

```
filebeat.inputs: #采集系统日志
- type: log  #通常采集日志文件的话type就为log，还有stdin、redis、docker等其它类型
  enabled: true  #启用开关
  paths:       - /var/log/message.log
  #include_lines: ['^ERR','^WARN']  #只采集正则所包含的日志
  tags: ["system"]  #定义标签，采集多个日志的时候可以用标签来创建索引，也可以用logstash的type来区分
```

启动 Filebeat

```
./filebeat -e -c filebeat.yml
```

## filebeat 模块

Filebeat 内置有大量 module，比如 Nginx module、MySQL module。使用这些 module 可以简化日志收集的过程，包括 Kibana 出图，甚至可以通过官方 module 学习出图。

1. 开启 Filebeat 模块相关配置

```
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml #模块配置文件路径，查看支持的模块时就会从该目录去寻找配置文件
  reload.enabled: true ##开启模块功能
  reload.period: 10s
```

2. 查看当前支持的 module，`filebeat modules list`
3. 启动指定的 module，`filebeat modules enable mysql`
4. 配置 module，只需要修改日志路径 modules.d/mysql.yml

```
- module: mysql
  error:
    enabled: true
    var.paths: ["/var/log/mysql/log/slowlog/mysql.log"]
  slowlog:
    enabled: true
    var.paths: ["/var/log/mysql/log/error.log"]
```

5. 初始化环境，如果安装有多个 module 的话只用初始化一次。`filebeat setup -e`
6. 重启 filebeat，需要注意的是索引名是自动创建好了的，不要自定义，然后到 Kibana 中看图即可

## 各种组合架构

### Elasticsearch+Logstash+Filebeat 架构

如果 Filebeat 直接把日志传给 ES 的话，由于其不支持正则、无法移除字段等，没办法实现数据分析，所以更优的配置是将 Filebeat 收集到的日志输出给一台 Logstash 服务器，由 Logstash 处理后再给 ES。为了实现这个要求，就需要修改 Filebeat 的 output 段以及 Logstash 的 input 段。

1. 修改 filebeat.yml 的 output.logstash 配置

```
output.logstash:
  hosts: ["127.0.0.1:5044"]
```

2. 修改 logstash 相关配置

### Elasticsearch+Logstash+Kafka+Filebeat 架构（推荐架构）

当生产环境中有多台 Filebeat 同时向 Logstash 传输数据再交给 ES 入库时，后端服务器的负载会比较大，为了减轻这种压力可以在 Filebeat 和 Logstash 之间加上一层消息队列，比如 Redis、Kafka。由于 Redis 太吃内存，而 Kafka 是基于磁盘存储，所以数据量很大的时候更推荐 Kafka

1. 修改 Filebeat 输出段的配置，将输出传送给 Kafka

```
output.kafka
  hosts: ["127.0.0.1:9092", "127.0.0.1:9093"]
  topic: system
```

2. 修改 Logstash input 相关配置，从 Kafka 读取数据

## 参考

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html

https://www.cnblogs.com/zsql/p/13137833.html

https://www.linuxe.cn/post-422.html

