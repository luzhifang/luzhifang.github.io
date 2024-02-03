# kafka 快速入门


## 架构

一个典型的 Kafka 体系架构包括若干 Producer、若干 Broker、若干 Consumer，以及一个 ZooKeeper 集群。其中 ZooKeeper 是 Kafka 用来负责集群元数据的管理、控制器的选举等操作的。Producer 将消息发送到 Broker，Broker 负责将收到的消息存储到磁盘中，而 Consumer 负责从 Broker 订阅并消费消息。

1. Producer：生产者，也就是发送消息的一方。生产者负责创建消息，然后将其投递到 Kafka 中。
2. Consumer：消费者，也就是接收消息的一方。消费者连接到 Kafka 上并接收消息，进而进行相应的业务逻辑处理。
3. Broker：服务代理节点。对于 Kafka 而言，Broker 可以简单地看作一个独立的 Kafka 服务节点或 Kafka 服务实例。大多数情况下也可以将 Broker 看作一台 Kafka 服务器，前提是这台服务器上只部署了一个 Kafka 实例。一个或多个 Broker 组成了一个 Kafka 集群。一般而言，我们更习惯使用首字母小写的 broker 来表示服务代理节点。

## 基本概念

1. 主题（Topic）：Kafka 中的消息以主题为单位进行归类，生产者负责将消息发送到特定的主题（发送到 Kafka 集群中的每一条消息都要指定一个主题），而消费者负责订阅主题并进行消费。一个主题可以横跨多个 broker，以此来提供比单个 broker 更强大的性能。
2. 分区（Partition）：一个主题可以细分为多个分区，一个分区只属于单个主题，很多时候也会把分区称为主题分区（Topic-Partition）。同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。
3. 偏移量（offset）：offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说，Kafka 保证的是分区有序而不是主题有序。
4. 副本（Replica）：Kafka 为分区引入了多副本（Replica）机制，通过增加副本数量可以提升容灾能力。副本之间是“一主多从”的关系，其中 leader 副本负责处理读写请求，follower 副本只负责与 leader 副本的消息同步。副本处于不同的 broker 中，当 leader 副本出现故障时，从 follower 副本中重新选举新的 leader 副本对外提供服务。
5. AR（Assigned Replicas）：所有副本。
6. ISR（In-Sync Replicas）：所有与 leader 副本保持一定程度同步的副本（包括 leader 副本在内）组成 ISR。只有在 ISR 集合中的副本才有资格被选举为新的 leader（不过这个原则也可以通过修改相应的参数配置来改变）。
7. OSR（Out-of-Sync Replicas）：与 leader 副本同步滞后过多的副本（不包括 leader 副本）组成 OSR。
8. HW：是 High Watermark 的缩写，俗称高水位，它标识了一个特定的消息偏移量（offset），消费者只能拉取到这个 offset 之前的消息。
9. LEO 是：Log End Offset 的缩写，它标识当前日志文件中下一条待写入消息的 offset。LEO 的大小相当于当前日志分区中最后一条消息的 offset 值加 1。分区 ISR 集合中的每个副本都会维护自身的 LEO，而 ISR 集合中最小的 LEO 即为分区的 HW。

## 前置条件

1. 需要先安装 Java 环境
2. 需要先安装 ZooKeeper 并启动

## 下载

1. 打开链接 https://kafka.apache.org/downloads 下载 Kafka 的 tgz 安装包

2. 然后解压并进入 kafka 路径下

```Shell
cd /usr/local
wget https://downloads.apache.org/kafka/3.1.2/kafka_2.12-3.1.2.tgz
tar -zxvf kafka_2.12-3.1.2.tgz
cd kafka_2.12-3.1.2
```

## 启动

配置文件放在主目录下 config/server.properties，主要需要注意的配置项如下

```Shell
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=1
# The address the socket server listens on. It will get the value returned from
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://127.0.0.1:9092
# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=127.0.0.1:2181/kafka
# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600
```

注意：确保集群中每个 broker 的 broker.id 配置参数的值不一样，以及 listeners 配置参数也需要修改为与 broker 对应的 IP 地址或域名，之后就可以各自启动服务。

修改好配置之后，启动命令如下(后台运行)

```Shell
bin/kafka-server-start.sh -daemon config/server.properties
```

## Topic 操作

kafka-topics.sh 脚本有 5 种指令类型：create、list、describe、alter 和 delete。

kafka-configs.sh 脚本是专门用来对配置进行操作的，这里的操作是指在运行状态下修改原有的配置，如此可以达到动态变更的目的。

kafka-reassign-partitions.sh 脚本来执行分区重分配的工作

1. 创建 Topic

```Shell
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

2. 查看 Topic 列表

```Shell
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

3. 查看 Topic 详细信息

```Shell
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test
```

4. 修改 Topic

```Shell
# 分区数只能增加不能减小
bin/kafka-topics.sh --alter --bootstrap-server localhost:9092 --partitions 2 --topic test
```

5. 删除 Topic

```Shell
bin/kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic test
```

## 消息发送

```Shell
# 输入命令后，会出现消息输入提示符，输入消息后按回车可发送消息
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

## 消息消费

```Shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

## 消息防丢

Consumer 使用拉（Pull）模式从服务端拉取消息，并且保存消费的具体位置，当消费者宕机后恢复上线时可以根据之前保存的消费位置重新拉取需要的消息进行消费，这样就不会造成消息丢失。

## 复制机制

Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制。事实上，同步复制要求所有能工作的 follower 副本都复制完，这条消息才会被确认为已成功提交，这种复制方式极大地影响了性能。而在异步复制方式下，follower 副本异步地从 leader 副本中复制数据，数据只要被 leader 副本写入就被认为已经成功提交。在这种情况下，如果 follower 副本都还没有复制完而落后于 leader 副本，突然 leader 副本宕机，则会造成数据丢失。Kafka 使用的这种 ISR 的方式则有效地权衡了数据可靠性和性能之间的关系。

## Kafka 为何如此之快

Kafka 实现了零拷贝原理来快速移动数据，避免了内核之间的切换。Kafka 可以将数据记录分批发送，从生产者到文件系统（Kafka 主题日志）到消费者，可以端到端的查看这些批次的数据。

批处理能够进行更有效的数据压缩并减少 I/O 延迟，Kafka 采取顺序写入磁盘的方式，避免了随机磁盘寻址的浪费。

总结一下其实就是四个要点

- 顺序读写
- 零拷贝
- 消息压缩
- 分批发送

## 参考

《深入理解 Kafka：核心设计与实践原理》 朱忠华

