---
title: gRPC
date: 2022/12/01 21:43:31
tags: 
 - gRPC
categories: 
 - gRPC
---

# 一、使用
## 1. 安装
### 1.1 安装golang中的gRPC库

```shell
go get -u google.golang.org/grpc
```

### 1.2 安装工具

安装编译器最简单的方式是去[protobuf仓库地址](https://link.zhihu.com/?target=https%3A//github.com/protocolbuffers/protobuf/releases)下载预编译好的 protoc 二进制文件，仓库中可以找到每个平台对应的编译器二进制文件，下载解压后配置到环境变量，查看是否成功。

```shell
protoc --version
```

### 1.3 安装插件

安装 protoc 之外还需要安装各个语言对应的编译插件，这里我们用的Go 语言，所以还需要安装一个 Go 语言的编译插件。

```
go install github.com/golang/protobuf/protoc-gen-go
```



## 2. DEMO

gRPC主要有4种请求和响应模式，分别是简单模式(Simple RPC)、服务端流式（Server-side streaming RPC）、客户端流式（Client-side streaming RPC）、和双向流式（Bidirectional streaming RPC）。

- 简单模式(Simple RPC)：客户端发起请求并等待服务端响应。
- 服务端流式（Server-side streaming RPC）：客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流，直到里面没有任何消息。
- 客户端流式（Client-side streaming RPC）：与服务端数据流模式相反，这次是客户端源源不断的向服务端发送数据流，而在发送结束后，由服务端返回一个响应。
- 双向流式（Bidirectional streaming RPC）：双方使用读写流去发送一个消息序列，两个流独立操作，双方可以同时发送和同时接收。

### 2.1 简单模式

本demo项目结构如下:

```text
helloworld/
├── client.go - 客户端代码
├── go.mod  - go模块配置文件
├── proto     - 协议目录
│   ├── helloworld.pb.go - rpc协议go版本代码
│   └── helloworld.proto - rpc协议文件
└── server.go  - rpc服务端代码
```

初始化命令如下：

```text
# 创建项目目录
mkdir helloworld
# 切换到项目目录
cd helloworld
# 创建RPC协议目录
mkdir proto
# 初始化go模块配置，用来管理第三方依赖
go mod init 
```

#### 2.1.1 定义服务

其实就是通过protobuf语法定义语言平台无关的接口。 文件: helloworld/proto/helloworld.proto

```text
syntax = "proto3";

//option go_package = "path;name";
//path 表示生成的go文件的存放地址，会自动生成目录的。
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
  // name字段
  string name = 1;
}

// 定义HelloReply消息
message HelloReply {
  // message字段
  string message = 1;
}
```

#### 2.1.2 **编译命令**

```shell
$ protoc --proto_path=IMPORT_PATH  --go_out=OUT_DIR  --go_opt=paths=source_relative path/to/file.proto
```

这里简单介绍一下 golang 的编译姿势:

- proto_path或者-I ：指定 import 路径，可以指定多个参数，编译时按顺序查找，不指定时默认查找当前目录。

- - proto 文件中也可以引入其他 .proto 文件，这里主要用于指定被引入文件的位置。

- go_out：golang编译支持，指定输出文件路径

- go_opt：指定参数，比如--go_opt=paths=source_relative就是表明生成文件输出使用相对路径。

- path/to/file.proto ：被编译的 .proto 文件放在最后面

上面通过proto定义的接口，没法直接在代码中使用，因此需要通过protoc编译器，将proto协议文件，编译成go语言代码。 在我们的demo中,按如下命令进行编译:

```shell
protoc -I proto/ --go_out=plugins=grpc:proto proto/helloworld.proto
```



#### 2.1.3 **实现服务端代码**

文件:helloworld/server.go

```text
package main

import (
 "log"
 "net"

 "golang.org/x/net/context"
 // 导入grpc包
 "google.golang.org/grpc"
 // 导入刚才我们生成的代码所在的proto包。
  pb "helloworld/proto"
 "google.golang.org/grpc/reflection"
)


// 定义server，用来实现proto文件，里面实现的Greeter服务里面的接口
type server struct{}

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

```text
# 切换到项目根目录，运行命令
go run server.go
```

#### 2.1.4 **客户端代码**

文件：helloworld/client.go

```text
package main

import (
 "log"
 "os"
 "time"

 "golang.org/x/net/context"
 // 导入grpc包
 "google.golang.org/grpc"
 // 导入刚才我们生成的代码所在的proto包。
  pb "helloworld/proto"
)

const (
 defaultName = "world"
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

```text
# 切换到项目根目录，运行命令
go run client.go
```



### 2.2 **服务端流式RPC**

上面的DEMO介绍了简单模式RPC，当数据量大或者需要不断传输数据时候，我们应该使用流式RPC，它允许我们边处理边传输数据。本节先介绍服务端流式RPC。

**服务端流式RPC**：客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流，直到里面没有任何消息。

**情景模拟：实时获取股票走势。**

- 客户端要获取某原油股的实时走势，客户端发送一个请求
- 服务端实时返回该股票的走势

#### 2.2.1 **新建proto文件**

新建server_stream.proto文件

```text
// 定义发送请求信息
message SimpleRequest{
    // 定义发送的参数，采用驼峰命名方式，小写加下划线，如：student_name
    // 请求参数
    string data = 1;
}

// 定义流式响应信息
message StreamResponse{
    // 流式响应数据
    string stream_value = 1;
}

//服务端流式rpc，只要在响应数据前添加stream即可
// 定义我们的服务（可定义多个服务,每个服务可定义多个接口）
service StreamServer{
    // 服务端流式rpc，在响应数据前添加stream
    rpc ListValue(SimpleRequest)returns(stream StreamResponse){};
}
```

编译参考demo部分的编译命令

#### 2.2.2 **创建server端**

定义我们的服务，并实现ListValue方法

```text
// SimpleService 定义我们的服务
type StreamService struct{}
// ListValue 实现ListValue方法
func (s *StreamService) ListValue(req *pb.SimpleRequest, srv pb.StreamServer_ListValueServer) error {
 for n := 0; n < 5; n++ {
  // 向流中发送消息， 默认每次send送消息最大长度为`math.MaxInt32`bytes
  err := srv.Send(&pb.StreamResponse{
   StreamValue: req.Data + strconv.Itoa(n),
  })
  if err != nil {
   return err
  }
 }
 return nil
}
```

启动gRPC服务器

```text
const (
 // Address 监听地址
 Address string = ":8000"
 // Network 网络通信协议
 Network string = "tcp"
)

func main() {
 // 监听本地端口
 listener, err := net.Listen(Network, Address)
 if err != nil {
  log.Fatalf("net.Listen err: %v", err)
 }
 log.Println(Address + " net.Listing...")
 // 新建gRPC服务器实例
 // 默认单次接收最大消息长度为`1024*1024*4`bytes(4M)，单次发送消息最大长度为`math.MaxInt32`bytes
 // grpcServer := grpc.NewServer(grpc.MaxRecvMsgSize(1024*1024*4), grpc.MaxSendMsgSize(math.MaxInt32))
 grpcServer := grpc.NewServer()
 // 在gRPC服务器注册我们的服务
 pb.RegisterStreamServerServer(grpcServer, &StreamService{})

 //用服务器 Serve() 方法以及我们的端口信息区实现阻塞等待，直到进程被杀死或者 Stop() 被调用
 err = grpcServer.Serve(listener)
 if err != nil {
  log.Fatalf("grpcServer.Serve err: %v", err)
 }
}
```

运行

```she
go run server.go
```

#### 2.2.3 **创建client端**

创建调用服务端ListValue方法

```go
// listValue 调用服务端的ListValue方法
func listValue() {
 // 创建发送结构体
 req := pb.SimpleRequest{
  Data: "stream server grpc ",
 }
 // 调用我们的服务(ListValue方法)
 stream, err := grpcClient.ListValue(context.Background(), &req)
 if err != nil {
  log.Fatalf("Call ListStr err: %v", err)
 }
 for {
  //Recv() 方法接收服务端消息，默认每次Recv()最大消息长度为`1024*1024*4`bytes(4M)
  res, err := stream.Recv()
  // 判断消息流是否已经结束
  if err == io.EOF {
   break
  }
  if err != nil {
   log.Fatalf("ListStr get stream err: %v", err)
  }
  // 打印返回值
  log.Println(res.StreamValue)
 }
}
```

启动gRPC客户端

```go
// Address 连接地址
const Address string = ":8000"

var grpcClient pb.StreamServerClient

func main() {
 // 连接服务器
 conn, err := grpc.Dial(Address, grpc.WithInsecure())
 if err != nil {
  log.Fatalf("net.Connect err: %v", err)
 }
 defer conn.Close()

 // 建立gRPC连接
 grpcClient = pb.NewStreamServerClient(conn)
 listValue()
}
```

运行客户端

```text
go run client.go
stream server grpc 0
stream server grpc 1
stream server grpc 2
stream server grpc 3
stream server grpc 4
```

> 客户端不断从服务端获取数据



### 2.3 **客户端流式RPC**

上一节介绍了服务端流式RPC，客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流的数据。本节将介绍客户端流式RPC。

**客户端流式RPC**：与服务端流式RPC相反，客户端不断的向服务端发送数据流，而在发送结束后，由服务端返回一个响应。

**情景模拟**：客户端大量数据上传到服务端。

#### 2.3.1 **新建proto文件**

新建client_stream.proto文件

```text
// 定义流式请求信息
message StreamRequest{
    //流式请求参数
    string stream_data = 1;
}

// 定义响应信息
message SimpleResponse{
    //响应码
    int32 code = 1;
    //响应值
    string value = 2;
}


//客户端流式rpc，只要在请求的参数前添加stream即可
service StreamClient{
    // 客户端流式rpc，在请求的参数前添加stream
    rpc RouteList (stream StreamRequest) returns (SimpleResponse){};
}
```

参照demo进行编译。

#### 2.3.2 **创建Server端**

定义我们的服务，并实现RouteList方法

```go
// SimpleService 定义我们的服务
type SimpleService struct{}
// RouteList 实现RouteList方法
func (s *SimpleService) RouteList(srv pb.StreamClient_RouteListServer) error {
 for {
  //从流中获取消息
  res, err := srv.Recv()
  if err == io.EOF {
   //发送结果，并关闭
   return srv.SendAndClose(&pb.SimpleResponse{Value: "ok"})
  }
  if err != nil {
   return err
  }
  log.Println(res.StreamData)
 }
}
```

启动gRPC服务器

```text
const (
 // Address 监听地址
 Address string = ":8000"
 // Network 网络通信协议
 Network string = "tcp"
)

func main() {
 // 监听本地端口
 listener, err := net.Listen(Network, Address)
 if err != nil {
  log.Fatalf("net.Listen err: %v", err)
 }
 log.Println(Address + " net.Listing...")
 // 新建gRPC服务器实例
 grpcServer := grpc.NewServer()
 // 在gRPC服务器注册我们的服务
 pb.RegisterStreamClientServer(grpcServer, &SimpleService{})

 //用服务器 Serve() 方法以及我们的端口信息区实现阻塞等待，直到进程被杀死或者 Stop() 被调用
 err = grpcServer.Serve(listener)
 if err != nil {
  log.Fatalf("grpcServer.Serve err: %v", err)
 }
}
```

运行服务端

```text
go run server.go
```

#### 2.3.3 **创建客户端**

创建调用服务端RouteList方法

```go
// routeList 调用服务端RouteList方法
func routeList() {
 //调用服务端RouteList方法，获流
 stream, err := streamClient.RouteList(context.Background())
 if err != nil {
  log.Fatalf("Upload list err: %v", err)
 }
 for n := 0; n < 5; n++ {
  //向流中发送消息
  err := stream.Send(&pb.StreamRequest{StreamData: "stream client rpc " + strconv.Itoa(n)})
  if err != nil {
   log.Fatalf("stream request err: %v", err)
  }
 }
 //关闭流并获取返回的消息
 res, err := stream.CloseAndRecv()
 if err != nil {
  log.Fatalf("RouteList get response err: %v", err)
 }
 log.Println(res)
}
```

启动gRPC客户端

```go
// Address 连接地址
const Address string = ":8000"

var streamClient pb.StreamClientClient

func main() {
 // 连接服务器
 conn, err := grpc.Dial(Address, grpc.WithInsecure())
 if err != nil {
  log.Fatalf("net.Connect err: %v", err)
 }
 defer conn.Close()

 // 建立gRPC连接
 streamClient = pb.NewStreamClientClient(conn)
 routeList()
}
```

运行客户端

```text
go run client.go
```

服务端不断从客户端获取到数据

```text
stream client rpc 0
stream client rpc 1
stream client rpc 2
stream client rpc 3
stream client rpc 4
```

### 2.4 **双向流式RPC**

上一节介绍了客户端流式RPC，客户端不断的向服务端发送数据流，在发送结束或流关闭后，由服务端返回一个响应。本节将介绍双向流式RPC。

**双向流式RPC**：客户端和服务端双方使用读写流去发送一个消息序列，两个流独立操作，双方可以同时发送和同时接收。

**情景模拟**：双方对话（可以一问一答、一问多答、多问一答，形式灵活）。

#### 2.4.1 **新建proto文件**

新建both_stream.proto文件

```text
// 定义流式请求信息
message StreamRequest{
    //流请求参数
    string question = 1;
}

// 定义流式响应信息
message StreamResponse{
    //流响应数据
    string answer = 1;
}


//双向流式rpc，只要在请求的参数前和响应参数前都添加stream即可
service Stream{
    // 双向流式rpc，同时在请求参数前和响应参数前加上stream
    rpc Conversations(stream StreamRequest) returns(stream StreamResponse){};
}
```

编译参照demo部分编译即可。

#### 2.4.2 **创建Server端**

1. 定义我们的服务，并实现RouteList方法 这里简单实现对话中一问一答的形式

```text
// StreamService 定义我们的服务
type StreamService struct{}
// Conversations 实现Conversations方法
func (s *StreamService) Conversations(srv pb.Stream_ConversationsServer) error {
 n := 1
 for {
  req, err := srv.Recv()
  if err == io.EOF {
   return nil
  }
  if err != nil {
   return err
  }
  err = srv.Send(&pb.StreamResponse{
   Answer: "from stream server answer: the " + strconv.Itoa(n) + " question is " + req.Question,
  })
  if err != nil {
   return err
  }
  n++
  log.Printf("from stream client question: %s", req.Question)
 }
}
```

启动gRPC服务器

```text
const (
 // Address 监听地址
 Address string = ":8000"
 // Network 网络通信协议
 Network string = "tcp"
)

func main() {
 // 监听本地端口
 listener, err := net.Listen(Network, Address)
 if err != nil {
  log.Fatalf("net.Listen err: %v", err)
 }
 log.Println(Address + " net.Listing...")
 // 新建gRPC服务器实例
 grpcServer := grpc.NewServer()
 // 在gRPC服务器注册我们的服务
 pb.RegisterStreamServer(grpcServer, &StreamService{})

 //用服务器 Serve() 方法以及我们的端口信息区实现阻塞等待，直到进程被杀死或者 Stop() 被调用
 err = grpcServer.Serve(listener)
 if err != nil {
  log.Fatalf("grpcServer.Serve err: %v", err)
 }
}
```

运行服务端

```text
go run server.go
:8000 net.Listing...
```

#### 2.4.3 **创建Client端**

创建调用服务端Conversations方法

```text
// conversations 调用服务端的Conversations方法
func conversations() {
 //调用服务端的Conversations方法，获取流
 stream, err := streamClient.Conversations(context.Background())
 if err != nil {
  log.Fatalf("get conversations stream err: %v", err)
 }
 for n := 0; n < 5; n++ {
  err := stream.Send(&pb.StreamRequest{Question: "stream client rpc " + strconv.Itoa(n)})
  if err != nil {
   log.Fatalf("stream request err: %v", err)
  }
  res, err := stream.Recv()
  if err == io.EOF {
   break
  }
  if err != nil {
   log.Fatalf("Conversations get stream err: %v", err)
  }
  // 打印返回值
  log.Println(res.Answer)
 }
 //最后关闭流
 err = stream.CloseSend()
 if err != nil {
  log.Fatalf("Conversations close stream err: %v", err)
 }
}
```

启动gRPC客户端

```text
// Address 连接地址
const Address string = ":8000"

var streamClient pb.StreamClient

func main() {
 // 连接服务器
 conn, err := grpc.Dial(Address, grpc.WithInsecure())
 if err != nil {
  log.Fatalf("net.Connect err: %v", err)
 }
 defer conn.Close()

 // 建立gRPC连接
 streamClient = pb.NewStreamClient(conn)
 conversations()
}
```

运行客户端，获取到服务端的应答

```text
go run client.go
from stream server answer: the 1 question is stream client rpc 0
from stream server answer: the 2 question is stream client rpc 1
from stream server answer: the 3 question is stream client rpc 2
from stream server answer: the 4 question is stream client rpc 3
from stream server answer: the 5 question is stream client rpc 4
```

服务端获取到来自客户端的提问

```text
from stream client question: stream client rpc 0
from stream client question: stream client rpc 1
from stream client question: stream client rpc 2
from stream client question: stream client rpc 3
from stream client question: stream client rpc 4#
```



# 二、分析

## 1. server端启动流程

### 1.1 构建本地监听端口

```go
lis, err := net.Listen("tcp", "127.0.0.1:8001")
if err != nil {
    log.Fatalf("failed to listen: %v", err)
}
```

### 1.2 创建server实例

```go
// 实例化grpc服务端
s := grpc.NewServer()
```

```go
// NewServer creates a gRPC server which has no service registered and has not
// started to accept requests yet.
// 创建一个新的server，该server还没有注册服务，并且没有接受请求
func NewServer(opt ...ServerOption) *Server {
    //把默认配置放到入参中
   opts := defaultServerOptions
   for _, o := range extraServerOptions {
      o.apply(&opts)
   }
   for _, o := range opt {
      o.apply(&opts)
   }
    // 构造Server实例
   s := &Server{
      lis:      make(map[net.Listener]bool),
      opts:     opts,
      conns:    make(map[string]map[transport.ServerTransport]bool),
      services: make(map[string]*serviceInfo),
      quit:     grpcsync.NewEvent(),
      done:     grpcsync.NewEvent(),
      czData:   new(channelzData),
   }
	//chains all unary server interceptors into one.
   chainUnaryServerInterceptors(s)
    //chains all stream server interceptors into one.
   chainStreamServerInterceptors(s)
   s.cv = sync.NewCond(&s.mu)
   // 判断是否开启链路追踪
   if EnableTracing {
      _, file, line, _ := runtime.Caller(1)
      s.events = trace.NewEventLog("grpc.Server", fmt.Sprintf("%s:%d", file, line))
   }

   if s.opts.numServerWorkers > 0 {
      s.initServerWorkers()
   }

   s.channelzID = channelz.RegisterServer(&channelzServer{s}, "")
   channelz.Info(logger, s.channelzID, "Server created")
   return s
}
```

### 1.3 注册服务

```go
// 注册Greeter服务
pb.RegisterGreeterServer(s, &server{})

func RegisterGreeterServer(s *grpc.Server, srv GreeterServer) {
	s.RegisterService(&_Greeter_serviceDesc, srv)
}

// RegisterService registers a service and its implementation to the gRPC
// server. It is called from the IDL generated code. This must be called before
// invoking Serve. If ss is non-nil (for legacy code), its type is checked to
// ensure it implements sd.HandlerType.
func (s *Server) RegisterService(sd *ServiceDesc, ss interface{}) {
	if ss != nil {
		ht := reflect.TypeOf(sd.HandlerType).Elem()
		st := reflect.TypeOf(ss)
		if !st.Implements(ht) {
			logger.Fatalf("grpc: Server.RegisterService found the handler of type %v that does not satisfy %v", st, ht)
		}
	}
	s.register(sd, ss)
}

func (s *Server) register(sd *ServiceDesc, ss interface{}) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.printf("RegisterService(%q)", sd.ServiceName)
	if s.serve {
		logger.Fatalf("grpc: Server.RegisterService after Server.Serve for %q", sd.ServiceName)
	}
	if _, ok := s.services[sd.ServiceName]; ok {
		logger.Fatalf("grpc: Server.RegisterService found duplicate service registration for %q", sd.ServiceName)
	}
	info := &serviceInfo{
		serviceImpl: ss,
		methods:     make(map[string]*MethodDesc),
		streams:     make(map[string]*StreamDesc),
		mdata:       sd.Metadata,
	}
	for i := range sd.Methods {
		d := &sd.Methods[i]
		info.methods[d.MethodName] = d
	}
	for i := range sd.Streams {
		d := &sd.Streams[i]
		info.streams[d.StreamName] = d
	}
	s.services[sd.ServiceName] = info
}
```

### 1.4 注册反射服务

```go
// 往grpc服务端注册反射服务
reflection.Register(s)

// Register registers the server reflection service on the given gRPC server.
func Register(s GRPCServer) {
	svr := NewServer(ServerOptions{Services: s})
	v1alphagrpc.RegisterServerReflectionServer(s, svr)
}
```

### 1.5 启动grpc服务

```go
// 启动grpc服务
if err := s.Serve(lis); err != nil {
   log.Fatalf("failed to serve: %v", err)
}

// Serve accepts incoming connections on the listener lis, creating a new
// ServerTransport and service goroutine for each. The service goroutines
// read gRPC requests and then call the registered handlers to reply to them.
// Serve returns when lis.Accept fails with fatal errors.  lis will be closed when
// this method returns.
// Serve will return a non-nil error unless Stop or GracefulStop is called.
func (s *Server) Serve(lis net.Listener) error {
	s.mu.Lock()
	s.printf("serving")
	s.serve = true
	if s.lis == nil {
		// Serve called after Stop or GracefulStop.
		s.mu.Unlock()
		lis.Close()
		return ErrServerStopped
	}

	s.serveWG.Add(1)
	defer func() {
		s.serveWG.Done()
		if s.quit.HasFired() {
			// Stop or GracefulStop called; block until done and return nil.
			<-s.done.Done()
		}
	}()

	ls := &listenSocket{Listener: lis}
	s.lis[ls] = true

	defer func() {
		s.mu.Lock()
		if s.lis != nil && s.lis[ls] {
			ls.Close()
			delete(s.lis, ls)
		}
		s.mu.Unlock()
	}()

	var err error
	ls.channelzID, err = channelz.RegisterListenSocket(ls, s.channelzID, lis.Addr().String())
	if err != nil {
		s.mu.Unlock()
		return err
	}
	s.mu.Unlock()
	channelz.Info(logger, ls.channelzID, "ListenSocket created")

	var tempDelay time.Duration // how long to sleep on accept failure
	for {
		rawConn, err := lis.Accept()
		if err != nil {
			if ne, ok := err.(interface {
				Temporary() bool
			}); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				s.mu.Lock()
				s.printf("Accept error: %v; retrying in %v", err, tempDelay)
				s.mu.Unlock()
				timer := time.NewTimer(tempDelay)
				select {
				case <-timer.C:
				case <-s.quit.Done():
					timer.Stop()
					return nil
				}
				continue
			}
			s.mu.Lock()
			s.printf("done serving; Accept = %v", err)
			s.mu.Unlock()

			if s.quit.HasFired() {
				return nil
			}
			return err
		}
		tempDelay = 0
		// Start a new goroutine to deal with rawConn so we don't stall this Accept
		// loop goroutine.
		//
		// Make sure we account for the goroutine so GracefulStop doesn't nil out
		// s.conns before this conn can be added.
		s.serveWG.Add(1)
		go func() {
			s.handleRawConn(lis.Addr().String(), rawConn)
			s.serveWG.Done()
		}()
	}
}
```

## 2. keepalive

#### 2.1 客户端keepalive

在gRPC中，会在新建Http2Client的时候，会启动一个goroutine来处理keepalive。

```go
// newHTTP2Client constructs a connected ClientTransport to addr based on HTTP2
// and starts to receive messages on it. Non-nil error returns if construction
// fails.
func newHTTP2Client(connectCtx, ctx context.Context, addr resolver.Address, opts ConnectOptions, onPrefaceReceipt func(), onGoAway func(GoAwayReason), onClose func()) (_ *http2Client, err error) {
    ...
	if t.keepaliveEnabled {
		t.kpDormancyCond = sync.NewCond(&t.mu)
		go t.keepalive()
    }
    ...
}
```

接下来，看下 [`keepalive` 方法](https://github.com/grpc/grpc-go/blob/master/internal/transport/http2_client.go#L1350) 的实现：

```go
func (t *http2Client) keepalive() {
	p := &ping{data: [8]byte{}} //ping 的内容
	timer := time.NewTimer(t.kp.Time) // 启动一个定时器, 触发时间为配置的 Time 值
	//for loop
	for {
		select {
		// 定时器触发
		case <-timer.C:
			if atomic.CompareAndSwapUint32(&t.activity, 1, 0) {
				timer.Reset(t.kp.Time)
				continue
			}
			// Check if keepalive should go dormant.
			t.mu.Lock()
			if len(t.activeStreams) < 1 && !t.kp.PermitWithoutStream {
				// Make awakenKeepalive writable.
				<-t.awakenKeepalive
				t.mu.Unlock()
				select {
				case <-t.awakenKeepalive:
					// If the control gets here a ping has been sent
					// need to reset the timer with keepalive.Timeout.
				case <-t.ctx.Done():
					return
				}
			} else {
				t.mu.Unlock()
				if channelz.IsOn() {
					atomic.AddInt64(&t.czData.kpCount, 1)
				}
				// Send ping.
				t.controlBuf.put(p)
			}

			// By the time control gets here a ping has been sent one way or the other.
			timer.Reset(t.kp.Timeout)
			select {
			case <-timer.C:
				if atomic.CompareAndSwapUint32(&t.activity, 1, 0) {
					timer.Reset(t.kp.Time)
					continue
				}
				t.Close()
				return
			case <-t.ctx.Done():
				if !timer.Stop() {
					<-timer.C
				}
				return
			}
		// 上层通知 context 结束
		case <-t.ctx.Done():
			if !timer.Stop() {
				// 返回 false，表示 timer 未被销毁
				<-timer.C
			}
			return
		}
	}
```

从客户端的 `keepalive` 实现中梳理下执行逻辑：

1. 填充 `ping` 包内容, 为 `[8]byte{}`，创建定时器, 触发时间为用户配置中的 `Time`
2. 循环处理，select 的两大分支，一为定时器触发后执行的逻辑，另一分支为 `t.ctx.Done()`，即 `keepalive` 的上层应用调用了 `cancel` 结束 context 子树
3. 核心逻辑在定时器触发的过程中

#### 2.2 服务端keepalive

gRPC 的服务端主要有两块逻辑：

1. 接收并相应客户端的 ping 包
2. 单独启动 goroutine 探测客户端是否存活

gRPC 服务端提供 keepalive 配置，分为两部分 `keepalive.EnforcementPolicy` 和 `keepalive.ServerParameters`，如下：

```go
var kaep = keepalive.EnforcementPolicy{
	MinTime:             5 * time.Second, // If a client pings more than once every 5 seconds, terminate the connection
	PermitWithoutStream: true,            // Allow pings even when there are no active streams
}

var kasp = keepalive.ServerParameters{
	MaxConnectionIdle:     15 * time.Second, // If a client is idle for 15 seconds, send a GOAWAY
	MaxConnectionAge:      30 * time.Second, // If any connection is alive for more than 30 seconds, send a GOAWAY
	MaxConnectionAgeGrace: 5 * time.Second,  // Allow 5 seconds for pending RPCs to complete before forcibly closing connections
	Time:                  5 * time.Second,  // Ping the client if it is idle for 5 seconds to ensure the connection is still active
	Timeout:               1 * time.Second,  // Wait 1 second for the ping ack before assuming the connection is dead
}

func main(){
	...
	s := grpc.NewServer(grpc.KeepaliveEnforcementPolicy(kaep), grpc.KeepaliveParams(kasp))
	...
}
```

`keepalive.EnforcementPolicy`：

- `MinTime`：如果客户端两次 ping 的间隔小于 `5s`，则关闭连接
- `PermitWithoutStream`： 即使没有 active stream, 也允许 ping

`keepalive.ServerParameters`：

- `MaxConnectionIdle`：如果一个 client 空闲超过 `15s`, 发送一个 GOAWAY, 为了防止同一时间发送大量 GOAWAY, 会在 `15s` 时间间隔上下浮动 `15*10%`, 即 `15+1.5` 或者 `15-1.5`
- `MaxConnectionAge`：如果任意连接存活时间超过 `30s`, 发送一个 GOAWAY
- `MaxConnectionAgeGrace`：在强制关闭连接之间, 允许有 `5s` 的时间完成 pending 的 rpc 请求
- `Time`： 如果一个 client 空闲超过 `5s`, 则发送一个 ping 请求
- `Timeout`： 如果 ping 请求 `1s` 内未收到回复, 则认为该连接已断开



服务端处理客户端的 `ping` 包的 response 的逻辑在 [`handlePing` 方法](https://github.com/grpc/grpc-go/blob/master/internal/transport/http2_server.go#L693) 中。
`handlePing` 方法会判断是否违反两条 policy, 如果违反则将 `pingStrikes++`, 当违反次数大于 `maxPingStrikes(2)` 时, 打印一条错误日志并且发送一个 goAway 包，断开这个连接，具体实现如下：

```go
func (t *http2Server) handlePing(f *http2.PingFrame) {
	if f.IsAck() {
		if f.Data == goAwayPing.data && t.drainChan != nil {
			close(t.drainChan)
			return
		}
		// Maybe it's a BDP ping.
		if t.bdpEst != nil {
			t.bdpEst.calculate(f.Data)
		}
		return
	}
	pingAck := &ping{ack: true}
	copy(pingAck.data[:], f.Data[:])
	t.controlBuf.put(pingAck)

	now := time.Now()
	defer func() {
		t.lastPingAt = now
	}()
	// A reset ping strikes means that we don't need to check for policy
	// violation for this ping and the pingStrikes counter should be set
	// to 0.
	if atomic.CompareAndSwapUint32(&t.resetPingStrikes, 1, 0) {
		t.pingStrikes = 0
		return
	}
	t.mu.Lock()
	ns := len(t.activeStreams)
	t.mu.Unlock()
	if ns < 1 && !t.kep.PermitWithoutStream {
		// Keepalive shouldn't be active thus, this new ping should
		// have come after at least defaultPingTimeout.
		if t.lastPingAt.Add(defaultPingTimeout).After(now) {
			t.pingStrikes++
		}
	} else {
		// Check if keepalive policy is respected.
		if t.lastPingAt.Add(t.kep.MinTime).After(now) {
			t.pingStrikes++
		}
	}

	if t.pingStrikes > maxPingStrikes {
		// Send goaway and close the connection.
		if logger.V(logLevel) {
			logger.Errorf("transport: Got too many pings from the client, closing the connection.")
		}
		t.controlBuf.put(&goAway{code: http2.ErrCodeEnhanceYourCalm, debugData: []byte("too_many_pings"), closeConn: true})
	}
}
```



注意，对 `pingStrikes` 累加的逻辑：

- `t.lastPingAt.Add(defaultPingTimeout).After(now)`：

- `t.lastPingAt.Add(t.kep.MinTime).After(now)`：

  

gRPC 服务端新建一个 HTTP2 server 的时候会启动一个单独的 goroutine 处理 keepalive 逻辑，[`newHTTP2Server` 方法](https://github.com/grpc/grpc-go/blob/master/internal/transport/http2_server.go#L129)：

```go
func newHTTP2Server(conn net.Conn, config *ServerConfig) (_ ServerTransport, err error) {
	...
	go t.keepalive()
	...
}
```

简单分析下 `keepalive` 的实现，核心逻辑是启动 `3` 个定时器，分别为 `maxIdle`、`maxAge` 和 `keepAlive`，然后在 `for select` 中处理相关定时器触发事件：

- `maxIdle` 逻辑： 判断 client 空闲时间是否超出配置的时间, 如果超时, 则调用 `t.drain`, 该方法会发送一个 GOAWAY 包
- `maxAge` 逻辑： 触发之后首先调用 `t.drain` 发送 GOAWAY 包, 接着重置定时器, 时间设置为 `MaxConnectionAgeGrace`, 再次触发后调用 `t.Close()` 直接关闭（有些 graceful 的意味）
- `keepalive` 逻辑： 首先判断 activity 是否为 `1`, 如果不是则置 `pingSent` 为 `true`, 并且发送 ping 包, 接着重置定时器时间为 `Timeout`, 再次触发后如果 activity 不为 1（即未收到 ping 的回复） 并且 `pingSent` 为 `true`, 则调用 `t.Close()` 关闭连接

```
func (t *http2Server) keepalive() {
	p := &ping{}
	var pingSent bool
	maxIdle := time.NewTimer(t.kp.MaxConnectionIdle)
	maxAge := time.NewTimer(t.kp.MaxConnectionAge)
	keepalive := time.NewTimer(t.kp.Time)
	// NOTE: All exit paths of this function should reset their
	// respective timers. A failure to do so will cause the
	// following clean-up to deadlock and eventually leak.
	defer func() {
		// 退出前，完成定时器的回收工作
		if !maxIdle.Stop() {
			<-maxIdle.C
		}
		if !maxAge.Stop() {
			<-maxAge.C
		}
		if !keepalive.Stop() {
			<-keepalive.C
		}
	}()
	for {
		select {
		case <-maxIdle.C:
			t.mu.Lock()
			idle := t.idle
			if idle.IsZero() { // The connection is non-idle.
				t.mu.Unlock()
				maxIdle.Reset(t.kp.MaxConnectionIdle)
				continue
			}
			val := t.kp.MaxConnectionIdle - time.Since(idle)
			t.mu.Unlock()
			if val <= 0 {
				// The connection has been idle for a duration of keepalive.MaxConnectionIdle or more.
				// Gracefully close the connection.
				t.drain(http2.ErrCodeNo, []byte{})
				// Resetting the timer so that the clean-up doesn't deadlock.
				maxIdle.Reset(infinity)
				return
			}
			maxIdle.Reset(val)
		case <-maxAge.C:
			t.drain(http2.ErrCodeNo, []byte{})
			maxAge.Reset(t.kp.MaxConnectionAgeGrace)
			select {
			case <-maxAge.C:
				// Close the connection after grace period.
				t.Close()
				// Resetting the timer so that the clean-up doesn't deadlock.
				maxAge.Reset(infinity)
			case <-t.ctx.Done():
			}
			return
		case <-keepalive.C:
			if atomic.CompareAndSwapUint32(&t.activity, 1, 0) {
				pingSent = false
				keepalive.Reset(t.kp.Time)
				continue
			}
			if pingSent {
				t.Close()
				// Resetting the timer so that the clean-up doesn't deadlock.
				keepalive.Reset(infinity)
				return
			}
			pingSent = true
			if channelz.IsOn() {
				atomic.AddInt64(&t.czData.kpCount, 1)
			}
			t.controlBuf.put(p)
			keepalive.Reset(t.kp.Timeout)
		case <-t.ctx.Done():
			return
		}
	}
}
```



# 三、通信报文格式

Protocol Buffers 是一种与语言、平台无关，可扩展的序列化结构化数据的方法，常用于通信协议，数据存储等等。相较于 JSON、XML，它更小、更快、更简单，因此也更受开发人员的青眯。

# 四、拦截器

在 gRPC 调用过程中，我们可以拦截 RPC 的执行，在 RPC 服务执行前或执行后运行一些自定义逻辑，这在某些场景下很有用，例如身份验证、日志等，我们可以在 RPC 服务执行前检查调用方的身份信息，若未通过验证，则拒绝执行，也可以在执行前后记录下详细的请求响应信息到日志。这种拦截机制与 Gin 中的中间件技术类似，在 gRPC 中被称为 **拦截器**，它是 gRPC 核心扩展机制之一

拦截器不止可以作用在服务端上，客户端同样可以拦截，在请求发出之前和收到响应之后执行一些自定义逻辑，根据拦截的 RPC 类型，可分为 **一元拦截器** 和 **流拦截器**。

## 1. 服务端拦截器

在 gRPC 服务端，可以插入一个或多个拦截器，收到的请求按注册顺序通过各个拦截器，返回响应时则倒序通过。

### 1.1 一元拦截器

通过以下步骤实现一元拦截器：

- 定义一元拦截器方法：

```go
// 函数名无特殊要求，参数需一致
// req包含请求的所有信息，info包含一元RPC服务的所有信息
func orderUnaryServerInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
	handler grpc.UnaryHandler) (interface{}, error) {
        // 前置处理逻辑
        log.Printf("[unary interceptor request] %s", info.FullMethod)
        // 完成RPC服务的正常执行
        m, err := handler(ctx, req)
        // 后置处理逻辑
        log.Printf("[unary interceptor resonse] %s", m)
        // 返回响应
        return m, err
}
```

- 注册定义的一元拦截器

```go
func main() {
    ...
	// 创建gRPC服务器实例的时候注册拦截器
    // NewServer 可传入多个拦截器
	s := grpc.NewServer(grpc.UnaryInterceptor(orderUnaryServerInterceptor))
    ...
}
```

### 1.2 流拦截器

流拦截器包括前置处理阶段和流操作阶段，前置处理阶段可以在流 RPC 进入具体服务实现之前进行拦截，而在流操作阶段，可以对流中的每一条消息进行拦截，通过以下步骤实现流拦截器：

- 自定义一个嵌入grpc.ServerStream的包装器

```go
type wrappedStream struct {
	grpc.ServerStream
}
```

- 实现包装器的 RecvMsg 和 SendMsg 方法

```go
// 自定义RecvMsg和SendMsg方法实现对每一个流消息的拦截
func (w *wrappedStream) RecvMsg(m interface{}) error {
	log.Printf("[stream interceptor recv] type: %T", m)
	return w.ServerStream.RecvMsg(m)
}
func (w *wrappedStream) SendMsg(m interface{}) error {
	log.Printf("[stream interceptor send] %s", m)
	return w.ServerStream.SendMsg(m)
}
```

- 实现流拦截器

```go
func orderServerStreamInterceptor(srv interface{}, ss grpc.ServerStream,
	info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
    // 前置处理阶段
	log.Printf("[stream interceptor request] %s", info.FullMethod)
	// 使用自定义包装器处理流
	err := handler(srv, &wrappedStream{ss})
	if err != nil {
		log.Printf("[stream Intercept error] %v", err)
	}
	return err
}
```

- 注册流拦截器

```go
func main() {
    ...
	s := grpc.NewServer(grpc.StreamInterceptor(orderServerStreamInterceptor))
    ...
}
```

## 2. 客户端拦截器

在服务端可以拦截收到的 RPC 调用，客户端同样可以拦截发出去的 RPC 请求以及收到的响应，同样可以实现一元拦截器以及流拦截器。

### 2.1 一元拦截器

和服务端一元拦截器一样的方法，只是方法参数略微有所差别，此外在建立连接的时候注册拦截器，同样可以注册多个拦截器：

```go
// method请求方法字符串，req包含请求的所有信息参数等，reply在实际RPC调用后存储响应信息，通过invoker实际调用
func orderUnaryClientInterceptor(ctx context.Context, method string, req, reply interface{},
	cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	// 前置处理阶段
	log.Println("method: " + method)
	// 实际的RPC调用
	err := invoker(ctx, method, req, reply, cc, opts...)
	// 后置处理
	log.Println(reply)
	return err
}

func main() {
    ...
	conn, err := grpc.Dial(address, grpc.WithInsecure(), 	          grpc.WithUnaryInterceptor(orderUnaryClientInterceptor))
    ...
}
```

### 2.2 流拦截器

流拦截器也是和服务端一样的步骤：

```go
type wrappedStream struct {
	grpc.ClientStream
}

func (w *wrappedStream) SendMsg(m interface{}) error {
	log.Printf("[stream interceptor send] %s", m)
	return w.ClientStream.SendMsg(m)
}
func (w *wrappedStream) RecvMsg(m interface{}) error {
	log.Printf("[stream interceptor recv] type: %T", m)
	return w.ClientStream.RecvMsg(m)
}

func orderClientStreamInterceptor(ctx context.Context, desc *grpc.StreamDesc,
	cc *grpc.ClientConn, method string, streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
    // 前置处理阶段，RPC请求发出之前拦截
	log.Printf("[client interceptor send] %s", method)
    // 发出RPC请求
	s, err := streamer(ctx, desc, cc, method, opts...)
	if err != nil {
		return nil, err
	}
	return &wrappedStream{s}, nil
}

func main() {
    ...
	conn, err := grpc.Dial(address, grpc.WithInsecure(),
		grpc.WithStreamInterceptor(orderClientStreamInterceptor))
    ...
}
```