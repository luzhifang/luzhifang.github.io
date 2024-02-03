# RESTful API 设计规范


## 1. RESTful API 介绍

REST 本身并没有创造新的技术、组件或服务，它代表的是一种软件架构风格，是一组架构约束条件和原则，而不是技术框架。REST 它是 Representational State Transfer 的简称，中文的含义是: "表征状态转移" 或 "表现层状态转化"。它是基于 HTTP、URI、XML、JSON 等标准和协议，支持轻量级、跨平台、跨语言的架构设计。

REST 是一种规范，而 RESTful API 则是满足这种规范的 API 接口。

REST 风格虽然适用于很多传输协议，但在实际开发中，由于 REST 天生和 HTTP 协议相辅相成，因此 HTTP 协议已经成了实现 RESTful API 事实上的标准。

## 2. REST 设计原则

1. 每一个 URI 代表一种资源。
2. 同一种资源有多种表现形式(xml/json)。
3. 规范统一接口。
4. 返回一致的数据格式。
5. 所有的操作都是无状态的。
6. 可缓存(客户端可以缓存响应的内容)。

## 3. URL 及参数设计规范

1. 在 uri 末尾不需要出现斜杠/
2. 在 uri 中使用斜杠/是表达层级关系的。层级太多可以考虑使用 querystring 参数。
3. 在 uri 中可以使用连接符-, 来提升可读性。
4. 在 uri 中不允许出现下划线字符\_。
5. 在 uri 中尽量使用小写字符。
6. 在 uri 中不允许出现文件扩展名. 比如接口为 /xxx/test, 不要写成 /xxx/api.json 这样的是不合法的。
7. 在 uri 中使用名词复数形式。例：/students/3248234/courses/physics，集合->成员->集合->成员。

具体可以看：[https://blog.restcase.com/7-rules-for-rest-api-uri-design/](https://blog.restcase.com/7-rules-for-rest-api-uri-design/)

## 4. REST 资源操作映射为 HTTP 方法

| HTTP 方法 | 操作                                       | 是否安全(是否修改状态) | 幂等 |
| --------- | ------------------------------------------ | ---------------------- | ---- |
| GET       | 查询，取出资源                             | 是                     | 是   |
| POST      | 新增，新建一个资源                         | 否                     | 否   |
| PUT       | 更新，更新资源(客户端提供改变后的完整资源) | 否                     | 是   |
| PATCH     | 更新，更新部分资源(客户端提供改变的属性)   | 否                     | 是   |
| DELETE    | 删除，删除资源(批量/users?ids=1,2,3)       | 否                     | 是   |

## 5. API 版本管理

API 版本有不同的标识方法，在 RESTful API 开发中，通常将版本标识放在如下 3 个位
置：

1. URL 中，比如/v1/users。
2. HTTP Header 中，比如 Accept: vnd.example-com.foo+json; version=1.0。
3. querystring 参数中，比如/users?version=v1

没有严格限制，推荐放在 URL 中的，比如/v1/users，这样做的好处是很直观，GitHub、Kubernetes、Etcd 等很多优秀的 API 均采用这种方式。

## 6. 统一分页 / 过滤 / 排序 / 搜索功能

1. 分页：在列出一个 Collection 下所有的 Member 时，应该提供分页功能，例如/users?offset=0&limit=20（limit，指定返回记录的数量；offset，指定返回记录的开始位置）。引入分页功能可以减少 API 响应的延时，同时可以避免返回太多条目，导致服务器 / 客户端响应特别慢，甚至导致服务器 / 客户端 crash 的情况。
2. 过滤：如果用户不需要一个资源的全部状态属性，可以在 URI 参数里指定返回哪些属性，例如/users?fields=email,username,address。
3. 排序：用户很多时候会根据创建时间或者其他因素，列出一个 Collection 中前 100 个 Member，这时可以在 URI 参数中指明排序参数，例如/users?sort=age,desc。
4. 搜索：当一个资源的 Member 太多时，用户可能想通过搜索，快速找到所需要的 Member，或者想搜下有没有名字为 xxx 的某类资源，这时候就需要提供搜索功能。搜索建议按模糊匹配来搜索。例如/users?name=xxx。

## 7. 统一返回格式

RESTful 规范中的请求应该返回统一的数据格式。对于返回的数据，一般会包含如下字段:

1. code：http 响应的状态码（或者自定义的错误码）。
2. message：
3. data：当请求成功的时候, 返回的数据信息。

返回成功的响应 JSON 格式一般为如下:

```
{
    "code": 200,
    "message": "success",
    "data": [{
        "name": "tugenhua",
        "age": 31
    }]
}
```

返回失败的响应 JSON 格式一般为如下:

```
{
    "code": 401,
    "message": "用户没有权限",
    "data": null
}
```

## 8. 状态码

1. 各类型状态码

   - 1xx: 信息，请求收到了，继续处理。
   - 2xx: 代表成功. 行为被成功地接收、理解及采纳。
   - 3xx: 重定向。
   - 4xx: 客户端错误，请求包含语法错误或请求无法实现。
   - 5xx: 服务器端错误.

2. 2xx 状态码

   - 200 OK [GET]: 服务器端成功返回用户请求的数据。
   - 201 CREATED [POST/PUT/PATCH]: 用户新建或修改数据成功。
   - 202 Accepted 表示一个请求已经进入后台排队(一般是异步任务)。
   - 204 NO CONTENT -[DELETE]: 用户删除数据成功。

3. 4xx 状态码

   - 400：Bad Request - [POST/PUT/PATCH]: 用户发出的请求有错误，服务器不理解客户端的请求，未做任何处理。
   - 401: Unauthorized; 表示用户没有权限(令牌、用户名、密码错误)。
   - 403：Forbidden: 表示用户得到授权了，但是访问被禁止了, 也可以理解为不具有访问资源的权限。
   - 404：Not Found: 所请求的资源不存在，或不可用。
   - 405：Method Not Allowed: 用户已经通过了身份验证, 但是所用的 HTTP 方法不在它的权限之内。
   - 406：Not Acceptable: 用户的请求的格式不可得(比如用户请求的是 JSON 格式，但是只有 XML 格式)。
   - 410：Gone - [GET]: 用户请求的资源被转移或被删除。且不会再得到的。
   - 415: Unsupported Media Type: 客户端要求的返回格式不支持，比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。
   - 422：Unprocessable Entity: 客户端上传的附件无法处理，导致请求失败。
   - 429：Too Many Requests: 客户端的请求次数超过限额。

4. 5xx 状态码

   - 500：INTERNAL SERVER ERROR; 服务器发生错误。
   - 502：网关错误。
   - 503: Service Unavailable 服务器端当前无法处理请求。
   - 504：网关超时。

## 9. 域名

1. 应该尽量将 API 部署在专用域名之下。例：https://api.example.com
2. 如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。例：https://example.com/api/

## 10. 参考

https://blog.restcase.com

