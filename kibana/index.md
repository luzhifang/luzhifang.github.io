# Kibana


## 简介

Kibana 可以为 Elasticsearch 提供一个可视化平台，将收集到的数据进行图形化展示。Kibana 7 开始默认支持中文，配图更为方便，推荐使用

## 安装

1. 二进制安装

```Shell
cd /usr/local
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.6-linux-x86_64.tar.gz
tar zxvf kibana-7.17.6-linux-x86_64.tar.gz
cd kibana-7.17.6
./bin/kibana
```

2. docker 方式安装

```Shell
docker pull docker.elastic.co/kibana/kibana:7.17.6
docker run -p 5601:5601 docker.elastic.co/kibana/kibana:7.17.6
```

## 配置

Kibana 配置只需要修改主配置文件 config/kibana.yml 里的以下信息即可

```
server.port: 5601 #kibana端口
server.host: "127.0.0.1" #kibana地址
server.maxPayload: 10485760
elasticsearch.hosts: ["http://172.20.1.221:9200"]
kibana.index: ".kibana" #kibana的元数据也会存放在ES的索引中，这里配置的就是索引的名字，一般不用修改
i18n.locale: "zh-CN" #修改为中文
```

## 使用

### 登录

首次登录要用上 elasticsearch 用户 kibana_system 和 elastic 的用户名密码

```Shell
# 交互设置所有密码
./bin/elasticsearch-setup-passwords interactive
# 重置 elastic 密码
./bin/elasticsearch-reset-password -u elastic
# 重置 kibana_system 密码
./bin/elasticsearch-reset-password -u kibana_system
```

### Discover

从发现页可以交互地探索 ES 的数据。可以访问与所选索引模式相匹配的每一个索引中的每一个文档。您可以提交搜索查询、筛选搜索结果和查看文档数据。还可以看到匹配搜索查询和获取字段值统计的文档的数量。如果一个时间字段被配置为所选择的索引模式，则文档的分布随着时间的推移显示在页面顶部的直方图中。

### Visualize

可视化能使你创造你的 Elasticsearch 指标数据的可视化。然后你可以建立仪表板显示相关的可视化。Kibana 的可视化是基于 Elasticsearch 查询。通过一系列的 Elasticsearch 聚合提取和处理您的数据，您可以创建图表显示你需要知道的关于趋势，峰值和骤降。您可以从搜索保存的搜索中创建可视化或从一个新的搜索查询开始。

### Dashboard

一个仪表板显示 Kibana 保存的一系列可视化。你可以根据需要安排和调整可视化，并保存仪表盘，可以被加载和共享。

### Monitoring

默认 Kibana 是没有该选项的。其实，Monitoring 是由 X-Pack 集成提供的。

该 X-pack 监控组件使您可以通过 Kibana 轻松地监控 ElasticSearch。您可以实时查看集群的健康和性能，以及分析过去的集群、索引和节点度量。此外，您可以监视 Kibana 本身性能。当你安装 X-pack 在群集上，监控代理运行在每个节点上收集和指数指标从 Elasticsearch。安装在 X-pack 在 Kibana 上，您可以查看通过一套专门的仪表板监控数据。

### Timelion

Timelion 是一个时间序列数据的可视化，可以结合在一个单一的可视化完全独立的数据源。它是由一个简单的表达式语言驱动的，你用来检索时间序列数据，进行计算，找出复杂的问题的答案，并可视化的结果。

这个功能由一系列的功能函数组成，同样的查询的结果，也可以通过 Dashboard 显示查看。

### Management

管理中的应用是在你执行你的运行时配置 kibana，包括初始设置和指标进行配置模式，高级设置，调整自己的行为和 Kibana，各种“对象”，你可以查看保存在整个 Kibana 的内容如发现页，可视化和仪表板。

### Dev Tools

使用户方便的通过浏览器直接与 Elasticsearch 进行交互。从 Kibana 5 开始改名并直接内建在 Kibana，就是 Dev Tools 选项。

## 参考

https://www.elastic.co/guide/cn/kibana/current/getting-started.html

https://www.linuxe.cn/post-309.html

