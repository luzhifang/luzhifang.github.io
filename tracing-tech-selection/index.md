# 分布式链路追踪系列2-技术选型


分布式链路追踪系统其中最著名的是 Google Dapper 论文所介绍的 Dapper。源于 Google 为了解决可能由不同团队，不同语言，不同模块，部署在不同服务器，不同数据中心的所带来的软件复杂性（很难去分析，无法做定位），构建了一个的分布式跟踪系统。

Tracing 介于 Logging 和 Metric 之间， 以请求的维度来串联服务间的调用关系并记录调用耗时，即保留了必要的信息，又将分散的日志事件通过 Span 串联，帮助我们更好的理解系统的行为、辅助调试和排查性能问题。

## 分布式链路追踪系统设计目标

1. 低侵入性：代码侵入性
2. 灵活的应用策略：收集数据的范围和粒度
3. 时效性：从 agent 采样，到 collect、storage 和 display 尽可能快
4. 决策支持：策略是否可配置
5. 可视化
6. 低消耗：在 Web 请求链路中，对请求的响应影响尽可能小
7. 延展性：随着业务量的增长，分布式追踪系统依然具有高可用和高性能表现

## 对比

| 名称       | 厂商        | 开发语言 | OpenTracing 兼容 | 侵入性 | 时效性 | 可视化 | 消耗             |
| ---------- | ----------- | -------- | ---------------- | ------ | ------ | ------ | ---------------- |
| Jaeger     | Uber        | Go       | 是               | 中     | 高     | 中     | 低               |
| Zipkin     | twitter     | Java     | 是               | 高     | 高     | 中     | 低               |
| Pinpoint   | NAVER       | Java     | 否               | 低     | -      | 高     | 低               |
| CAT        | 大众点评    | Java     | 否               | 高     | 中     | 高     | 低               |
| Appdash    | sourcegraph | Go       | 是               | 低     | 高     | 低     | 不支持大规模部署 |
| SkyWalking | 华为        | Java     | 是               | 低     | 中     | 高     | 低               |

## 小结

一般选择的时候会选择语言亲和性比较强的追踪系统，比如 Go 选择 jaeger，Java 选择 Zipkin 和 Skywalking。

jaeger 兼容 Zipkin，在这点上 Jaeger 完胜。

## 参考

https://jckling.github.io/2021/04/02/Jaeger/%E5%85%A8%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%E4%B8%8E%20Jaeger%20%E5%85%A5%E9%97%A8/

