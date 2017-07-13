# Hello gRPC

按照惯例，这里也从一个Hello项目开始，本项目定义了一个Hello Service，客户端发送包含字符串名字的请求，服务端返回Hello消息。

流程： 

1. 编写`.proto`描述文件
2. 编译生成`.pb.go`文件
3. 服务端实现约定的接口并提供服务
4. 客户端按照约定调用方法请求服务

项目目录：

```
$GOPATH/src/grpc-go-practice/

example/
|—— hello/
	|—— client/
    	|—— main.go   // 客户端
	|—— server/
    	|—— main.go   // 服务端
|—— proto/
	|—— hello.proto   // proto描述文件
	|—— hello.pb.go   // proto编译后文件
```

编写描述文件：proto/hello.proto

```
syntax = "proto3"; // 指定proto版本

package proto;     // 指定包名

// 定义Hello服务
service Hello {
	// 定义SayHello方法
	rpc SayHello(HelloRequest) returns (HelloReply) {}
}

// HelloRequest 请求结构
message HelloRequest {
	string name = 1;
}

// HelloReply 响应结构
message HelloReply {
    string message = 1;
}
```
hello.proto文件中定义了一个Hello Service，该服务包含一个`SayHello`方法，同时声明了`HelloRequest`和`HelloReply`消息结构用于请求和响应。客户端使用`HelloRequest`参数调用`SayHello`方法请求服务端，服务端响应`HelloReply`消息。protobuf的语法后面会详细介绍，这里先不用过多纠结。


编译生成`.pb.go`文件：

```sh
# 编译hello.proto
protoc -I . --go_out=plugins=grpc:. ./hello.proto
```
生成的`.pb.go`文件，按照`.proto`文件中的说明，包含服务端接口`HelloServer`描述，客户端接口及实现`HelloClient`，及`HelloRequest`、`HelloResponse`结构体，**不要手动编辑该文件**。


实现服务端接口：server/main.go

```go
package main

import (
	"net"

	pb "go-grpc-practice/example/proto" // 引入编译生成的包

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/grpclog"
)

const (
	// Address gRPC服务地址
	Address = "127.0.0.1:50052"
)

// 定义helloService并实现约定的接口
type helloService struct{}

// HelloService ...
var HelloService = helloService{}

func (h helloService) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	resp := new(pb.HelloReply)
	resp.Message = "Hello " + in.Name + "."

	return resp, nil
}

func main() {
	listen, err := net.Listen("tcp", Address)
	if err != nil {
		grpclog.Fatalf("failed to listen: %v", err)
	}

	// 实例化grpc Server
	s := grpc.NewServer()

	// 注册HelloService
	pb.RegisterHelloServer(s, HelloService)

	grpclog.Println("Listen on " + Address)

	s.Serve(listen)
}

```
运行：

```sh
go run main.go

Listen on 50051  //服务端已开启并监听50051端口
```
服务端引入编译后的`proto`包，实现约定的接口，接口描述可以查看`hello.pb.go`文件中的`HelloServer`接口描述。实例化grpc Server并注册HelloService，开始提供服务。


客户端调用：client/main.go

```go
package main

import (
	pb "go-grpc-practice/example/proto" // 引入proto包

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/grpclog"
)

const (
	// Address gRPC服务地址
	Address = "127.0.0.1:50052"
)

func main() {
	// 连接
	conn, err := grpc.Dial(Address, grpc.WithInsecure())

	if err != nil {
		grpclog.Fatalln(err)
	}

	defer conn.Close()

	// 初始化客户端
	c := pb.NewHelloClient(conn)

	// 调用方法
	reqBody := new(pb.HelloRequest)
	reqBody.Name = "gRPC"
	r, err := c.SayHello(context.Background(), reqBody)
	if err != nil {
		grpclog.Fatalln(err)
	}

	grpclog.Println(r.Message)
}
```
运行：

```sh
go run main.go

Hello gRPC     // 接收到服务端响应
```
客户端初始化连接后直接调用声明的方法，即可向服务端发起请求，使用姿势就像调用本地方法一样。如果你收到了"Hello gRPC"的回复，恭喜你已经会使用go-gRPC了。

下节将详细介绍protobuf的语法，并详细解析protobuf声明与编译后golang代码的对应关系。
