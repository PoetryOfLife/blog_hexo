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
from stream client question: stream client rpc 4
```