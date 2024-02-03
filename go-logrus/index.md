# Go常用库-logrus


## 简介

logrus 是一个结构化、插件化的日志记录库。完全兼容 golang 标准库中的日志模块。

它有以下特点：

- 完全兼容标准日志库，拥有七种日志级别：Trace, Debug, Info, Warning, Error, Fataland Panic。
- 可扩展的 Hook 机制，允许使用者通过 Hook 的方式将日志分发到任意地方，如本地文件系统，logstash，elasticsearch 或者 mq 等，或者通过 Hook 定义日志内容和格式等
- 可选的日志输出格式，内置了两种日志格式 JSONFormater 和 TextFormatter，还可以自定义日志格式
- Field 机制，通过 Filed 机制进行结构化的日志记录
- 线程安全

## 安装

```Shell
go get github.com/sirupsen/logrus
```

## 简单示例

```Go
package main

import (
  log "github.com/sirupsen/logrus"
)

func main() {
  log.WithFields(log.Fields{
	"animal": "walrus",
  }).Info("a walrus appears")
}
```

输出：

```
time="2021-11-11T17:41:48+08:00" level=info msg="a walrus appears" animal=walrus
```

## 日志级别

Logrus 有七个日志级别：Trace, Debug, Info, Warning, Error, Fataland Panic。

```Go
log.Trace("Something very low level.")
log.Debug("Useful debugging information.")
log.Info("Something noteworthy happened!")
log.Warn("You should probably take a look at this.")
log.Error("Something failed but I'm not quitting.")
// Calls os.Exit(1) after logging
log.Fatal("Bye.")
// Calls panic() after logging
log.Panic("I'm bailing.")
```

### 设置日志级别

你可以在 Logger 上设置日志记录级别，然后它只会记录具有该级别或以上级别任何内容的条目：

```Go
// 会记录info及以上级别 (warn, error, fatal, panic)
log.SetLevel(log.InfoLevel)
```

## 字段

Logrus 鼓励通过日志字段进行谨慎的结构化日志记录，而不是冗长的、不可解析的错误消息。

例如，区别于使用 log.Fatalf("Failed to send event %s to topic %s with key %d")，你应该使用如下方式记录更容易发现的内容:

```Go
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

WithFields 的调用是可选的。

## 默认字段

通常，将一些字段始终附加到应用程序的全部或部分的日志语句中会很有帮助。例如，你可能希望始终在请求的上下文中记录 request_id 和 user_ip。

区别于在每一行日志中写上 log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})，你可以向下面的示例代码一样创建一个 logrus.Entry 去传递这些字段。

```Go
requestLogger := log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})
requestLogger.Info("something happened on that request") # will log request_id and user_ip
requestLogger.Warn("something not great happened")
```

## 日志条目

除了使用 WithField 或 WithFields 添加的字段外，一些字段会自动添加到所有日志记录事中:

time：记录日志时的时间戳
msg：记录的日志信息
level：记录的日志级别

## 设置输出方式

```Go
// 输出到 Stdout
log.SetOutput(os.Stdout)

// 输出到文件里
logfile, _ := os.OpenFile("./logrus.log", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0644)
logrus.SetOutput(logfile)

// 输出到多个地方
logrus.SetOutput(io.MultiWriter(os.Stdout, logfile))
```

## 格式化

logrus 内置以下两种日志格式化程序：

logrus.TextFormatter logrus.JSONFormatter

```Go
log.SetFormatter(&log.JSONFormatter{})
```

可以根据 Formatter 接口自定义日志格式。

还支持一些第三方的格式化程序，详见项目首页。

## 记录函数名

如果你希望将调用的函数名添加为字段，请通过以下方式设置：

```Go
log.SetReportCaller(true)
```

这会将调用者添加为”method”，如下所示：

```
{"animal":"penguin","level":"fatal","method":"github.com/sirupsen/arcticcreatures.migrate","msg":"a penguin swims by","time":"2014-03-10 19:57:38.562543129 -0400 EDT"}
```

注意：，开启这个模式会增加性能开销。

## 切分日志文件

如果日志文件太大了，想切分成小文件，但是 logrus 没有提供这个功能。

一种是借助 linux 系统的 logrotate 命令来切分 logrus 生成的日志文件。

另外一种是用 logrus 的 hook 功能，做一个切分日志的插件。

我们使用[lumberjack](https://github.com/natefinch/lumberjack)切分。

```Go
package main

import (
  log "github.com/sirupsen/logrus"

  "gopkg.in/natefinch/lumberjack.v2"
)

func main() {
  logger := &lumberjack.Logger{
    Filename:   "./testlogrus.log",
    MaxSize:    500,  // 日志文件大小，单位是 MB
    MaxBackups: 3,    // 最大过期日志保留个数
    MaxAge:     28,   // 保留过期文件最大时间，单位 天
    Compress:   true, // 是否压缩日志，默认是不压缩。这里设置为true，压缩日志
  }

  log.SetOutput(logger) // logrus 设置日志的输出方式

}
```

## Hooks

你可以添加日志级别的钩子（Hook）。例如，向异常跟踪服务发送 Error、Fatal 和 Panic、信息到 StatsD 或同时将日志发送到多个位置，例如 syslog。

Logrus 配有内置钩子。在 init 中添加这些内置钩子或你自定义的钩子：

```Go
import (
  log "github.com/sirupsen/logrus"
  "gopkg.in/gemnasium/logrus-airbrake-hook.v2" // the package is named "airbrake"
  logrus_syslog "github.com/sirupsen/logrus/hooks/syslog"
  "log/syslog"
)

func init() {

  // Use the Airbrake hook to report errors that have Error severity or above to
  // an exception tracker. You can create custom hooks, see the Hooks section.
  log.AddHook(airbrake.NewHook(123, "xyz", "production"))

  hook, err := logrus_syslog.NewSyslogHook("udp", "localhost:514", syslog.LOG_INFO, "")
  if err != nil {
    log.Error("Unable to connect to local syslog daemon")
  } else {
    log.AddHook(hook)
  }
}
```

意：Syslog 钩子还支持连接到本地 syslog（例如. “/dev/log” or “/var/run/syslog” or “/var/run/log”)。有关详细信息，请查看[syslog hook README](https://github.com/sirupsen/logrus/blob/master/hooks/syslog/README.md)。

## 线程安全

默认的 logger 在并发写的时候是被 mutex 保护的，比如当同时调用 hook 和写 log 时 mutex 就会被请求，有另外一种情况，文件是以 appending mode 打开的， 此时的并发操作就是安全的，可以用 logger.SetNoLock()来关闭它。

## 参考

https://github.com/sirupsen/logrus

