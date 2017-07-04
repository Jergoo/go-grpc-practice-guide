# 内置Trace

grpc默认提供了客户端和服务端的trace日志，可惜没有提供自定义接口，当前只能查看基本的事件日志和请求日志，对于基本的请求状态查看也是很有帮助的，客户端与服务端基本一致，这里已服务端开启trace为例，修改hello项目的server代码：

server/main.go

```go
package main

import (
	"net"
	"net/http"

	pb "git.vodjk.com/go-grpc/example/proto" // 引入编译生成的包

	"golang.org/x/net/context"
	"golang.org/x/net/trace"  // 引入trace包
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

	// 开启trace
	go startTrace()

	grpclog.Println("Listen on " + Address)
	s.Serve(listen)
}

func startTrace() {
	trace.AuthRequest = func(req *http.Request) (any, sensitive bool) {
		return true, true
	}
	go http.ListenAndServe(":50051", nil)
	grpclog.Println("Trace listen on 50051")
}

```
这里我们开启一个http服务监听50051端口，用来查看grpc请求的trace信息

运行：

```
go run main.go

Listen on 127.0.0.1:50052                                                       
Trace listen on 50051

# 进入client目录执行一次客户端请求     
```


### 服务端事件查看

访问：localhost:50051/debug/events，结果如图：

![](/assets/grpc_trace_events.png)

可以看到服务端注册的服务和服务正常启动的事件信息，默认trace中只有这一个事件


### 请求日志信息查看

访问：localhost:50051/debug/requests，结果如图：

![](/assets/grpc_trace_requests.png)

这里可以显示最近的请求状态，包括请求的服务、参数、耗时、响应，对于简单的状态查看还是很方便的，默认值显示最近10条记录，不可以修改。
