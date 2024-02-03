# Protocol Buffers 详解


## 简介

protocol buffers （ProtoBuf）是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10 倍）、更快（20 ~ 100 倍）、更为简单。

json\xml 都是基于文本格式，protobuf 是二进制格式。

你可以通过 ProtoBuf 定义数据结构，然后通过 ProtoBuf 工具生成各种语言版本的数据结构类库，用于操作 ProtoBuf 协议数据。

## protoc 下载安装

Protobuf 的 release 版本，下载可以移步：[Protobuf release](https://github.com/protocolbuffers/protobuf/releases)版本。

如果是 Linux 操作系统下，可以直接下载：protoc-xxx-linux-x86_64.zip。

这个版本包含了 protoc 二进制文件以及与 protobuf 一起分发的一组标准.proto 文件。

进入 bin 文件夹，查看 protoc 的版本信息：

```Shell
# 如果打印出了protoc的版本信息，就表示没有任何问题。
# 可以加到环境变量，方便使用
./protoc --version
```

## protoc 使用

```Shell
protoc --java_out=. test.proto
protoc --go_out=. test.proto
```

go 使用 protoc 的话需要额外安装 protoc-gen-go

```Shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

## 使用 docker 生成代码

```Shell
docker run --rm -v $(pwd):$(pwd) -w $(pwd) rvolosatovs/protoc --go_out=. --go-grpc_out=. --grpc-gateway_out=. --openapiv2_out=. -I=. test.proto
```

## proto 文件解析

```Code
// 定义proto的版本
syntax = "proto3";

// 引入其它proto文件
import "other.proto";
import "other2.proto";

// 定义proto的包名，包名可以避免对message 类型之间的名字冲突，同名的Message可以通过package进行区分。
// 在没有为特定语言定义option xxx_package的时候，它还可以用来生成特定语言的包名，比如Java package, go package。
package foo.bar;

// option可以用在proto的scope中，或者message、enum、service的定义中。
// 可以是Protobuf定义的option，或者自定义的option。
option java_package = "com.example.foo";
// protoc-gen-go 版本大于1.4.0, proto文件需要加上go_package,否则无法生成
option go_package = "main";

// 定义消息
message SearchRequest {
  string query = 1;
}

message SearchResponse {
  string answer = 1;
}

// 定义服务
// 需要与协议缓冲区搭配使用，最直接的 RPC 系统是 gRPC
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

## 数据类型

### 定义消息

消息（message），在 protobuf 中指的就是我们要定义的数据结构。

```Code
syntax = "proto3";

message SearchRequest {
  // 字段类型 字段名称 = 标识号
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

**保留标识号（Reserved）**

如果你想保留一些标识号，留给以后用，可以使用下面语法：

```Code
message Foo {
  reserved 2, 15, 9 to 11; // 保留2，15，9到11这些标识号
}
```

### 基本类型

| .proto Type | <div style="width:280px">Notes</div>                                         | C++ Type | Java Type  | Python Type | Go Type | Ruby Type                      | C# Type    | PHP Type       |
| ----------- | ---------------------------------------------------------------------------- | -------- | ---------- | ----------- | ------- | ------------------------------ | ---------- | -------------- |
| double      |                                                                              | double   | double     | float       | float64 | Float                          | double     | float          |
| float       |                                                                              | float    | float      | float       | float32 | Float                          | float      | float          |
| int32       | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用 sint64 替代 | int32    | int        | int         | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| uint32      | 使用变长编码                                                                 | uint32   | int        | int/long    | uint32  | Fixnum 或者 Bignum（根据需要） | uint       | integer        |
| uint64      | 使用变长编码                                                                 | uint64   | long       | int/long    | uint64  | Bignum                         | ulong      | integer/string |
| sint32      | 使用变长编码，这些编码在负值时比 int32 高效的多                              | int32    | int        | int         | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| sint64      | 使用变长编码，有符号的整型值。编码时比通常的 int64 高效。                    | int64    | long       | int/long    | int64   | Bignum                         | long       | integer/string |
| fixed32     | 总是 4 个字节，如果数值总是比总是比 228 大的话，这个类型会比 uint32 高效。   | uint32   | int        | int         | uint32  | Fixnum 或者 Bignum（根据需要） | uint       | integer        |
| fixed64     | 总是 8 个字节，如果数值总是比总是比 256 大的话，这个类型会比 uint64 高效。   | uint64   | long       | int/long    | uint64  | Bignum                         | ulong      | integer/string |
| sfixed32    | 总是 4 个字节                                                                | int32    | int        | int         | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| sfixed64    | 总是 8 个字节                                                                | int64    | long       | int/long    | int64   | Bignum                         | long       | integer/string |
| bool        |                                                                              | bool     | boolean    | bool        | bool    | TrueClass/FalseClass           | bool       | boolean        |
| string      | 一个字符串必须是 UTF-8 编码或者 7-bit ASCII 编码的文本。                     | string   | String     | str/unicode | string  | String (UTF-8)                 | string     | string         |
| bytes       | 可能包含任意顺序的字节数据。                                                 | string   | ByteString | str         | []byte  | String (ASCII-8BIT)            | ByteString | string         |

### 枚举类型

```Code
syntax = "proto3";//指定版本信息，不指定会报错

enum PhoneType //枚举消息类型，使用enum关键词定义,一个电话类型的枚举类型
{
    MOBILE = 0; //proto3版本中，首成员必须为0，成员不应有相同的值
    HOME = 1;
    WORK = 2;
}

// 定义一个电话消息
message PhoneNumber
{
    string number = 1; // 电话号码字段
    PhoneType type = 2; // 电话类型字段，电话类型使用PhoneType枚举类型
}
```

### 数组类型

在 protobuf 消息中定义数组类型，是通过在字段前面增加 repeated 关键词实现，标记当前字段是一个数组。

```Code
message Msg {
  // 只要使用repeated标记类型定义，就表示数组类型。
  repeated int32 arrays = 1;
}
```

### map 类型

protocol buffers 支持 map 类型定义。

```Code
syntax = "proto3";
message Product
{
    string name = 1; // 商品名
    // 定义一个k/v类型，key是string类型，value也是string类型
    map<string, string> attrs = 2; // 商品属性，键值对
}
```

### 消息嵌套

我们在各种语言开发中类的定义是可以互相嵌套的，也可以使用其他类作为自己的成员属性类型。

在 protobuf 中同样支持消息嵌套，可以在一个消息中嵌套另外一个消息，字段类型可以是另外一个消息类型。

```Code
// 定义Result消息
message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3; // 字符串数组类型
}

// 定义SearchResponse消息
message SearchResponse {
  // 引用上面定义的Result消息类型，作为results字段的类型
  repeated Result results = 1; // repeated关键词标记，说明results字段是一个数组
}
```

## 编码原理

Protocol buffers 核心就是对单个数据的编码（Varint 编码）以及对数据整体的编码（Message Structure 编码）。

Protocol Buffer 利用 `Varint` 原理压缩数据，同时使用`Tag - Value` (`Tag - Length - Value`)的编码结构的实现，减少了分隔符的使用，数据存储更加紧凑。

### Varint 编码

Varint 是一种紧凑的表示数字的方法。它用一个或多个字节来表示一个数字，值越小的数字使用越少的字节数。这能减少用来表示数字的字节数。

Varint 中的每个字节（最后一个字节除外）都设置了最高有效位 most significant bit(MSB)，这一位表示是否还会有更多字节出现。每个字节的低 7 位用于以 7 位组的形式存储数字的二进制补码表示，最低有效组首位。

> **最高位为 1 代表后面 7 位仍然表示数字；最高位为 0，后面 7 位用原码补齐。**

如果用不到 1 个字节，那么最高有效位设为 0 ，如下面这个例子，1 用一个字节就可以表示，所以 msb 为 0.

```C
0000 0001
```

如果需要多个字节表示，msb 就应该设置为 1 。例如 300，如果用 Varint 表示的话：

```C
1010 1100 0000 0010
```

**编码方式：**

1. 将被编码数转换为二进制表示
2. 从低位到高位按照 7 位 一组进行划分
3. 将大端序转为小端序，即以分组为单位进行首尾顺序交换。因为 protobuf 使用是小端序，所以需要转换一下
4. 给每组加上最高有效位(最后一个字节高位补 0，其余各字节高位补 1)组成编码后的数据。
5. 最后转成 10 进制。

比如对数字 123456 进行 varint 编码：

1. 123456 用二进制表示为`1 11100010 01000000`，
2. 每次从低向高取 7 位 变成`111 1000100 1000000`
3. 大端序转为小端序，即交换字节顺序变成`1000000 1000100 111`
4. 然后加上最高有效位(即：最后一个字节高位补 0，其余各字节高位补 1)变成`11000000 11000100 00000111`
5. 最后再转成 10 进制，所以经过 varint 编码后 123456 占用三个字节分别为`192 196 7`。

#### 解码

解码的过程就是将字节依次取出，去掉最高有效位，因为是小端排序所以先解码的字节要放在低位，之后解码出来的二进制位继续放在之前已经解码出来的二进制的高位最后转换为 10 进制数完成 varint 编码的解码过程。

#### 缺点

负数需要 10 个字节显示（因为计算机定义负数的符号位为数字的最高位）。

> 具体是先将负数是转成了 long 类型，再进行 varint 编码，这就是占用 10 个 字节的原因了。

**protobuf 采取的解决方式：使用 sint32/sint64 类型表示负数，通过先采用 Zigzag 编码，将正数、负数和 0 都映射到无符号数，最后再采用 varint 编码。**

> 具体实现 github.com/golang/protobuf

### ZigZag 编码

ZigZag 是将符号数统一映射到无符号号数的一种编码方案，具体映射函数为：

```Code
Zigzag(n) = (n << 1) ^ (n >> 31), n 为 sint32 时

Zigzag(n) = (n << 1) ^ (n >> 63), n 为 sint64 时
```

比如：对于 0 -1 1 -2 2 映射到无符号数 0 1 2 3 4。

| 原始值 | 映射值 |
| ------ | ------ |
| 0      | 0      |
| -1     | 1      |
| 1      | 2      |
| 2      | 3      |
| -2     | 4      |

### Message Structure 编码

message 的每个字段 field 在序列化时，一个 field 对应一个 key-value 对，整个二进制文件就是一连串紧密排列的 key-value 对，key 也称为 tag。

采用这种 key-value 对的结构无需使用分隔符来分割不同的 field。对于可选的 field，如果消息中不存在该 field，那么在最终的 message 中就没有该 field，这些特性都有助于节约消息本身的大小。

key 由 wire type 和 FieldNumber 两部分编码而成，具体地说，`key=(field_number<<3)|wire_type`。

key 的最低 3 个 bit 为 wire type，wire type 类型如下表所示：

| Type | Meaning Used For |
| ---- | ---------------- | -------------------------------------------------------- |
| 0    | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | 64-bit           | fixed64, sfixed64, double                                |
| 2    | Length-delimi    | string, bytes, embedded messages, packed repeated fields |
| 3    | Start group      | Groups (deprecated)                                      |
| 4    | End group        | Groups (deprecated)                                      |
| 5    | 32-bit           | fixed32, sfixed32, float                                 |

由于 key 的第三位最多表示 8 个值，而 wire type 目前的种类是 6 种。由于采用 varint 的编码方式，只剩下 4 位的空闲存放 field_number，因此之前在定义每个字段的标识号的时候建议不要超过 15。

wire type 被如此设计，主要是为了解决一个问题，如何知道接下来 value 部分的长度(字节数)，如果：

- **wire type=0、1、5，编码为 key+数据，只有一个数据，可能占数个字节，数据在编码时自带终止标记；**
- **wire type=2，编码为 key+length+数据，length 指示了数据长度，可能有多个数据，顺序排序。**

需要注意的是：

- **如果出现嵌套 message，直接将嵌套 message 部分的编码接在 length 后即可；**
- **repeated 后面接的字段，如果是个 message，它重复出现多少次，编码时其 key 就会出现几次；如果接的是 proto 定义的字段，且以 packed = true 压缩存储时，只会出现 1 个 key；如果不以压缩方式存储，其 key 也会出现多次。**在 proto3 中，默认以压缩方式进行存储，proto2 中则需要显式地声明。
- **没有压缩 float、double 这些浮点类型，说 Protocol Buffer 压缩数据没有到极限，原因就在这里**

## 参考

https://www.tizi365.com/archives/367.html

https://zhuanlan.zhihu.com/p/478299451

https://learnku.com/articles/55924

https://colobu.com/2019/10/03/protobuf-ultimate-tutorial-in-go/

https://www.lixueduan.com/posts/protobuf/02-encode-core/

https://blog.csdn.net/qq_38410730/article/details/103702827

https://developers.google.com/protocol-buffers/docs/proto3?hl=zh-cn#services

