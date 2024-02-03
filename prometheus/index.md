# Prometheus


## Prometheus 简介

[Prometheus](https://prometheus.io/) 最开始是由 SoundCloud 开发的开源监控告警系统，是 Google BorgMon 监控系统的开源版本。在 2016 年，Prometheus 加入 CNCF，成为继 Kubernetes 之后第二个被 CNCF 托管的项目。随着 Kubernetes 在容器编排领头羊地位的确立，Prometheus 也成为 Kubernetes 容器监控的标配。本文接下来将会对 Prometheus 做一个介绍。

### 特性

- 通过指标名称和标签(key/value 对）区分的多维度、时间序列数据模型
- 灵活的查询语法 PromQL
- 不需要依赖额外的存储，一个服务节点就可以工作
- 利用 http 协议，通过 pull 模式来收集时间序列数据
- 需要 push 模式的应用可以通过中间件 gateway 来实现
- 监控目标支持服务发现和静态配置
- 支持各种各样的图表和监控面板组件

### 核心组件

- Prometheus Server， 主要用于抓取数据和存储时序数据，另外还提供查询和 Alert Rule 配置管理。
- client libraries，用于对接 Prometheus Server, 可以查询和上报数据。
- push gateway ，用于批量，短期的监控数据的汇总节点，主要用于业务数据汇报等。
- 各种汇报数据的 exporters ，例如汇报机器数据的 node_exporter, 汇报 MongoDB 信息的 MongoDB exporter 等等。
- 用于告警通知管理的 alertmanager 。
- 各种支持工具。

### 架构

<img src="/images/monitor-alarm/prometheus-architecture.png" alt="" width="90%" />

从这个架构图，也可以看出 Prometheus 的主要模块包含， Server, Exporters, Pushgateway, PromQL, Alertmanager, WebUI 等。

- `Retrieval`：是负责定时去暴露的目标页面上去抓取采样指标数据。
- `Storage`：是负责将采样数据写入指定的时序数据库存储。
- `PromQL`：是 Prometheus 提供的查询语言模块。可以和一些 webui 比如 grfana 集成。
- `Jobs` / `Exporters`：Prometheus 可以从 Jobs 或 Exporters 中拉取监控数据。Exporter 以 Web API 的形式对外暴露数据采集接口。
- `Prometheus Server`：Prometheus 还可以从其他的 Prometheus Server 中拉取数据。
- `Pushgateway`：对于一些以临时性 Job 运行的组件，Prometheus 可能还没有来得及从中 pull 监控数据的情况下，这些 Job 已经结束了，Job 运行时可以在运行时将监控数据推送到 Pushgateway 中，Prometheus 从 Pushgateway 中拉取数据，防止监控数据丢失。
- `Service discovery`：是指 Prometheus 可以动态的发现一些服务，拉取数据进行监控，如从 DNS，Kubernetes，Consul 中发现, file_sd 是静态配置的文件。
- `AlertManager`：是一个独立于 Prometheus 的外部组件，用于监控系统的告警，通过配置文件可以配置一些告警规则，Prometheus 会把告警推送到 AlertManager。

### 工作流程

- Prometheus server 定期从配置好的 jobs 或者 exporters 中拉 metrics，或者接收来自 Pushgateway 发过来的 metrics，或者从其他的 Prometheus server 中拉 metrics。
- Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules，记录新的时间序列或者向 Alertmanager 推送警报。
- Alertmanager 根据配置文件，对接收到的警报进行处理，发出告警。
- 在图形界面中，可视化采集数据。

## 监控系统对比

### Prometheus vs Zabbix

- Zabbix 使用的是 C 和 PHP, Prometheus 使用 Golang, 整体而言 Prometheus 运行速度更快一点。
- Zabbix 属于传统主机监控，主要用于物理主机，交换机，网络等监控，Prometheus 不仅适用主机监控，还适用于 Cloud, SaaS, Openstack，Container 监控。
- Zabbix 在传统主机监控方面，有更丰富的插件。
- Zabbix 可以在 WebGui 中配置很多事情，但是 Prometheus 需要手动修改文件配置。

### 总结

- Prometheus 属于一站式监控告警平台，依赖少，功能齐全。
- Prometheus 支持对云或容器的监控，其他系统主要对主机监控。
- Prometheus 数据查询语句表现力更强大，内置更强大的统计函数。
- Prometheus 在数据存储扩展性以及持久性上没有 InfluxDB，OpenTSDB，Sensu 好。

## 安装

### 二进制包安装

下载请到 https://prometheus.io/download/

```Shell
cd /usr/local
wget https://github.com/prometheus/prometheus/releases/download/v1.6.2/prometheus-1.6.2.linux-amd64.tar.gz
tar -zxvf prometheus-1.6.2.linux-amd64.tar.gz
cd prometheus-1.6.2.linux-amd64
./prometheus --version
./prometheus --config.file=prometheus.yml
```

打开浏览器访问 http://localhost:9090/graph

### docker 安装

```Shell
docker run --name prometheus -d -p 9090:9090 quay.io/prometheus/prometheus
```

运行 `docker start prometheus` 启动服务

运行 `docker stats prometheus` 查看 prometheus 状态

运行 `docker stop prometheus` 停止服务

## 基本概念

### 数据模型

Prometheus 存储的是时序数据, 即按照相同时序(相同的名字和标签)，以时间维度存储连续的数据的集合。

格式上由 metric 的名字和一系列的标签（键值对）唯一标识而成。

不同的标签代表不同的时间序列。

### metric types（指标类型）

#### Counter

Counter 表示收集的数据是按照某个趋势（增加／减少）一直变化的，我们往往用它记录服务请求总量、错误总数等。

例如 Prometheus server 中 http_requests_total, 表示 Prometheus 处理的 http 请求总数，我们可以使用 delta, 很容易得到任意区间数据的增量，这个会在 PromQL 一节中细讲。

#### Gauge

Gauge 表示搜集的数据是一个瞬时的值，与时间没有关系，可以任意变高变低，往往可以用来记录内存使用率、磁盘使用率等。

例如 Prometheus server 中 go_goroutines, 表示 Prometheus 当前 goroutines 的数量。

#### Histogram

Histogram 由 <basename>\_bucket{le="<upper inclusive bound>"}，<basename>\_bucket{le="+Inf"}, <basename>\_sum，<basename>\_count 组成，主要用于表示一段时间范围内对数据进行采样（通常是请求持续时间或响应大小），并能够对其指定区间以及总数进行统计，通常它采集的数据展示为直方图。

例如 Prometheus server 中 prometheus_local_storage_series_chunks_persisted, 表示 Prometheus 中每个时序需要存储的 chunks 数量，我们可以用它计算待持久化的数据的分位数。

#### Summary

Summary 和 Histogram 类似，由 <basename>{quantile="<φ>"}，<basename>\_sum，<basename>\_count 组成，主要用于表示一段时间内数据采样结果（通常是请求持续时间或响应大小），它直接存储了 quantile 数据，而不是根据统计区间计算出来的。

例如 Prometheus server 中 prometheus_target_interval_length_seconds。

### 作业和实例

Prometheus 中，将任意一个独立的数据源（target）称之为实例（instance）。包含相同类型的实例的集合称之为作业（job）。

## Exporter

Exporter 是 prometheus 监控中重要的组成部分，负责数据指标的采集。广义上讲所有可以向 Prometheus 提供监控样本数据的程序都可以被称为一个 Exporter。而 Exporter 的一个实例称为 target。官方给出的插件有 blackbox_exporter、node_exporter、mysqld_exporter、snmp_exporter 等，第三方的插件有 redis_exporter，cadvisor 等。

### node_exporter

node_exporter 主要用来采集机器的性能指标数据，包括 cpu，内存，磁盘，io 等基本信息。

GitHub 地址：https://github.com/prometheus/node_exporter

官方教程：https://prometheus.io/docs/guides/node-exporter/

### mysqld_exporter

mysql_exporter 是用来收集 MysQL 或者 Mariadb 数据库相关指标的，mysql_exporter 需要连接到数据库并有相关权限。

GitHub 地址：https://github.com/prometheus/mysqld_exporter

### redis_exporter

redis_exporter 是用来收集 Redis 数据库相关指标的。

GitHub 地址：https://github.com/oliver006/redis_exporter

除了直接使用社区提供的 Exporter 程序以外，用户还可以基于 Prometheus 提供的 Client Library 创建自己的 Exporter 程序，目前 [Promthues 社区官方](https://github.com/prometheus)提供了对以下编程语言的支持：Go、Java/Scala、Python、Ruby。同时还有第三方实现的如：Bash、C++、Common Lisp、Erlang,、Haskeel、Lua、Node.js、PHP、Rust 等。

## 参考

https://prometheus.io/docs

https://songjiayang.gitbooks.io/prometheus

https://yunlzheng.gitbook.io/prometheus-book/
