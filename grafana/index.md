# Grafana


## Grafana 介绍

Grafana 是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。它主要有以下六大特点：

1. 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式；
2. 数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch 和 KairosDB 等；
3. 通知提醒：以可视方式定义最重要指标的警报规则，Grafana 将不断计算并发送通知，在数据达到阈值时通过 Slack、PagerDuty 等获得通知；
4. 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源；
5. 注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记；
6. 过滤器：Ad-hoc 过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。

## 安装

### 下载

官网下载地址：[Grafana](https://grafana.com/grafana/download)

可以根据需要选择二进制安装和 docker 安装。

### 启动

启动服务，打开浏览器，输入 IP+端口，3000 为 Grafana 的默认侦听端口。

系统默认用户名和密码为 admin/admin，第一次登陆系统会要求修改密码，修改密码后登陆。

## 使用

### 添加数据源

依次点击设置图标 -> Data sources -> Add data source -> Prometheus

### 添加面板

依次点击 Add panel -> Add a new panel

## 工作原理

1. exporter 负责收集数据，本实例中是 node-exporter 这个服务，会查询你的本地电脑的信息，比如内存还有多少、CPU 负载之类。
2. Prometheus 定时从 exporter 里拉取存储的统计数据。
3. Grafana 每次要展现一个仪表盘的时候，会向 Prometheus 发送一个查询请求。

## docker-compose 示例

完整示例请看[Demo](https://github.com/luzhifang/prometheus-grafana-demo)

## 参考

https://cloud.tencent.com/developer/article/1807679

