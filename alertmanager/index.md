# Prometheus Alertmanager


## Alertmanager 简介

告警能力在 Prometheus 的架构中被划分成两个独立的部分。如下所示，通过在 Prometheus 中定义 AlertRule（告警规则），Prometheus 会周期性的对告警规则进行计算，如果满足告警触发条件就会向 Alertmanager 发送告警信息。

<img src="/images/monitor-alarm/prometheus-alert-architecture.png" alt="" width="90%" />

在 Prometheus 中一条告警规则主要由以下几部分组成：

- 告警名称：用户需要为告警规则命名，当然对于命名而言，需要能够直接表达出该告警的主要内容
- 告警规则：告警规则实际上主要由 PromQL 进行定义，其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警

在 Prometheus 中，还可以通过 Group（告警组）对一组相关的告警进行统一定义。当然这些定义都是通过 YAML 文件来统一管理的。

Alertmanager 作为一个独立的组件，**负责接收并处理来自 Prometheus Server(也可以是其它的客户端程序)的告警信息**。Alertmanager 可以对这些告警信息进行进一步的处理，比如当接收到大量重复告警时能够消除重复的告警信息，同时对告警信息进行分组并且路由到正确的通知方，Prometheus 内置了对邮件，Slack 等多种通知方式的支持，同时还支持与 Webhook 的集成，以支持更多定制化的场景。例如，**企业微信和钉钉**。

### 特性

Alertmanager 除了提供基本的告警通知能力以外，还主要提供了如：分组、抑制以及静默等告警特性：

<img src="/images/monitor-alarm/alertmanager-features.png" alt="" width="100%" />

## 自定义 Prometheus 告警规则

Prometheus 中的告警规则允许你基于 PromQL 表达式定义告警触发条件，Prometheus 后端对这些触发规则进行周期性计算，当满足触发条件后则会触发告警通知。默认情况下，用户可以通过 Prometheus 的 Web 界面查看这些告警规则以及告警的触发状态。当 Promthues 与 Alertmanager 关联之后，可以将告警发送到外部服务如 Alertmanager 中并通过 Alertmanager 可以对这些告警进行进一步的处理。

修改 prometheus.yml，添加

```
rule_files:
  - rules/*.yml
```

创建告警文件 rules/hoststats-alert.yml 内容如下：

```
groups:
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu_seconds_total{mode!='idle'}[5m]))) by (instance) > 0.85
    for: 1m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} CPU usgae high"
      description: "{{ $labels.instance }} CPU usage above 85% (current value: {{ $value }})"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal - node_memory_MemAvailable)/node_memory_MemTotal > 0.85
    for: 1m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} MEM usgae high"
      description: "{{ $labels.instance }} MEM usage above 85% (current value: {{ $value }})"
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```

在告警规则文件中，我们可以将一组相关的规则设置定义在一个 group 下。在每一个 group 中我们可以定义多个告警规则(rule)。一条告警规则主要由以下几部分组成：

- alert：告警规则的名称。
- expr：基于 PromQL 表达式告警触发条件，用于计算是否有时间序列满足该条件。
- for：评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为 pending。
- labels：自定义标签，允许用户指定要附加到告警上的一组附加标签。
- annotations：用于指定一组附加信息，比如用于描述告警详细信息的文字等，annotations 的内容在告警产生时会一同作为参数发送到 Alertmanager。
  - summary：描述告警的概要信息
  - description：用于描述告警的详细信息
  - summary 和 description 以及 labels 均支持模板化

编辑好之后，重启 Prometheus 后访问 Prometheus UIhttp://127.0.0.1:9090/rules可以查看当前以加载的规则文件。

切换到 Alerts 标签http://127.0.0.1:9090/alerts可以查看当前告警的活动状态。

此时，我们可以手动拉高系统的 CPU 使用率，验证 Prometheus 的告警流程，在主机上运行以下命令：

```
cat /dev/zero>/dev/null
```

运行命令后查看 CPU 使用率情况

Prometheus 首次检测到满足触发条件后，hostCpuUsageAlert 显示由一条告警处于活动状态。由于告警规则中设置了 1m 的等待时间，当前告警状态为 PENDING

如果 1 分钟后告警条件持续满足，则会实际触发告警并且告警状态为 FIRING

## 部署 Alertmanager

Alertmanager 最新版本的下载地址可以从 Prometheus 官方网站 https://prometheus.io/download/ 获取。

```Shell
cd /usr/local
wget https://github.com/prometheus/alertmanager/releases/download/v0.15.2/alertmanager-0.15.2.linux-amd64.tar.gz
tar -zxvf alertmanager-0.15.2.linux-amd64/
cd alertmanager-0.15.2.linux-amd64/
./alertmanager
```

查看运行状态：Alertmanager 启动后可以通过 9093 端口访问，http://localhost:9093

### 关联 Prometheus 与 Alertmanager：

在 Prometheus 的架构中被划分成两个独立的部分。Prometheus 负责产生告警，而 Alertmanager 负责告警产生后的后续处理。因此 Alertmanager 部署完成后，需要在 Prometheus 中设置 Alertmanager 相关的信息。

编辑 Prometheus 配置文件 prometheus.yml,并添加以下内容

```
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

## 配置解析

Alertmanager 主要负责对 Prometheus 产生的告警进行统一处理，因此在 Alertmanager 配置中一般会包含以下几个主要部分：

- 全局配置（global）：用于定义一些全局的公共参数，如全局的 SMTP 配置，Slack 配置等内容；
- 模板（templates）：用于定义告警通知时的模板，如 HTML 模板，邮件模板等；
- 告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
- 接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack 或者 Webhook 等，接收人一般配合告警路由使用；
- 抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生

```
global:
  resolve_timeout: 5m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

配置好 webhook，再启动一个 go http 服务就可以接收到 alertmanager 发过来的告警信息了，我们可以对接公司 IM 或者短信平台都可以。

```
import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		s, _ := io.ReadAll(r.Body)
		fmt.Fprintf(w, "%s", s)
	})
	http.ListenAndServe(":5001", nil)
}
```

## 参考

https://prometheus.io/docs/alerting/latest/

https://yunlzheng.gitbook.io/prometheus-book/

