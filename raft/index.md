# Raft 协议


## 简介

Raft 协议是一种分布式一致性算法（共识算法），共识就是多个节点对某一个事件达成一致的算法，即使出现部分节点故障，网络延时等情况，也不影响各节点，进而提高系统的整体可用性。Raft 是使用较为广泛的分布式协议，我们熟悉的 etcd 注册中心就采用了这个算法；
Raft 算法将分布式一致性分解为多个子问题，包括 Leader 选举（Leader election）、日志复制（Log replication）、安全性（Safety）、日志压缩（Log compaction）等。raft 将系统中的角色分为领导者（Leader）、跟从者（Follower）和候选者（Candidate）。

- Leader：接受客户端请求，并向 Follower 同步请求日志，当日志同步到大多数节点上后高速 Follower 提交日志。
- Follower：接受并持久化 Leader 同步的日志，在 Leader 告知日志可以提交后，提交日志。当 Leader 出现故障时，主动推荐自己为候选人。
- Candidate：Leader 选举过程中的临时角色。向其他节点发送请求投票信息，如果获得大多数选票，则晋升为 Leader。

<img src="/images/distributed-system/raft-write-process.jpg" alt="" width="90%" />

Raft 要求系统在任意时刻最多只有一个 Leader，正常工作期间只有 Leader 和 Follower，Raft 算法将时间划分为任意不同长度的任期(Term),每一任期的开始都是一次选举，一个或多个候选人会试图称为 Leader，在成功选举 Leader 后，Leader 会在整个任期内管理整个集群，如果 Leader 选举失败，该任期就会因为没有 Leader 而结束，开始下一任期，并立刻开始下一次选举。

## Leader 选举

1. Raft 使用心跳机制来触发领导者选举，当服务器启动时，**初始化都是 Follower 身份**，
2. 由于没有 Leader，Followers 无法与 Leader 保持心跳，因此，Followers 会认为 Leader 已经下线，进而转为 Candidate 状态，
3. 然后 Candidate 向集群其他节点请求投票，同意自己成为 Leader，如果 Candidate 收到超过半数节点的投票(N/2 +1)，它将获胜成为 Leader。

Leader 向所有 Follower 周期性发送 heartbeat，如果 Follower 在选举超时时间（150 到 300 毫秒随机值）内没有收到 Leader 的 heartbeat，就会等待一段随机的时间后发起一次 Leader 选举。

## 日志同步

Raft 算法实现日志同步的具体过程如下：

1. Leader 收到来自客户端的请求，将之封装成 log entry 并追加到自己的日志中；
2. Leader 并行地向系统中所有节点发送日志复制消息；
3. 接收到消息的节点确认消息没有问题，则将 log entry 追加到自己的日志中，并向 Leader 返回 ACK 表示接收成功；
4. Leader 若在随机超时时间内收到大多数节点的 ACK(N/2 +1),则将该 log entry 应用到状态机并向客户端返回成功。

## 安全性

### 选举安全性

选举安全性，即任一任期内最多一个 leader 被选出。这一点非常重要，在一个复制集中任何时刻只能有一个 leader。系统中同时有多余一个 leader，被称之为脑裂（brain split），这是非常严重的问题，会导致数据的覆盖丢失。在 raft 中，两点保证了这个属性：

- 一个节点某一任期内最多只能投一票；
- 只有获得 majority 投票的节点才会成为 leader。

因此，某一任期内一定只有一个 leader。

### log matching

很有意思，log 匹配特性， 就是说如果两个节点上的某个 log entry 的 log index 相同且 term 相同，那么在该 index 之前的所有 log entry 应该都是相同的。

- leader 在某一 term 的任一位置只会创建一个 log entry，且 log entry 是 append-only。
- consistency check。leader 在 AppendEntries 中包含最新 log entry 之前的一个 log 的 term 和 index，如果 follower 在对应的 term index 找不到日志，那么就会告知 leader 不一致。

### Leader 完整性（Leader Completeness）

- 指 Leader 日志的完整性，当 Log 在任期 Term1 被 Commit 后，那么以后任期 Term2、Term3…等的 Leader 必须包含该 Log；
- Raft 在选举阶段就使用 Term 的判断用于保证完整性：当请求投票的该 Candidate 的 Term 较大或 Term 相同 Index 更大则投票，否则拒绝该请求。

## 如何解决脑裂问题

当 raft 在集群中遇见网络分区的时候,集群就会因此而相隔开，在不同的网络分区里会因为无法接收到原来的 leader 发出的心跳而超时选主，这样就会造成多 leader 现象，在网络分区 1 和网络分区 2 中，出现了两个 leaderA 和 D，假设此时要更新分区 2 的值，因为分区 2 无法得到集群中的大多数节点的 ACK，会复制失败。而网络分区 1 会成功，因为分区 1 中的节点更多，leaderA 能得到大多数回应。

<img src="/images/distributed-system/raft-network-partition.jpg" alt="" width="90%" />

当网络恢复的时候,集群不再是双分区,raft 会有如下操作：

1. leaderD 发现自己的 Term 小于 LeaderA,会自动下台(step down)成为 follower,leaderA 保持不变依旧是集群中的主 leader 角色。
2. 分区中的所有节点会回滚 roll back 自己的数据日志,并匹配新 leader 的 log 日志，然后实现同步提交更新自身的值。
3. 最终集群达到整体一致，集群存在唯一 leader（节点 A）。

## 参考

https://thesecretlivesofdata.com/raft/

