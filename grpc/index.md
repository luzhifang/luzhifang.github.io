# Go 使用 gRPC


## 简介

gRPC 是一个高性能、通用的开源 RPC 框架，其由 Google 主要面向移动应用开发并基于 HTTP/2 协议标准而设计，基于 ProtoBuf(Protocol Buffers) 序列化协议开发，且支持众多开发语言。

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等等。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

## RPC 原理

1. 服务调用方（client）以本地调用方式调用服务；
2. client stub 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
3. client stub 找到服务地址，并将消息发送到服务端；
4. server 端接收到消息；
5. server stub 收到消息后进行解码；
6. server stub 根据解码结果调用本地的服务；
7. 本地服务执行并将结果返回给 server stub；
8. server stub 将返回结果打包成能够进行网络传输的消息体；
9. 按地址将消息发送至调用方；
10. client 端接收到消息；
11. client stub 收到消息并进行解码；
12. 调用方得到最终结果。

## 环境准备

首先需要安装 protoc 和 protoc-gen-go，详见[Protocol Buffers](/protobuf)。

然后安装 protoc-gen-go-grpc。

```Shell
# 安装好之后将 $GOPATH/bin 添加到环境变量中
# export PATH="$PATH:$(go env GOPATH)/bin"
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## helloworld 案例

### 定义服务

创建 helloworld.proto 文件

```Code
syntax = "proto3";

//name 表示生成的go文件所属的包名
option go_package="./;proto";

// 定义包名
package proto;

// 定义Greeter服务
service Greeter {
   // 定义SayHello方法，接受HelloRequest消息， 并返回HelloReply消息
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 定义HelloRequest消息
message HelloRequest {
  string name = 1;
}

// 定义HelloReply消息
message HelloReply {
  string message = 1;
}
```

### 生成代码

```Shell
protoc --go_out=. --go-grpc_out=. helloworld.proto
```

### 实现服务端代码

创建文件:helloworld/server.go

```Go
package main

import (
	"log"
	"net"

	"golang.org/x/net/context"
	// 导入grpc包
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
	// 导入刚才我们生成的代码所在的proto包。
	pb "grpc-demo/proto"
)

// 定义server，用来实现proto文件，里面实现的Greeter服务里面的接口
type server struct {
	pb.UnimplementedGreeterServer
}

// 实现SayHello接口
// 第一个参数是上下文参数，所有接口默认都要必填
// 第二个参数是我们定义的HelloRequest消息
// 返回值是我们定义的HelloReply消息，error返回值也是必须的。
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	// 创建一个HelloReply消息，设置Message字段，然后直接返回。
	return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
	// 监听127.0.0.1:50051地址
	lis, err := net.Listen("tcp", "127.0.0.1:50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// 实例化grpc服务端
	s := grpc.NewServer()

	// 注册Greeter服务
	pb.RegisterGreeterServer(s, &server{})

	// 往grpc服务端注册反射服务
	reflection.Register(s)

	// 启动grpc服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

运行:

```Shell
# 切换到项目根目录，运行命令
go run server.go
```

### 实现客户端代码

创建文件：helloworld/client.go

```Go
package main

import (
	"log"
	"time"

	"golang.org/x/net/context"
	// 导入grpc包
	"google.golang.org/grpc"
	// 导入刚才我们生成的代码所在的proto包。
	pb "grpc-demo/proto"
)

func main() {
	// 连接grpc服务器
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	// 延迟关闭连接
	defer conn.Close()

	// 初始化Greeter服务客户端
	c := pb.NewGreeterClient(conn)

	// 初始化上下文，设置请求超时时间为1秒
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	// 延迟关闭请求会话
	defer cancel()

	// 调用SayHello接口，发送一条消息
	r, err := c.SayHello(ctx, &pb.HelloRequest{Name: "world"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}

	// 打印服务的返回的消息
	log.Printf("Greeting: %s", r.Message)
}
```

运行:

```Shell
# 切换到项目根目录，运行命令
go run client.go
```

## gRPC 的服务类型

gRPC 主要有 4 种请求和响应模式，分别是简单模式(Simple RPC)、服务端流式（Server-side streaming RPC）、客户端流式（Client-side streaming RPC）、和双向流式（Bidirectional streaming RPC）。

- 简单模式(Simple RPC)：客户端发起请求并等待服务端响应。
- 服务端流式（Server-side streaming RPC）：客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流，直到里面没有任何消息。
- 客户端流式（Client-side streaming RPC）：与服务端数据流模式相反，这次是客户端源源不断的向服务端发送数据流，而在发送结束后，由服务端返回一个响应。
- 双向流式（Bidirectional streaming RPC）：双方使用读写流去发送一个消息序列，两个流独立操作，双方可以同时发送和同时接收。

**在服务定义的请求参数前面加 stream 表示客户端流式 RPC，在返回值前面加 stream 表示服务端流式，都加表示双向流式**

```Shell
//双向流式rpc，只要在请求的参数前和响应参数前都添加stream即可
service Stream {
    // 双向流式 rpc ，同时在请求参数前和响应参数前加上 stream
    rpc Conversations(stream StreamRequest) returns(stream StreamResponse){};
}
```

## 参考

https://grpc.io/docs/languages/go/quickstart/

https://doc.oschina.net/grpc?t=58008

https://www.jianshu.com/p/17d02fe0841a

https://zhuanlan.zhihu.com/p/411317961

https://www.liwenzhou.com/posts/Go/rpc/

