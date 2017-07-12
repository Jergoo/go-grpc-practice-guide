# 认证

## TLS认证示例

这里直接扩展hello项目，实现TLS认证机制

首先需要准备证书，在hello目录新建keys目录用于存放证书文件。

### 证书制作

#### 制作私钥 (.key)

```sh
# Key considerations for algorithm "RSA" ≥ 2048-bit
openssl genrsa -out server.key 2048
    
# Key considerations for algorithm "ECDSA" ≥ secp384r1
# List ECDSA the supported curves (openssl ecparam -list_curves)
openssl ecparam -genkey -name secp384r1 -out server.key
```


#### 自签名公钥(x509) (PEM-encodings `.pem`|`.crt`) 

```sh
openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650
```

#### 自定义信息

```
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:XxXx
Locality Name (eg, city) []:XxXx
Organization Name (eg, company) [Internet Widgits Pty Ltd]:XX Co. Ltd
Organizational Unit Name (eg, section) []:Dev
Common Name (e.g. server FQDN or YOUR name) []:server name
Email Address []:xxx@xxx.com
```

### 目录结构

```
$GOPATH/src/grpc-go-practice/

example/
|—— hello-tls/
	|—— client/
    	|—— main.go   // 客户端
	|—— server/
    	|—— main.go   // 服务端
|—— keys/				 // 证书目录
	|—— server.key
	|—— server.pem
|—— proto/
	|—— hello.proto   // proto描述文件
	|—— hello.pb.go   // proto编译后文件
```


### 示例代码

`proto/helloworld.proto`及`proto/hello.pb.go`文件不需要改动

修改服务端代码：server/main.go

```
package main

import (
	"net"

	pb "go-grpc-practice/example/proto"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials" // 引入grpc认证包
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

	// TLS认证
	creds, err := credentials.NewServerTLSFromFile("../../keys/server.pem", "../../keys/server.key")
	if err != nil {
		grpclog.Fatalf("Failed to generate credentials %v", err)
	}

	// 实例化grpc Server, 并开启TLS认证
	s := grpc.NewServer(grpc.Creds(creds))

	// 注册HelloService
	pb.RegisterHelloServer(s, HelloService)

	grpclog.Println("Listen on " + Address + " with TLS")

	s.Serve(listen)
}
```
运行：

```
go run main.go

Listen on 127.0.0.1:50052 with TLS
```
服务端在实例化grpc Server时，可配置多种选项，TLS认证是其中之一


客户端添加TLS认证：client/main.go

```
package main

import (
	pb "go-grpc-practice/example/proto" // 引入proto包

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials" // 引入grpc认证包
	"google.golang.org/grpc/grpclog"
)

const (
	// Address gRPC服务地址
	Address = "127.0.0.1:50052"
)

func main() {
	// TLS连接
	creds, err := credentials.NewClientTLSFromFile("../../keys/server.pem", "server name")
	if err != nil {
		grpclog.Fatalf("Failed to create TLS credentials %v", err)
	}
	conn, err := grpc.Dial(Address, grpc.WithTransportCredentials(creds))

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

```
go run main.go

Hello gRPC
```
客户端添加TLS认证的方式和服务端类似，在创建连接`Dial`时，同样可以配置多种选项，后面的示例中会看到更多的选项。


## Token认证示例

再进一步，继续扩展hello-tls项目，实现TLS + Token认证机制


### 目录结构

```
$GOPATH/src/grpc-go-practice/

example/
|—— hello-token/
	|—— client/
    	|—— main.go   // 客户端
	|—— server/
    	|—— main.go   // 服务端
|—— keys/             // 证书目录
	|—— server.key
	|—— server.pem
|—— proto/
	|—— hello.proto   // proto描述文件
	|—— hello.pb.go   // proto编译后文件
```


### 示例代码

先修改客户端实现：client/main.go

```
package main

import (
	pb "go-grpc-practice/example/proto" // 引入proto包

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials" // 引入grpc认证包
	"google.golang.org/grpc/grpclog"
)

const (
	// Address gRPC服务地址
	Address = "127.0.0.1:50052"

	// OpenTLS 是否开启TLS认证
	OpenTLS = true
)

// customCredential 自定义认证
type customCredential struct{}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "101010",
		"appkey": "i am key",
	}, nil
}

func (c customCredential) RequireTransportSecurity() bool {
	if OpenTLS {
		return true
	}

	return false
}

func main() {
	var err error
	var opts []grpc.DialOption

	if OpenTLS {
		// TLS连接
		creds, err := credentials.NewClientTLSFromFile("../../keys/server.pem", "server name")
		if err != nil {
			grpclog.Fatalf("Failed to create TLS credentials %v", err)
		}
		opts = append(opts, grpc.WithTransportCredentials(creds))
	} else {
		opts = append(opts, grpc.WithInsecure())
	}

	// 使用自定义认证
	opts = append(opts, grpc.WithPerRPCCredentials(new(customCredential)))

	conn, err := grpc.Dial(Address, opts...)

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
这里我们定义了一个`customCredential`结构，并实现了两个方法`GetRequestMetadata`和`RequireTransportSecurity`。这是gRPC提供的自定义认证方式，每次RPC调用都会传输认证信息。`customCredential`其实是实现了`grpc/credential`包内的`PerRPCCredentials`接口。每次调用，token信息会通过请求的metadata传输到服务端。下面具体看一下服务端如何获取metadata中的信息。

修改server/main.go中的SayHello方法：

```
package main

import (
	"fmt"
	"net"

	pb "go-grpc-practice/example/proto"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/credentials" // 引入grpc认证包
	"google.golang.org/grpc/grpclog"
	"google.golang.org/grpc/metadata" // 引入grpc meta包
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
	// 解析metada中的信息并验证
	md, ok := metadata.FromContext(ctx)
	if !ok {
		return nil, grpc.Errorf(codes.Unauthenticated, "无Token认证信息")
	}

	var (
		appid  string
		appkey string
	)

	if val, ok := md["appid"]; ok {
		appid = val[0]
	}

	if val, ok := md["appkey"]; ok {
		appkey = val[0]
	}

	if appid != "101010" || appkey != "i am key" {
		return nil, grpc.Errorf(codes.Unauthenticated, "Token认证信息无效: appid=%s, appkey=%s", appid, appkey)
	}

	resp := new(pb.HelloReply)
	resp.Message = fmt.Sprintf("Hello %s.\nToken info: appid=%s,appkey=%s", in.Name, appid, appkey)

	return resp, nil
}

func main() {
	listen, err := net.Listen("tcp", Address)
	if err != nil {
		grpclog.Fatalf("failed to listen: %v", err)
	}

	// TLS认证
	creds, err := credentials.NewServerTLSFromFile("../../keys/server.pem", "../../keys/server.key")
	if err != nil {
		grpclog.Fatalf("Failed to generate credentials %v", err)
	}

	// 实例化grpc Server, 并开启TLS认证
	s := grpc.NewServer(grpc.Creds(creds))

	// 注册HelloService
	pb.RegisterHelloServer(s, HelloService)

	grpclog.Println("Listen on " + Address + " with TLS + Token")

	s.Serve(listen)
}

```
运行：

```
go run main.go

Listen on 50052 with TLS + Token
```

运行客户端程序 client/main.go：

```
go run main.go

// 认证成功结果
Hello gRPC
Token info: appid=101010,appkey=i am key

// 认证失败结果：
rpc error: code = 16 desc = Token认证信息无效: appID=101010, appKey=i am not key
```

