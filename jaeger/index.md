# 分布式链路追踪系列3-jaeger快速入门


## 简介

受 Dapper 和 OpenZipkin 启发的 Jaeger 是由 Uber 开源的分布式跟踪系统，兼容 OpenTracing 标准。它用于监视和诊断基于微服务的分布式系统，功能包括：

- 分布式上下文传播
- 分布式交易监控
- 根本原因分析
- 服务依赖性分析
- 性能/延迟优化

## 特性

- 兼容 OpenTracing 的数据模型和工具库
- 多语言支持
- 对每个服务/端点概率使用一致的前期采样
- 多种存储后端支持：Cassandra，Elasticsearch，内存。
- 系统拓扑图
- 自适应采样
- 收集后数据处理管道

## 架构

jaeger 架构如下：

1. Collector 直接写入存储

<img src="/images/tracing/jaeger-architecture.png" alt="" width="90%" />

2. Collecter 写入 Kafka 作为初始缓冲区

<img src="/images/tracing/jaeger-architecture-kafka.png" alt="" width="90%" />

Jaeger 主要包括以下这些组件：（每一个组件都支持单独部署）

- `jaeger-client`：Jaeger 的客户端，实现了 OpenTracing 的 API，支持主流编程语言。客户端直接集成在目标 Application 中，其作用是记录和发送 Span 到 Jaeger Agent。在 Application 中调用 Jaeger Client Library 记录 Span 的过程通常被称为埋点。
- `jaeger-agent`：暂存 Jaeger Client 发来的 Span，并批量向 Jaeger Collector 发送 Span，一般每台机器上都会部署一个 Jaeger Agent。官方的介绍中还强调了 Jaeger Agent 可以将服务发现的功能从 Client 中抽离出来，不过从架构角度讲，如果是部署在 Kubernetes 或者是 Nomad 中，Jaeger Agent 存在的意义并不大。
- `jaeger-collector`：接受 Jaeger Agent 发来的数据，并将其写入存储后端，目前支持采用 Cassandra 和 Elasticsearch 作为存储后端。比较推荐用 Elasticsearch，既可以和日志服务共用同一个 ES，又可以使用 Kibana 对 Trace 数据进行额外的分析。架构图中的存储后端是 Cassandra，旁边还有一个 Spark，讲的就是可以用 Spark 等其他工具对存储后端中的 Span 进行直接分析。
- `jaeger-query` & `jaeger-ui`：读取存储后端中的数据，以直观的形式呈现。
- `ingester`：从 Kafka Topic 中读取数据并写入到另一个存储后端（Cassandra, Elasticsearch）。

## 快速部署本地环境

### All in one

多合一是用于快速本地测试的可执行文件，具有内存存储组件，可启动 Jaeger UI，收集器，查询和代理。开始多合一的最简单方法是使用发布到 DockerHub 的预构建映像（单个命令行）。

```Shell
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.14
```

容器需要暴露的端口

| 端口  | 协议 | 所属模块  | 功能                                                                                         |
| ----- | ---- | --------- | -------------------------------------------------------------------------------------------- |
| 6831  | UDP  | agent     | accept jaeger.thrift over Thrift-compact protocol (used by most SDKs)                        |
| 6832  | UDP  | agent     | accept jaeger.thrift over Thrift-binary protocol (used by Node.js SDK)                       |
| 5775  | UDP  | agent     | (deprecated) accept zipkin.thrift over compact Thrift protocol (used by legacy clients only) |
| 5778  | HTTP | agent     | serve configs (sampling, etc.)                                                               |
| 16686 | HTTP | query     | serve frontend                                                                               |
| 14268 | HTTP | collector | accept jaeger.thrift directly from clients                                                   |
| 14250 | HTTP | collector | accept model.proto                                                                           |
| 9411  | HTTP | collector | Zipkin compatible endpoint (optional)                                                        |

### 手动安装

根据不同系统，下载[安装包](https://www.jaegertracing.io/download/)

运行 `jaeger-all-in-one[.exe]`

```Shell
jaeger-all-in-one --collector.zipkin.http-port=9411
```

安装好之后可以通过 http://localhost:16686 访问 Jaeger UI。

## 采样

Jaeger 库实现了一致的前期（或基于头）的采样。例如，假设我们有一个简单的调用图，其中服务 A 调用服务 B，服务 B 调用服务 C：A-> B->C。当服务 A 收到不包含跟踪信息的请求时，Jaeger 跟踪器将开始新的跟踪，为其分配一个随机跟踪 ID，然后根据当前安装的采样策略做出采样决定。采样决策将与请求一起传播到 B 和 C，因此这些服务将不会再次做出采样决策，而是会尊重顶级服务 A 做出的决策。这种方法保证了，如果对跟踪进行了采样，则所有其跨度将记录在后端。如果每个服务都做出自己的抽样决定，那么我们很少会在后端获得完整的跟踪。

支持设置采样率是 Jaeger 的一个亮点，在生产环境中，如果对每个请求都开启 Trace，必然会对系统性能带来一定压力，除此之外，数量庞大的 Span 也会占用大量的存储空间。为了尽量消除分布式追踪采样对系统带来的影响，设置采样率是一个很好的办法。

### 客户端采样配置

当使用配置对象来实例化 tracer 时，可以通过 sampler.type 和 sampler.param 属性选择采样类型。Jaeger 支持下面四种采样策略：

- Constant（sampler.type=const）：const 意为常量，采样器始终对所有 traces 做出相同的决定。sample.param=1 则采样所有 tracer，sample.param=0 则都不采样。
- **Probabilistic (sampler.type=probabilistic)**：概率采样，采样概率介于 0-1 之间，通过 sample.param 属性进行配置，例如,在 sampler.param=0.1 的情况下，将在 10 条 traces 中大约采样 1 条。
- **Rate Limiting (sampler.type=ratelimiting)**：设置每秒的采样次数上限。当 sampler.param=2 的时候，将以每秒 2 条 traces 的速率对请求进行采样。
- Remote (sampler.type=remote）：默认配置，client 将从 jaeger-agent 中获取当前服务使用的采样策略，这允许 Client 从 Jaeger Agent 中动态获取采样率设置。

### 自适应采样器

自适应采样器是一个组合了两个功能的复合采样器：

- 它基于每个操作（即基于 span 操作名称）做出抽样决策。这在 API 服务中特别有用，这些 API 服务的端点的流量可能非常不同，并且对整个服务使用单个概率采样器可能会使某些低 QPS 端点饿死（从不采样）。
- 它支持最低的保证采样率，例如始终允许每秒最多 N 条 traces，然后以一定的概率采样所有高于此值的采样率（一切都是针对每个操作，而不是针对每个服务）。

### Jaeger 采样配置示例

收集器可以通过 --sampling.strategies-file 选项通过静态采样策略实例化(如果使用 Remote sample r 配置, 则将传播到相应的服务)。该选项需要一个已定义采样策略的 json 文件的路径。

如果未提供任何配置，则收集器将为所有服务返回默认概率抽样策略，概率为 0.001（0.1％）

```Code
{
  "service_strategies": [
    {
      "service": "foo",
      "type": "probabilistic",
      "param": 0.8,
      "operation_strategies": [
        {
          "operation": "op1",
          "type": "probabilistic",
          "param": 0.2
        },
        {
          "operation": "op2",
          "type": "probabilistic",
          "param": 0.4
        }
      ]
    },
    {
      "service": "bar",
      "type": "ratelimiting",
      "param": 5
    }
  ],
  "default_strategy": {
    "type": "probabilistic",
    "param": 0.5,
    "operation_strategies": [
      {
        "operation": "/health",
        "type": "probabilistic",
        "param": 0.0
      },
      {
        "operation": "/metrics",
        "type": "probabilistic",
        "param": 0.0
      }
    ]
  }
}
```

service_strategies 元素定义特定于服务的采样策略，而 operation_strategies 元素定义特定于操作的采样策略。可能有两种策略：概率策略和速率限制，如上所述（注意：operation_strategies 不支持速率限制）。如果服务不是 service_strategies 定义在内的操作，则采用 default_strategy 定义的采样策略。

## Go 简单示例

```Go
package main

import (
	"github.com/opentracing/opentracing-go"
	"github.com/uber/jaeger-client-go"
	"github.com/uber/jaeger-client-go/config"
	"io"
	"time"
)

func main() {
	tracer, closer, err := NewTracer("demo-service", "127.0.0.1:6831")
	if err != nil {
		panic(err)
	}
	defer closer.Close()

	// 创建第一个 span A
	ASpan := tracer.StartSpan("A")
	// 可手动调用 Finish()
	defer ASpan.Finish()

	CallBService(tracer, ASpan)
}

// CallBService 调用B方法，可以假设是另外一个服务
func CallBService(tracer opentracing.Tracer, parentSpan opentracing.Span) {
	// 继承上下文关系，创建子 span
	BSpan := tracer.StartSpan("B", opentracing.ChildOf(parentSpan.Context()))
	BSpan.Finish()
}

// NewTracer 新建tracer
func NewTracer(serviceName, addr string) (opentracing.Tracer, io.Closer, error) {
	cgf := config.Configuration{
		// 服务名称
		ServiceName: serviceName,
		// 采样配置
		Sampler: &config.SamplerConfig{
			Type:  jaeger.SamplerTypeConst,
			Param: 1,
		},
		// 上报配置
		Reporter: &config.ReporterConfig{
			BufferFlushInterval: 1 * time.Second,
			LogSpans:            true,
			LocalAgentHostPort:  addr,
			// 直接上报到collector
			// CollectorEndpoint: "http://127.0.0.1:14268/api/traces",
		},
	}
	return cgf.NewTracer()
}
```

## Go HTTP 示例

1. 客户端

```Go
package main

import (
	"fmt"
	"github.com/opentracing/opentracing-go"
	"github.com/opentracing/opentracing-go/ext"
	"github.com/opentracing/opentracing-go/log"
	"github.com/uber/jaeger-client-go"
	"github.com/uber/jaeger-client-go/config"
	"io"
	"net/http"
	"time"
)

func main() {
	tracer, closer, err := NewClientTracer("demo-service", "127.0.0.1:6831")
	if err != nil {
		panic(err)
	}
	defer closer.Close()

	// 创建第一个 span A
	ASpan := tracer.StartSpan("A")
	// 可手动调用 Finish()
	defer ASpan.Finish()

	CallUserInfo(tracer, ASpan)
}

// CallUserInfo 请求远程服务，获得用户信息
func CallUserInfo(tracer opentracing.Tracer, parentSpan opentracing.Span) {
	// 继承上下文关系，创建子 span
	childSpan := tracer.StartSpan(
		"B",
		opentracing.ChildOf(parentSpan.Context()),
	)

	url := "http://127.0.0.1:8081/Get?username=jeff"
	req, _ := http.NewRequest("GET", url, nil)
	// 设置 tag
	ext.SpanKindRPCClient.Set(childSpan)
	ext.HTTPUrl.Set(childSpan, url)
	ext.HTTPMethod.Set(childSpan, "GET")
	err := tracer.Inject(childSpan.Context(), opentracing.HTTPHeaders, opentracing.HTTPHeadersCarrier(req.Header))
	if err != nil {
		log.Error(err)
	}

	resp, _ := http.DefaultClient.Do(req)
	log.Event(fmt.Sprintf("%v", resp))
	defer childSpan.Finish()
}

// NewClientTracer 新建tracer
func NewClientTracer(serviceName, addr string) (opentracing.Tracer, io.Closer, error) {
	cgf := config.Configuration{
		// 服务名称
		ServiceName: serviceName,
		// 采样配置
		Sampler: &config.SamplerConfig{
			Type:  jaeger.SamplerTypeConst,
			Param: 1,
		},
		// 上报配置
		Reporter: &config.ReporterConfig{
			BufferFlushInterval: 1 * time.Second,
			LogSpans:            true,
			LocalAgentHostPort:  addr,
			// 直接上报到collector
			// CollectorEndpoint: "http://127.0.0.1:14268/api/traces",
		},
	}
	return cgf.NewTracer()
}
```

2. 服务端

```Go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/opentracing/opentracing-go"
	"github.com/opentracing/opentracing-go/ext"
	"github.com/opentracing/opentracing-go/log"
	"github.com/uber/jaeger-client-go"
	"github.com/uber/jaeger-client-go/config"
	"io"
	"net/http"
	"time"
)

func main() {
	r := gin.Default()
	// 插入中间件处理
	r.Use(UseOpenTracing())
	r.GET("/Get", GetUserInfo)
	err := r.Run("0.0.0.0:8081") // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
	if err != nil {
		panic(err)
	}
}

func GetUserInfo(ctx *gin.Context) {
	userName := ctx.Param("username")
	log.Event("收到请求，用户名称为:" + userName)
	ctx.String(http.StatusOK, "他的博客是 https://luzhifang.github.io/")
}

func UseOpenTracing() gin.HandlerFunc {
	handler := func(c *gin.Context) {
		tracer, closer, _ := NewServerTracer("userInfoWebService", "127.0.0.1:6831")
		spanContext, err := tracer.Extract(opentracing.HTTPHeaders, opentracing.HTTPHeadersCarrier(c.Request.Header))
		if err != nil {
			log.Error(err)
		}
		defer closer.Close()

		// 生成依赖关系，并新建一个 span、
		// 这里很重要，因为生成了  References []SpanReference 依赖关系
		startSpan := tracer.StartSpan(c.Request.URL.Path, ext.RPCServerOption(spanContext))
		defer startSpan.Finish()

		// 记录 tag
		// 记录请求 Url
		ext.HTTPUrl.Set(startSpan, c.Request.URL.Path)
		// Http Method
		ext.HTTPMethod.Set(startSpan, c.Request.Method)
		// 记录组件名称
		ext.Component.Set(startSpan, "Gin-Http")

		// 在 header 中加上当前进程的上下文信息
		c.Request = c.Request.WithContext(opentracing.ContextWithSpan(c.Request.Context(), startSpan))
		// 传递给下一个中间件
		c.Next()
		// 继续设置 tag
		ext.HTTPStatusCode.Set(startSpan, uint16(c.Writer.Status()))
	}

	return handler
}

// NewServerTracer 新建tracer
func NewServerTracer(serviceName, addr string) (opentracing.Tracer, io.Closer, error) {
	cgf := config.Configuration{
		// 服务名称
		ServiceName: serviceName,
		// 采样配置
		Sampler: &config.SamplerConfig{
			Type:  jaeger.SamplerTypeConst,
			Param: 1,
		},
		// 上报配置
		Reporter: &config.ReporterConfig{
			BufferFlushInterval: 1 * time.Second,
			LogSpans:            true,
			LocalAgentHostPort:  addr,
			// 直接上报到collector
			// CollectorEndpoint: "http://127.0.0.1:14268/api/traces",
		},
	}
	return cgf.NewTracer()
}
```

## 参考

https://juejin.cn/post/6844903971010641934

https://xiaoming.net.cn/2021/03/25/Opentracing%E6%A0%87%E5%87%86%E5%92%8CJaeger%E5%AE%9E%E7%8E%B0

