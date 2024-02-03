# 消息队列选型


| 维度               | Kafka                                        | RabbitMQ                           | ZeroMQ                                     | RocketMQ                                                   | ActiveMQ                           |
| ------------------ | -------------------------------------------- | ---------------------------------- | ------------------------------------------ | ---------------------------------------------------------- | ---------------------------------- |
| 资料文档           | 中等                                         | 多                                 | 少                                         | 少                                                         | 多                                 |
| 开发语言           | Scala                                        | Erlang                             | C                                          | Java                                                       | Java                               |
| 支持的协议         | 自定义基于 TCP                               | AMQP                               | TCP、UDP                                   | 自定义                                                     | OpenWire、STOMP、REST、XMPP、AMQP  |
| 消息存储           | 内存、磁盘、数据库。支持大量堆积。           | 内存、磁盘。支持少量堆积。         | 消息发送端的内存或者磁盘中。不支持持久化。 | 磁盘。支持大量堆积。                                       | 内存、磁盘、数据库。支持少量堆积。 |
| 消息事务           | 支持                                         | 支持                               | 不支持                                     | 支持                                                       | 支持                               |
| 负载均衡           | 支持                                         | 支持不好                           | 不支持                                     | 支持                                                       | 支持（基于 zookeeper）             |
| 集群方式           | 支持                                         | 支持简单集群                       | 不支持                                     | 支持                                                       | 支持                               |
| 管理界面           | 一般                                         | 好                                 | 无                                         | 有，非自带                                                 | 一般                               |
| 可用性             | 非常高（分布式）                             | 高（主从）                         | 高                                         | 非常高（分布式）                                           | 高（主从）                         |
| 消息重复           | 支持 at least once、at most once             | 支持 at least once、at most once   | 有重传，没有持久化                         | 支持 at least once                                         | 支持 at least once                 |
| 吞吐量 TPS         | 极大                                         | 比较大                             | 极大                                       | 大                                                         | 比较大                             |
| 消费推拉模式       | 拉                                           | 推                                 | 推                                         | 拉                                                         | 推                                 |
| 订阅形式和消息分发 | 基于 topic 及按 topic 正则匹配的发布订阅模式 | direct、topic、Headers 和 fanout。 | 点对点(p2p)                                | 基于 topic/messageTag 及按消息类型、属性正则匹配的发布订阅 | 点对点(p2p)、广播（发布-订阅）     |
| 顺序消息           | 支持                                         | 不支持                             | 不支持                                     | 支持                                                       | 不支持                             |
| 消息确认           | 支持                                         | 支持                               | 支持                                       | 支持                                                       | 支持                               |
| 消息回溯           | 支持指定分区 offset 位置的回溯               | 不支持                             | 不支持                                     | 支持指定时间点的回溯                                       | 不支持                             |
| 消息重试           | 不支持，但是可以实现                         | 不支持，可以利用消息确认机制实现   | 不支持                                     | 支持                                                       | 不支持                             |
| 并发度             | 高                                           | 极高                               | 高                                         | 高                                                         | 高                                 |
| 延迟队列           | 不支持                                       | 支持                               | 不支持                                     | 支持                                                       | 不支持                             |
| 死信队列           | 不支持                                       | 支持                               | 不支持                                     | 支持                                                       | 不支持                             |
| 优先级队列         | 不支持                                       | 支持                               | 不支持                                     | 不支持                                                     | 不支持                             |

## 参考

https://cloud.tencent.com/developer/article/1449951

https://cloud.tencent.com/developer/article/1944357
