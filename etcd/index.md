# etcd


## 介绍

### 简介

Etcd 是 CoreOS 基于 Raft 开发的分布式 key-value 存储，可用于服务发现、共享配置以及一致性保障（如数据库选主、分布式锁等）。

在分布式系统中，如何管理节点间的状态一直是一个难题，etcd 像是专门为集群环境的服务发现和注册而设计，它提供了数据 TTL 失效、数据改变监视、多值、目录监听、分布式锁原子操作等功能，可以方便的跟踪并管理集群节点的状态。

etcd 包含以下特点：

- 简单：基于 HTTP+JSON 的 API 让你用 curl 就可以轻松使用。
- 安全：支持 SSL 证书认证机制。
- 快速: 单实例每秒 1000 次写操作，2000+次读操作
- 可靠：采用 Raft 算法，实现分布式系统数据的可用性和一致性。
- 数据持久化：默认数据一更新就进行持久化。
- 监测变更：监测特定的键或目录以进行更改，并对值的更改做出反应

主要功能：

- 基本的 key-value 存储
- 监听机制
- key 的过期及续约机制，用于监控和服务发现
- 原子 Compare And Swap 和 Compare And Delete，用于分布式锁和 leader 选举

Raft 算法：工程上使用较为广泛的强一致性、去中心化、高可用的分布式协议。Raft 是一个共识算法(consensus algorithm)，所谓共识，就是多个节点对某个事情达成一致的看法，即使是在部分节点故障、网络延时、网络分割的情况下。

### 基础知识

每个 etcd cluster 都是有若干个 member 组成的，每个 member 是一个独立运行的 etcd 实例，单台机器上可以运行多个 member。

在正常运行的状态下，集群中会有一个 leader，其余的 member 都是 followers。leader 向 followers 同步日志，保证数据在各个 member 都有副本。leader 还会定时向所有的 member 发送心跳报文，如果在规定的时间里 follower 没有收到心跳，就会重新进行选举。

客户端所有的请求都会先发送给 leader，leader 向所有的 followers 同步日志，等收到超过半数的确认后就把该日志存储到磁盘，并返回响应客户端。

每个 etcd 服务有三大主要部分组成：raft 实现、WAL 日志存储、数据的存储和索引。WAL 会在本地磁盘（就是之前提到的 --data-dir）上存储日志内容（wal file）和快照（snapshot）。

### 核心组件

<img src="/images/k8s/etcd-architecture.png" alt="" width="90%" />

从 etcd 的架构图中我们可以看到，etcd 主要分为四个部分。

- HTTP Server：用于处理用户发送的 API 请求以及其它 etcd 节点的同步与心跳信息请求。
- Store：用于处理 etcd 支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，是 etcd 对用户提供的大多数 API 功能的具体实现。
- Raft：Raft 强一致性算法的具体实现，是 etcd 的核心。
- WAL：Write Ahead Log（预写式日志），是 etcd 的数据存储方式。除了在内存中存有所有数据的状态以及节点的索引以外，etcd 就通过 WAL 进行持久化存储。WAL 中，所有的数据提交前都会事先记录日志。Snapshot 是为了防止数据过多而进行的状态快照；Entry 表示存储的具体日志内容。

通常，一个用户的请求发送过来，会经由 HTTP Server 转发给 Store 进行具体的事务处理，如果涉及到节点的修改，则交给 Raft 模块进行状态的变更、日志的记录，然后再同步给别的 etcd 节点以确认数据提交，最后进行数据的提交，再次同步。

### 读写数据

为了保证数据的强一致性，etcd 集群中所有的数据流向都是一个方向，从 Leader （主节点）流向 Follower，也就是所有 Follower 的数据必须与 Leader 保持一致，如果不一致会被覆盖。

简单点说就是，用户可以对 etcd 集群中的所有节点进行读写，读取非常简单因为每个节点保存的数据是强一致的。对于写入来说，如果写入请求来自 Leader 节点即可直接写入然后 Leader 节点会把写入分发给所有 Follower；如果写入请求来自其他 Follower 节点，那么写入请求会给转发给 Leader 节点，由 Leader 节点写入之后再分发给集群上的所有其他节点。

### 存储机制

etcd v3 store 分为两部分，一部分是内存中的索引，kvindex，是基于 Google 开源的一个 Golang 的 btree 实现的，另外一部分是后端存储。按照它的设计，backend 可以对接多种存储，当前使用的 boltdb。boltdb 是一个单机的支持事务的 kv 存储，etcd 的事务是基于 boltdb 的事务实现的。etcd 在 boltdb 中存储的 key 是 reversion，value 是 etcd 自己的 key-value 组合，也就是说 etcd 会在 boltdb 中把每个版本都保存下，从而实现了多版本机制。

reversion 主要由两部分组成，第一部分 main rev，每次事务进行加一，第二部分 sub rev，同一个事务中的每次操作加一。

etcd 提供了命令和设置选项来控制 compact，同时支持 put 操作的参数来精确控制某个 key 的历史版本数。

内存 kvindex 保存的就是 key 和 reversion 之前的映射关系，用来加速查询。

<img src="/images/k8s/etcd-store.jpg" alt="" width="90%" />

### Watch 机制

etcd v3 的 watch 机制支持 watch 某个固定的 key，也支持 watch 一个范围（可以用于模拟目录的结构的 watch），所以 watchGroup 包含两种 watcher，一种是 key watchers，数据结构是每个 key 对应一组 watcher，另外一种是 range watchers, 数据结构是一个 IntervalTree，方便通过区间查找到对应的 watcher。

同时，每个 WatchableStore 包含两种 watcherGroup，一种是 synced，一种是 unsynced，前者表示该 group 的 watcher 数据都已经同步完毕，在等待新的变更，后者表示该 group 的 watcher 数据同步落后于当前最新变更，还在追赶。

当 etcd 收到客户端的 watch 请求，如果请求携带了 revision 参数，则比较请求的 revision 和 store 当前的 revision，如果大于当前 revision，则放入 synced 组中，否则放入 unsynced 组。同时 etcd 会启动一个后台的 goroutine 持续同步 unsynced 的 watcher，然后将其迁移到 synced 组。也就是这种机制下，etcd v3 支持从任意版本开始 watch，没有 v2 的 1000 条历史 event 表限制的问题（当然这是指没有 compact 的情况下）

### 容量管理

单个对象不建议超过 1.5M

默认容量 2G

不建议超过 8G

## 常用命令

```Shell
# 查看集群成员状态
etcdctl member list --write-out=table
# 写入数据
etcdctl --endpoints=localhost:12379 put /a b
# 读取数据
etcdctl --endpoints=localhost:12379 get /a
# 按key的前缀查询数据
etcdctl --endpoints=localhost:12379 get --prefix /
# 只显示键值
etcdctl --endpoints=localhost:12379 get --prefix / --keys-only --debug
# 查询历史版本值
etcdctl get x --rev=2
# 创建Snapshot
etcdctl --endpoints https://127.0.0.1:3379 --cert /tmp/etcd-certs/certs/127.0.0.1.pem --
key /tmp/etcd-certs/certs/127.0.0.1-key.pem --cacert /tmp/etcd-certs/certs/ca.pem
snapshot save snapshot.db
# 恢复数据
etcdctl snapshot restore snapshot.db \
--name infra2 \
--data-dir=/tmp/etcd/infra2 \
--initial-cluster
infra0=http://127.0.0.1:3380,infra1=http://127.0.0.1:4380,infra2=http://127.0.0.1:5380 \
--initial-cluster-token etcd-cluster-1 \
--initial-advertise-peer-urls http://127.0.0.1:5380
```

## 安装

### docker 方式安装

```Shell
# 启动
docker run -it -d -p 2379:2379 -p 2380:2380 --name etcd quay.io/coreos/etcd
# 查询
docker exec -it etcd etcdctl member list
```

### shell 脚本方式安装

编写以下脚本，并运行

```Code
ETCD_VER=v3.5.7

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
/tmp/etcd-download-test/etcdutl version
```

运行以下命令

```Shell
# start a local etcd server
/tmp/etcd-download-test/etcd

# write,read to etcd
/tmp/etcd-download-test/etcdctl --endpoints=localhost:2379 put foo bar
/tmp/etcd-download-test/etcdctl --endpoints=localhost:2379 get foo
```

### 二进制方式安装

1. 访问[GitHub releases page](https://github.com/etcd-io/etcd/releases/)上，根据自己的系统下载对应的软件包，下载完成后解压。
2. 进入到解压的目录，将 etcd 和 etcdctl 可执行文件移动到$GOPATH/bin 目录下。然后执行命令 etcd --version，同样会看到版本信息。

```Shell
$ etcd --version
etcd Version: 3.3.13
Git SHA: 98d3084
Go Version: go1.10.8
Go OS/Arch: linux/amd64
```

## 常用参数说明

--name：方便理解的节点名称，默认为 default，在集群中应该保持唯一，可以使用 hostname。

--data-dir：数据保存的目录，默认为${name}.etcd。

--snapshot-count：最大快照次数，指定有多少事务（transaction）被提交时，触发截取快照保存到磁盘，默认值 100000。

--max-snapshots: 最大保留快照数，默认 5 个。

--heartbeat-interval：心跳周期，leader 多久发送一次心跳到 followers。默认值是 100ms。

--eletion-timeout：重新投票的超时时间，如果 follow 在该时间间隔没有收到心跳包，会触发重新投票，默认为 1000ms。

--listen-peer-urls：和同伴通信的地址，比如 http://ip:2380，如果有多个，使用逗号分隔。需要所有节点都能够访问，所以不要使用 localhost！

--initial-advertise-peer-urls：该节点同伴监听地址，这个值会告诉集群中其他节点。

--listen-client-urls：对外提供服务的地址：比如 http://ip:2379,http://127.0.0.1:2379，客户端会连接到这里和etcd交互。

--advertise-client-urls：对外公告的该节点客户端监听地址，这个值会告诉集群中其他节点。

--initial-cluster：集群中所有节点的信息，格式为 node1=http://ip1:2380,node2=http://ip2:2380,…。注意：这里的 node1 是节点的 --name 指定的名字；后面的 ip1:2380 是 --initial-advertise-peer-urls 指定的值。

--initial-cluster-state：新建集群的时候，这个值为 new；假如已经存在的集群，这个值为 existing。

--initial-cluster-token：创建集群的 token，这个值每个集群保持唯一。这样的话，如果你要重新创建集群，即使配置和之前一样，也会再次生成新的集群和节点 uuid；否则会导致多个集群之间的冲突，造成未知的错误。

--log-level: 日志等级。info, warn, error, panic, or fatal。

所有以 --init 开头的配置都是在 bootstrap 集群的时候才会用到，后续节点的重启会被忽略。

## 参考

https://etcd.io/docs/v3.5/install/

<!-- https://www.cnblogs.com/lishen2021/p/14691019.html -->

https://github.com/cncamp/101/tree/master/module5/etcd-ha-demo
