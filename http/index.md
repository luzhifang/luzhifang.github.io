# HTTP


## HTTP 协议简介

HTTP（Hypertext Transfer Protocol，超文本传输协议）是在万维网上进行通信时所使用的协议方案。HTTP 有很多应用，但最著名的是用于 Web 浏览器和 Web 服务器之间的双工通信。

## URI 与 URL

- URI(Uniform Resource Identifier)：统一资源标识符
- URL(Uniform Resource Locator)：统一资源定位符；URL 是 URI 的子集
- URN(Uniform Resource Name)：URI 的第二种形式就是统一资源名（URN）。URN 是作为特定内容的唯一名称使用的，与目前的资源所在地无关。使用这些与位置无关的 URN，就可以将资源四处搬移。

URI 格式如下图：

<img src="/images/net/uri-format.png" alt="" width="90%" />

## WEB 结构组件

| 组件       | 功能                                                                      |
| ---------- | ------------------------------------------------------------------------- |
| 代理       | 位于客户端和服务器之间的 HTTP 中间实体。                                  |
| 缓存       | HTTP 的仓库，使常用页面的副本可以保存在离客户端更近的地方。               |
| 网关       | 连接其他应用程序的特殊 Web 服务器，通常用于将 HTTP 流量转换成其他的协议。 |
| 隧道       | 对 HTTP 通信报文进行盲转发的特殊代理。                                    |
| Agent 代理 | 发起自动 HTTP 请求的半智能 Web 客户端。                                   |

## 媒体类型

HTTP 仔细地给每种要通过 Web 传输的对象都打上了名为 MIME 类型（MIME type）的数据格式标签。最初设计 MIME（Multipurpose Internet Mail Extension，多用途因特网邮件扩展）是为了解决在不同的电子邮件系统之间搬移报文时存在的问题。MIME 在电子邮件系统中工作得非常好，因此 HTTP 也采纳了它，用它来描述并标记多媒体内容。

常见的 MIME 类型

- 超文本标记语言文本 .html、.html：text/html
- 普通文本 .txt： text/plain
- RTF 文本 .rtf： application/rtf
- GIF 图形 .gif： image/gif
- JPEG 图形 .jpeg、.jpg： image/jpeg
- au 声音文件 .au： audio/basic
- MIDI 音乐文件 mid、.midi： audio/midi、audio/x-midi
- RealAudio 音乐文件 .ra、.ram： audio/x-pn-realaudio
- MPEG 文件 .mpg、.mpeg： video/mpeg
- AVI 文件 .avi： video/x-msvideo
- GZIP 文件 .gz： application/x-gzip
- TAR 文件 .tar： application/x-tar
- multipart 类型：multipart/form-data 上传文件时使用

## HTTP 报文

### 请求报文

1. 请求行，分别是
2. 请求头
3. 空行
4. 数据主体（body）部分

```XML
<method> <request-URL> <version>
<headers>

<entity-body>
```

### 响应报文

1. 状态行
2. 响应头
3. 空行
4. 数据主体（body）部分

```XML
<version> <status> <reason-phrase>
<headers>

<entity-body>
```

## HTTP 首部

### 通用首部

| header            | 解释                                           | 示例                                           |
| ----------------- | ---------------------------------------------- | ---------------------------------------------- |
| Cache-Control     | 控制缓存的行为                                 | no-cache/no-store/max-age=0                    |
| Connection        | 决定当前的事务完成后，是否会关闭网络连接       | close/keep-alive                               |
| Date              | 创建报文的日期时间                             | Date: Tue, 15 Nov 2010 08:12:31 GMT            |
| Keep-Alive        | 用来设置超时时长和最大请求数                   | Keep-Alive: timeout=5, max=100                 |
| Via               | 代理服务器的相关信息                           | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)    |
| Warning           | 错误通知，警告实体可能存在的问题               | Warning: 199 Miscellaneous warning             |
| Trailer           | 允许发送方在分块发送的消息后面添加额外的元信息 | Trailer: Max-Forwards                          |
| Transfer-Encoding | 指定报文主体的传输编码方式                     | Transfer-Encoding:chunked                      |
| Upgrade           | 升级为其他协议                                 | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11 |

### 请求首部

| header                    | 解释                                            | 示例                                           |
| ------------------------- | ----------------------------------------------- | ---------------------------------------------- |
| Accept                    | 客户端可以处理的内容类型                        | \*/\*,text/\*                                  |
| Accept-Charset            | 客户端可以处理的字符集类型                      | iso-latin-1                                    |
| Accept-Encoding           | 客户端能够理解的内容编码方式                    | compress, gzip                                 |
| Accept-Language           | 客户端可以理解的自然语言                        | en,zh                                          |
| Authorization             | Web 认证信息                                    | Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==             |
| Cookie                    | 通过 Set-Cookie 设置的值                        | $Version=1; Skin=new;                          |
| DNT                       | 表明用户对于网站追踪的偏好                      |                                                |
| From                      | 用户的电子邮箱地址                              | user@email.com                                 |
| Host                      | 请求资源所在服务器                              | www.zcmhi.com                                  |
| If-Match                  | 比较实体标记（ETag）                            | 737060cd8c284d8af7ad3082f209582d               |
| If-Modified-Since         | 比较资源的更新时间                              | Sat, 29 Oct 2010 19:43:31 GMT                  |
| If-None-Match             | 比较实体标记（与 If-Match 相反）                | 737060cd8c284d8af7ad3082f209582d               |
| If-Range                  | 资源未更新时发送实体 Byte 的范围请求            | 737060cd8c284d8af7ad3082f209582d               |
| If-Unmodified-Since       | 比较资源的更新时间（与 If-Modified-Since 相反） | Sat, 29 Oct 2010 19:43:31 GMT                  |
| Origin                    | 表示当前请求资源所在页面的协议和域名            | http://test.my.com                             |
| Proxy-Authorization       | 代理服务器要求客户端的认证信息                  | Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==             |
| Range                     | 只请求实体的一部分，指定范围                    | Range: bytes=500-999                           |
| Referer                   | 先前网页的地址，当前请求网页紧随其后,即来路     | Referer: http://www.zcmhi.com/archives/71.html |
| TE                        | 指定用户代理希望使用的传输编码类型              | TE: trailers,deflate;q=0.5                     |
| Upgrade-Insecure-Requests | 表示客户端优先选择加密及带有身份验证的响应      |                                                |
| User-Agent                | 浏览器信息                                      | User-Agent: Mozilla/5.0 (Linux; X11)           |

### 响应首部

| header                      | 解释                                                              | 示例                                                |
| --------------------------- | ----------------------------------------------------------------- | --------------------------------------------------- |
| Accept-Ranges               | 是否接受字节范围请求                                              | none/bytes                                          |
| Age                         | 消息对象在缓存代理中存贮的时长，以秒为单位                        | 60                                                  |
| ETag                        | 资源的匹配信息                                                    | 737060cd8c284d8af7ad3082f209582d                    |
| Location                    | 令客户端重定向至指定 URI                                          | Location: http://www.zcmhi.com/archives/94.html     |
| Proxy-Authenticate          | 代理服务器对客户端的认证信息                                      | Basic                                               |
| Public-Key-Pins             | 包含该 Web 服务器用来进行加密的 public key （公钥）信息           |                                                     |
| Public-Key-Pins-Report-Only | 设置在公钥固定不匹配时，发送错误信息到 report-uri                 |                                                     |
| Referrer-Policy             | 用来监管哪些访问来源信息，会在 Referer 中发送                     |                                                     |
| Server                      | HTTP 服务器的安装信息                                             | Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)        |
| Set-Cookie                  | 服务器端向客户端发送 cookie                                       | Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1 |
| Strict-Transport-Security   | 它告诉浏览器只能通过 HTTPS 访问当前资源                           |                                                     |
| Timing-Allow-Origin         | 用于指定特定站点，以允许其访问 Resource Timing API 提供的相关信息 |                                                     |
| Tk                          | 显示了对相应请求的跟踪情况                                        |                                                     |
| Vary                        | 服务器缓存的管理信息                                              | Vary: \*                                            |
| WWW-Authenticate            | 定义了使用何种验证方式去获取对资源的连接                          | Basic                                               |
| X-XSS-Protection            | 当检测到跨站脚本攻击 (XSS)时，浏览器将停止加载页面                |                                                     |

### 实体首部

| header           | 解释                                     | 示例                                         |
| ---------------- | ---------------------------------------- | -------------------------------------------- |
| Allow            | 对某网络资源的有效的请求行为             | GET, HEAD                                    |
| Content-Encoding | 用于对特定媒体类型的数据进行压缩         | gzip                                         |
| Content-Language | 访问者希望采用的语言或语言组合           | en,zh                                        |
| Content-Length   | 发送给接收方的消息主体的大小             | 348                                          |
| Content-Location | 请求资源可替代的备用的另一地址           | /index.htm                                   |
| Content-Range    | 在整个返回体中本部分的字节位置           | Content-Range: bytes 21010-47021/47022       |
| Content-Type     | 告诉客户端实际返回的内容的内容类型       | text/html; charset=utf-8                     |
| Expires          | 包含日期/时间， 即在此时候之后，响应过期 | Expires: Thu, 01 Dec 2010 16:00:00 GMT       |
| Last-Modified    | 资源的最后修改日期时间                   | Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT |

注意：还有一些扩展首部，规范中没有定义的新首部。

## HTTP 请求方法

HTTP1.0 定义了三种请求方法： GET, POST 和 HEAD 方法。

HTTP1.1 新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT 方法。

| 方法    | 描述                                                                                                                                     |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| GET     | 请求指定的页面信息，并返回实体主体。                                                                                                     |
| HEAD    | 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头                                                                          |
| POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。 |
| PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。                                                                                         |
| DELETE  | 请求服务器删除指定的页面。                                                                                                               |
| CONNECT | HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。                                                                                |
| OPTIONS | 允许客户端查看服务器的性能。                                                                                                             |
| TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                                                                                               |
| PATCH   | 是对 PUT 方法的补充，用来对已知资源进行局部更新 。                                                                                       |

### GET 和 POST 的区别

1. get 是获取数据，post 是修改数据
2. get 把请求的数据放在 url 上， 以?分割 URL 和传输数据，参数之间以&相连，所以 get 不太安全。而 post 把数据放在 HTTP 的包体内（request body 相对安全）
3. get 提交的数据最大是 2k（ 限制实际上取决于浏览器）， post 理论上没有限制。
4. GET 产生一个 TCP 数据包，浏览器会把 http header 和 data 一并发送出去，服务器响应 200(返回数据); POST 产生两个 TCP 数据包，浏览器先发送 header，服务器响应 100 continue，浏览器再发送 data，服务器响应 200 ok(返回数据)。
5. GET 请求会被浏览器主动缓存，而 POST 不会，除非手动设置。
6. 本质区别：GET 是幂等的，而 POST 不是幂等的

这里的幂等性：幂等性是指一次和多次请求某一个资源应该具有同样的副作用。简单来说意味着对同一 URL 的多个请求应该返回同样的结果。

正因为它们有这样的区别，所以不应该且不能用 get 请求做数据的增删改这些有副作用的操作。因为 get 请求是幂等的，在网络不好的隧道中会尝试重试。如果用 get 请求增数据，会有重复操作的风险，而这种重复操作可能会导致副作用（浏览器和操作系统并不知道你会用 get 请求去做增操作）。

## HTTP 状态码

HTTP 状态码分类

| 状态码 | 类型                        | 说明                       |
| ------ | --------------------------- | -------------------------- |
| 1xx    | Informational(信息性状态码) | 接收的请求正在处理         |
| 2xx    | Success(成功)               | 请求正常处理完毕           |
| 3xx    | Redirection(重定向)         | 需要进行附加操作以完成请求 |
| 4xx    | Client Error(客户端错误)    | 服务器无法处理请求         |
| 5xx    | Server Error(服务端错误)    | 服务器处理请求出错         |

HTTP 状态码列表

| 状态码 | 状态码英文名称                  | 中文描述                                                                                                                                                         |
| ------ | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 100    | Continue                        | 继续。客户端应继续其请求                                                                                                                                         |
| 101    | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到 HTTP 的新版本协议                                                                 |
| 200    | OK                              | 请求成功。一般用于 GET 与 POST 请求                                                                                                                              |
| 201    | Created                         | 已创建。成功请求并创建了新的资源                                                                                                                                 |
| 202    | Accepted                        | 已接受。已经接受请求，但未处理完成                                                                                                                               |
| 203    | Non-Authoritative Information   | 非授权信息。请求成功。但返回的 meta 信息不在原始的服务器，而是一个副本                                                                                           |
| 204    | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档                                                                         |
| 205    | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域                                                               |
| 206    | Partial Content                 | 部分内容。服务器成功处理了部分 GET 请求                                                                                                                          |
| 300    | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择                                                           |
| 301    | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新 URI，返回信息会包括新的 URI，浏览器会自动定向到新 URI。今后任何新的请求都应使用新的 URI 代替                              |
| 302    | Found                           | 临时移动。与 301 类似。但资源只是临时被移动。客户端应继续使用原有 URI                                                                                            |
| 303    | See Other                       | 查看其它地址。与 301 类似。使用 GET 和 POST 请求查看                                                                                                             |
| 304    | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305    | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                                                                                                                           |
| 306    | Unused                          | 已经被废弃的 HTTP 状态码                                                                                                                                         |
| 307    | Temporary Redirect              | 临时重定向。与 302 类似。使用 GET 请求重定向                                                                                                                     |
| 400    | Bad Request                     | 客户端请求的语法错误，服务器无法理解                                                                                                                             |
| 401    | Unauthorized                    | 请求要求用户的身份认证                                                                                                                                           |
| 402    | Payment Required                | 保留，将来使用                                                                                                                                                   |
| 403    | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求                                                                                                                   |
| 404    | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面                                                     |
| 405    | Method Not Allowed              | 客户端请求中的方法被禁止                                                                                                                                         |
| 406    | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                                                                                                                       |
| 407    | Proxy Authentication Required   | 请求要求代理的身份认证，与 401 类似，但请求者应当使用代理进行授权                                                                                                |
| 408    | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                                                                                                                         |
| 409    | Conflict                        | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突                                                                                          |
| 410    | Gone                            | 客户端请求的资源已经不存在。410 不同于 404，如果资源以前有现在被永久删除了可使用 410 代码，网站设计人员可通过 301 代码指定资源的新位置                           |
| 411    | Length Required                 | 服务器无法处理客户端发送的不带 Content-Length 的请求信息                                                                                                         |
| 412    | Precondition Failed             | 客户端请求信息的先决条件错误                                                                                                                                     |
| 413    | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个 Retry-After 的响应信息  |
| 414    | Request-URI Too Large           | 请求的 URI 过长（URI 通常为网址），服务器无法处理                                                                                                                |
| 415    | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                                                                                                                                 |
| 416    | Requested range not satisfiable | 客户端请求的范围无效                                                                                                                                             |
| 417    | Expectation Failed              | 服务器无法满足 Expect 的请求头信息                                                                                                                               |
| 500    | Internal Server Error           | 服务器内部错误，无法完成请求                                                                                                                                     |
| 501    | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                                                                                                                             |
| 502    | Bad Gateway                     | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应                                                                                   |
| 503    | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的 Retry-After 头信息中                                                            |
| 504    | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求                                                                                                               |
| 505    | HTTP Version not supported      | 服务器不支持请求的 HTTP 协议的版本，无法完成处理                                                                                                                 |

## Cookie 与 Session

HTTP 作为无状态协议，必然需要在某种方式保持连接状态。保持状态可使用 Cookie 和 Session。

### Cookie

Cookie 是客户端保持状态的方法。

Cookie 简单的理解就是存储由服务器发至客户端并由客户端保存的一段字符串。为了保持会话，服务器可以在响应客户端请求时将 Cookie 字符串放在 Set-Cookie 下，客户机收到 Cookie 之后保存这段字符串，之后再请求时候带上 Cookie 就可以被识别。

除了上面提到的这些，Cookie 在客户端的保存形式可以有两种，一种是会话 Cookie 一种是持久 Cookie，会话 Cookie 就是将服务器返回的 Cookie 字符串保持在内存中，关闭浏览器之后自动销毁，持久 Cookie 则是存储在客户端磁盘上，其有效时间在服务器响应头中被指定，在有效期内，客户端再次请求服务器时都可以直接从本地取出。需要说明的是，存储在磁盘中的 Cookie 是可以被多个浏览器代理所共享的。

### Session

Session 是服务器保持状态的方法。

首先需要明确的是，Session 保存在服务器上，可以保存在数据库、文件或内存中，每个用户有独立的 Session 用户在客户端上记录用户的操作。我们可以理解为每个用户有一个独一无二的 Session ID 作为 Session 文件的 Hash 键，通过这个值可以锁定具体的 Session 结构的数据，这个 Session 结构中存储了用户操作行为。

当服务器需要识别客户端时就需要结合 Cookie 了。每次 HTTP 请求的时候，客户端都会发送相应的 Cookie 信息到服务端。实际上大多数的应用都是用 Cookie 来实现 Session 跟踪的，第一次创建 Session 的时候，服务端会在 HTTP 协议中告诉客户端，需要在 Cookie 里面记录一个 Session ID，以后每次请求把这个会话 ID 发送到服务器，我就知道你是谁了。如果客户端的浏览器禁用了 Cookie，会使用一种叫做 URL 重写的技术来进行会话跟踪，即每次 HTTP 交互，URL 后面都会被附加上一个诸如 sid=xxxxx 这样的参数，服务端据此来识别用户，这样就可以帮用户完成诸如用户名等信息自动填入的操作了。

## HTTP 安全

### XSS 攻击

跨站点脚本攻击，指攻击者通过篡改网页，嵌入恶意脚本程序，在用户浏览网页时，控制用户浏览器进行恶意操作的一种攻击方式。

防范措施：

1. 前端，服务端，同时需要校验字符串输入的长度限制。
2. 前端，服务端，同时需要对 HTML 转义处理。将其中的”<”,”>”等特殊字符进行转义编码。 防 XSS 的核心是必须对输入的数据做过滤处理。

### CSRF 攻击

跨站点请求伪造，指攻击者通过跨站请求，以合法的用户的身份进行非法操作。可以这么理解 CSRF 攻击：攻击者盗用你的身份，以你的名义向第三方网站发送恶意请求。CRSF 能做的事情包括利用你的身份发邮件，发短信，进行交易转账，甚至盗取账号信息。

防范措施：

1. token 机制。在 HTTP 请求中进行 token 验证，如果请求中没有 token 或者 token 内容不正确，则认为 CSRF 攻击而拒绝该请求。
2. 验证码。通常情况下，验证码能够很好的遏制 CSRF 攻击，但是很多情况下，出于用户体验考虑，验证码只能作为一种辅助手段，而不是最主要的解决方案。
3. referer 识别。在 HTTP Header 中有一个字段 Referer，它记录了 HTTP 请求的来源地址。如果 Referer 是其他网站，就有可能是 CSRF 攻击，则拒绝该请求。

### 文件上传漏洞

文件上传漏洞，指的是用户上传一个可执行的脚本文件，并通过此脚本文件获得了执行服务端命令的能力。 许多第三方框架、服务，都曾经被爆出文件上传漏洞，比如很早之前的 Struts2，以及富文本编辑器等等，可被攻击者上传恶意代码，有可能服务端就被人黑了。

防范措施：

1. 文件上传的目录设置为不可执行。
2. 判断文件类型。在判断文件类型的时候，可以结合使用 MIME Type，后缀检查等方式。因为对于上传文件，不能简单地通过后缀名称来判断文件的类型，因为攻击者可以将可执行文件的后缀名称改为图片或其他后缀类型，诱导用户执行。
3. 对上传的文件类型进行白名单校验，只允许上传可靠类型。
4. 上传的文件需要进行重新命名，使攻击者无法猜想上传文件的访问路径，将极大地增加攻击成本，同时向 shell.php.rar.ara 这种文件，因为重命名而无法成功实施攻击。
5. 限制上传文件的大小。
6. 单独设置文件服务器的域名。

## HTTP 性能优化

- 并行连接：通过多条 TCP 连接发起并发的 HTTP 请求。
- 持久连接：重用 TCP 连接，以消除连接及关闭时延。为解决每进行一次 HTTP 通信就要断开一次 TCP 连接，HTTP/1.1 通过 keep-alive 持久连接，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。
- 管道化连接：通过共享的 TCP 连接发起并发的 HTTP 请求。
- 复用的连接：交替传送请求和响应报文（实验阶段）。

## HTTP1.0 和 HTTP1.1 的区别

1. 长连接(Persistent Connection)

HTTP1.1 支持长连接和请求的流水线处理，在一个 TCP 连接上可以传送多个 HTTP 请求和响应，减少了建立和关闭连接的消耗和延迟，在 HTTP1.1 中默认开启长连接 keep-alive，一定程度上弥补了 HTTP1.0 每次请求都要创建连接的缺点。HTTP1.0 需要使用 keep-alive 参数来告知服务器端要建立一个长连接。

2. 节约带宽

HTTP1.0 中存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能。HTTP1.1 支持只发送 header 信息（不带任何 body 信息），如果服务器认为客户端有权限请求服务器，则返回 100，客户端接收到 100 才开始把请求 body 发送到服务器；如果返回 401，客户端就可以不用发送请求 body 了节约了带宽。

3. HOST 域

在 HTTP1.0 中认为每台服务器都绑定一个唯一的 IP 地址，因此，请求消息中的 URL 并没有传递主机名（hostname），HTTP1.0 没有 host 域。随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个 IP 地址。HTTP1.1 的请求消息和响应消息都支持 host 域，且请求消息中如果没有 host 域会报告一个错误（400 Bad Request）。

4. 缓存处理

在 HTTP1.0 中主要使用 header 里的 If-Modified-Since,Expires 来做为缓存判断的标准，HTTP1.1 则引入了更多的缓存控制策略例如 Entity tag，If-Unmodified-Since, If-Match, If-None-Match 等更多可供选择的缓存头来控制缓存策略。

5. 错误通知的管理

在 HTTP1.1 中新增了 24 个错误状态响应码，如 409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

## HTTP1.1 和 HTTP2.0 的区别

1. 多路复用

HTTP2.0 使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比 HTTP1.1 大了好几个数量级。HTTP1.1 也可以多建立几个 TCP 连接，来支持处理更多并发的请求，但是创建 TCP 连接本身也是有开销的。

2. 头部数据压缩

在 HTTP1.1 中，HTTP 请求和响应都是由状态行、请求/响应头部、消息主体三部分组成。一般而言，消息主体都会经过 gzip 压缩，或者本身传输的就是压缩过后的二进制文件，但状态行和头部却没有经过任何压缩，直接以纯文本传输。随着 Web 功能越来越复杂，每个页面产生的请求数也越来越多，导致消耗在头部的流量越来越多，尤其是每次都要传输 UserAgent、Cookie 这类不会频繁变动的内容，完全是一种浪费。

HTTP1.1 不支持 header 数据的压缩，HTTP2.0 使用 HPACK 算法对 header 的数据进行压缩，这样数据体积小了，在网络上传输就会更快。

3. 服务器推送

服务端推送是一种在客户端请求之前发送数据的机制。网页使用了许多资源：HTML、样式表、脚本、图片等等。在 HTTP1.1 中这些资源每一个都必须明确地请求。这是一个很慢的过程。浏览器从获取 HTML 开始，然后在它解析和评估页面的时候，增量地获取更多的资源。因为服务器必须等待浏览器做每一个请求，网络经常是空闲的和未充分使用的。

为了改善延迟，HTTP2.0 引入了 server push，它允许服务端推送资源给浏览器，在浏览器明确地请求之前，免得客户端再次创建连接发送请求到服务器端获取。这样客户端可以直接从本地加载这些资源，不用再通过网络。

## 参考

[《HTTP 权威指南》](https://weread.qq.com/web/reader/1d9321c0718ff5e11d9afe8)

https://www.huweihuang.com/linux-notes/tcpip/http-basics.html

https://www.runoob.com/http/http-status-codes.html

https://juejin.cn/post/6844903489596833800

