# 分布式链路追踪系列1-OpenTracing 规范


## 简介

OpenTracing 是一个中立的（厂商无关、平台无关）分布式追踪的 API 规范，提供了统一接口方便开发者在自己的服务中集成一种或者多种分布式追踪的实现。

## OpenTracing 数据模型

**Trace**：OpenTracing 中的`Trace`（调用链）通过归属于此调用链的`Span`来隐性的定义，一条`Trace`可以被认为是一个由多个`Span`组成的有向无环图（DAG 图）。

**References**：`Span` 与 `Span` 的关系被命名为`References`。

**Span**：可以被翻译为跨度，可以被理解为一次方法调用, 一个程序块的调用, 或者一次 RPC/数据库访问.只要是一个具有完整时间周期的程序访问，都可以被认为是一个 `span`。

例如：下面的示例**Trace**就是由 8 个**Span**组成：

```
单个Trace中，span间的因果关系


        [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C 是 Span A 的孩子节点, ChildOf)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G 在 Span F 后被调用, FollowsFrom)

```

有些时候，使用下面这种，基于时间轴的时序图可以更好的展现**Trace**（调用链）：

```
单个Trace中，span间的时间关系

––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

每个**Span**包含以下的状态:

- An operation name，操作名称
- A start timestamp，起始时间
- A finish timestamp，结束时间
- **Span Tag**：一组键值对构成的 Span 标签集合。键值对中，键必须为 string，值可以是字符串，布尔，或者数字类型。
- **Span Log**：一组 span 的日志集合。
  每次 log 操作包含一个键值对，以及一个时间戳。
  键值对中，键必须为 string，值可以是任意类型。
  但是需要注意，不是所有的支持 OpenTracing 的 Tracer,都需要支持所有的值类型。
- **SpanContext**：上下文对象。SpanContext 携带着一些用于跨服务通信的（跨进程）数据，主要包含：
  - 任何一个 OpenTracing 的实现，都需要将当前调用链的状态（例如：trace 和 span 的 id），依赖一个独特的 Span 去跨进程边界传输。
  - **Baggage Items**：Trace 的随行数据，是一个键值对集合，它存在于 trace 中，也需要跨进程边界传输。
- **References**：Span 间的关系，相关的零个或者多个 Span（**Span**间通过**SpanContext**建立这种关系）

### Span 间关系

一个 Span 可以与一个或者多个**SpanContexts**存在因果关系。OpenTracing 目前定义了两种关系：`ChildOf`（父子） 和 `FollowsFrom`（跟随）。**这两种关系明确的给出了两个父子关系的 Span 的因果模型。** 将来，OpenTracing 可能提供非因果关系的 span 间关系。（例如：span 被批量处理，span 被阻塞在同一个队列中，等等）。

**`ChildOf` 引用:** 一个 span 可能是一个父级 span 的孩子，即"ChildOf"关系。在"ChildOf"引用关系下，父级 span 某种程度上取决于子 span。下面这些情况会构成"ChildOf"关系：

- 一个 RPC 调用的服务端的 span，和 RPC 服务客户端的 span 构成 ChildOf 关系
- 一个 sql insert 操作的 span，和 ORM 的 save 方法的 span 构成 ChildOf 关系
- 很多 span 可以并行工作（或者分布式工作）都可能是一个父级的 span 的子项，他会合并所有子 span 的执行结果，并在指定期限内返回

下面都是合理的表述一个"ChildOf"关系的父子节点关系的时序图。

```
    [-Parent Span---------]
         [-Child Span----]

    [-Parent Span--------------]
         [-Child Span A----]
          [-Child Span B----]
        [-Child Span C----]
         [-Child Span D---------------]
         [-Child Span E----]
```

**`FollowsFrom` 引用:** 一些父级节点不以任何方式依赖他们子节点的执行结果，这种情况下，我们说这些子 span 和父 span 之间是"FollowsFrom"的因果关系。"FollowsFrom"关系可以被分为很多不同的子类型，未来版本的 OpenTracing 中将正式的区分这些类型

下面都是合理的表述一个"FollowFrom"关系的父子节点关系的时序图。

```
    [-Parent Span-]  [-Child Span-]


    [-Parent Span--]
     [-Child Span-]


    [-Parent Span-]
                [-Child Span-]
```

## OpenTracing API

OpenTracing 标准中有三个重要的相互关联的类型，分别是`Tracer`, `Span` 和 `SpanContext`。每种类型的行为都会在各语言实现层面上，会演变成一个方法。

### `Tracer`

`Tracer`接口用来创建`Span`，以及处理如何`Inject`(serialize) 和 `Extract` (deserialize)，用于跨进程边界传递。它具有如下能力：

#### 创建一个新`Span`

必填参数

- **operation name**, 操作名, 一个具有可读性的字符串，代表这个 span 所做的工作（例如：RPC 方法名，方法名，或者一个大型计算中的某个阶段或子任务）。操作名应该是一个**抽象、通用，明确、具有统计意义**的名称。因此，`"get_user"` 作为操作名，比 `"get_user/314159"`更好。

例如，假设一个获取账户信息的 span 会有如下可能的名称：

| 操作名            | 指导意见                                                  |
| :---------------- | :-------------------------------------------------------- |
| `get`             | 太抽象                                                    |
| `get_account/792` | 太明确                                                    |
| `get_account`     | 正确的操作名，关于`account_id=792`的信息应该使用`Tag`操作 |

可选参数

- 零个或者多个关联（**references**）的`SpanContext`，如果可能，同时快速指定关系类型，`ChildOf` 还是 `FollowsFrom`。
- 一个可选的显性传递的**开始时间**；如果忽略，当前时间被用作开始时间。
- 零个或者多个**tag**。

**返回值**，返回一个已经启动`Span`实例。

#### 将`SpanContext`上下文 Inject（注入）到 carrier

必填参数

- **`SpanContext`** 实例
- **format**（格式化）描述，一般会是一个字符串常量，但不做强制要求。通过此描述，通知`Tracer`实现，如何对`SpanContext`进行编码放入到 carrier 中。
- **carrier**，根据**format**确定。`Tracer`实现根据**format**声明的格式，将`SpanContext`序列化到 carrier 对象中。

#### 将`SpanContext`上下文从 carrier 中 Extract（提取）

必填参数

- **format**（格式化）描述，一般会是一个字符串常量，但不做强制要求。通过此描述，通知`Tracer`实现，如何从 carrier 中解码`SpanContext`。
- **carrier**，根据**format**确定。`Tracer`实现根据**format**声明的格式，从 carrier 中解码`SpanContext`。

**返回值**，返回一个`SpanContext`实例，可以使用这个`SpanContext`实例，通过`Tracer`创建新的`Span`。

#### 注意，对于 Inject（注入）和 Extract（提取），**format**是必须的。

Inject（注入）和 Extract（提取）依赖于可扩展的**format**参数。**format**参数规定了另一个参数"carrier"的类型，同时约束了"carrier"中`SpanContext`是如何编码的。所有的 Tracer 实现，都必须支持下面的**format**。

- **Text Map**: 基于字符串：字符串的 map,对于 key 和 value 不约束字符集。
- **HTTP Headers**: 适合作为 HTTP 头信息的，基于字符串：字符串的 map。（[RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.2.4).在工程实践中，如何处理 HTTP 头具有多样性，强烈建议 tracer 的使用者谨慎使用 HTTP 头的键值空间和转义符）
- **Binary**: 一个简单的二进制大对象，记录`SpanContext`的信息。

### Span

当`Span`结束后(`span.finish()`)，除了通过`Span`获取`SpanContext`外，下列其他所有方法都不允许被调用。

#### 获取`Span`的`SpanContext`

不需要任何参数。

**返回值**，`Span`构建时传入的`SpanContext`。这个返回值在`Span`结束后(`span.finish()`)，依然可以使用。

#### 复写操作名（operation name）

必填参数

- 新的操作名**operation name**，覆盖构建`Span`时，传入的操作名。

#### 结束`Span`

可选参数

- 一个明确的**完成时间**;如果省略此参数，使用当前时间作为完成时间。

#### 为`Span`设置 tag

必填参数

- tag key，必须是 string 类型
- tag value，类型为字符串，布尔或者数字

注意，OpenTracing 标准包含**["standard tags，标准 Tag"](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/semantic_conventions.md)**，此文档中定义了 Tag 的标准含义。

#### Log 结构化数据

必填参数

- 一个或者多个键值对，其中键必须是字符串类型，值可以是任意类型。某些 OpenTracing 实现，可能支持更多的 log 值类型。

可选参数

- 一个明确的时间戳。如果指定时间戳，那么它必须在 span 的开始和结束时间之内。

注意，OpenTracing 标准包含**["standard log keys，标准 log 的键"](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/semantic_conventions.md)**，此文档中定义了这些键的标准含义。

#### 设置一个**baggage**（随行数据）元素

Baggage 元素是一个键值对集合，将这些值设置给给定的`Span`，`Span`的`SpanContext`，以及**所有和此`Span`有直接或者间接关系的本地`Span`。** 也就是说，baggage 元素随 trace 一起保持在带内传递。（译者注：带内传递，在这里指，随应用程序调用过程一起传递）

Baggage 元素具有强大的功能，使得 OpenTracing 能够实现全栈集成（例如：任意的应用程序数据，可以在移动端创建它，显然的，它会一直传递了系统最底层的存储系统），同时他也会产生巨大的开销，请小心使用此特性。

必填参数

- **baggage key**, 字符串类型
- **baggage value**, 字符串类型

#### 获取一个**baggage**元素

必填参数

- **baggage key**, 字符串类型

**返回值**，相应的**baggage value**,或者可以标识元素值不存在的返回值（译者注：如 Null）。

### `SpanContext`

相对于 OpenTracing 中其他的功能，`SpanContext`更多的是一个“概念”。也就是说，OpenTracing 实现中，需要重点考虑，并提供一套自己的 API。
OpenTracing 的使用者仅仅需要，在创建 span、向传输协议 Inject（注入）和从传输协议中 Extract（提取）时，使用`SpanContext`和`references`。

OpenTracing 要求，`SpanContext`是**不可变**的，目的是防止由于`Span`的结束和相互关系，造成的复杂生命周期问题。

#### 遍历所有的 baggage 元素

遍历模型依赖于语言，实现方式可能不一致。在语义上，要求调用者可以通过给定的`SpanContext`实例，高效的遍历所有的 baggage 元素

### `NoopTracer`

所有的 OpenTracing API 实现，必须提供某种方式的`NoopTracer`实现。`NoopTracer`可以被用作控制或者测试时，进行无害的 inject 注入（等等）。例如，在 OpenTracing-Java 实现中，`NoopTracer`在他自己的模块中。

### 可选 API 元素

有些语言的 OpenTracing 实现，为了在串行处理中，传递活跃的`Span`或`SpanContext`，提供了一些工具类。例如，`opentracing-go`中，通过`context.Context`机制，可以设置和获取活跃的`Span`。

## 参考

https://opentracing.io/docs/overview/

https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md

https://github.com/opentracing/specification/blob/master/specification.md

https://github.com/opentracing/specification/blob/master/semantic_conventions.md

