+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/go/basics/](https://grpc.io/docs/languages/go/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Go.

​	Go 中 gRPC 的基本教程介绍。



This tutorial provides a basic Go programmer’s introduction to working with gRPC.

​	本教程为 Go 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Go gRPC API to write a simple client and server for your service.
  使用 Go gRPC API 为您的服务编写一个简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and the [Go generated code guide](https://protobuf.dev/reference/go/go-generated).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南和 Go 生成的代码指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Setup 设置

You should have already installed the tools needed to generate client and server interface code – if you haven’t, see the [Prerequisites]({{< ref "/Languages/Go/Quickstart#prerequisites" >}}) section of [Quick start]({{< ref "/Languages/Go/Quickstart" >}}) for setup instructions.

​	您应该已经安装了生成客户端和服务器接口代码所需的工具 - 如果您还没有安装，请参阅快速入门中的先决条件部分以获取设置说明。

### Get the example code 获取示例代码

The example code is part of the [grpc-go](https://github.com/grpc/grpc-go) repo.

​	示例代码是 grpc-go 存储库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-go/archive/v1.60.1.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone -b v1.60.1 --depth 1 https://github.com/grpc/grpc-go
   ```

2. Change to the example directory:

   ​	更改到示例目录：

   ```sh
   $ cd grpc-go/examples/route_guide
   ```

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). For the complete `.proto` file, see [routeguide/route_guide.proto](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide/route_guide.proto).

​	我们的第一步（如您从 gRPC 简介中了解到的）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。有关完整的 `.proto` 文件，请参阅 routeguide/route_guide.proto。

To define a service, you specify a named `service` in your `.proto` file:

​	要定义服务，您需要在 `.proto` 文件中指定一个命名的 `service` ：

```proto
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the `RouteGuide` service:

​	然后在服务定义中定义 `rpc` 方法，指定它们的请求和响应类型。gRPC 允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

- A *simple RPC* where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.

  ​	一个简单的 RPC，其中客户端使用存根向服务器发送请求并等待响应返回，就像一个普通的函数调用一样。

  ```proto
  // Obtains the feature at a given position.
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A *server-side streaming RPC* where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in our example, you specify a server-side streaming method by placing the `stream` keyword before the *response* type.

  ​	一个服务器端流式 RPC，其中客户端向服务器发送请求并获取一个流来读取一系列消息。客户端从返回的流中读取，直到没有更多消息。如您在我们的示例中所见，您可以通过在响应类型之前放置 `stream` 关键字来指定服务器端流式方法。

  ```proto
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A *client-side streaming RPC* where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a client-side streaming method by placing the `stream` keyword before the *request* type.

  ​	一个客户端流式 RPC，其中客户端编写一系列消息并使用提供的流将它们发送到服务器。一旦客户端完成编写消息，它就会等待服务器读取所有消息并返回其响应。您可以通过在请求类型之前放置 `stream` 关键字来指定客户端流式方法。

  ```proto
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A *bidirectional streaming RPC* where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  ​	双方使用读写流发送一系列消息的双向流式 RPC。两个流独立运行，因此客户端和服务器可以按照他们喜欢的任何顺序进行读写：例如，服务器可以等到收到所有客户端消息后再写出其响应，或者可以交替读取消息然后写出消息，或者其他一些读写组合。每个流中的消息顺序都将保留。通过在请求和响应之前放置 `stream` 关键字来指定此类型的函数。

  ```proto
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
  ```

Our `.proto` file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the `Point` message type:

​	我们的 `.proto` 文件还包含用于我们服务函数中的所有请求和响应类型的协议缓冲区消息类型定义 - 例如，以下是 `Point` 消息类型：

```proto
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

### Generating client and server code 生成客户端和服务器代码

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC Go plugin. This is similar to what we did in the [Quick start]({{< ref "/Languages/Go/Quickstart" >}}).

​	接下来，我们需要从我们的 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。我们使用协议缓冲区编译器 `protoc` 和特殊的 gRPC Go 插件来执行此操作。这与我们在快速入门中所做的事情类似。

From the `examples/route_guide` directory, run the following command:

​	在 `examples/route_guide` 目录中，运行以下命令：

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto
```

Running this command generates the following files in the [routeguide](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide) directory:

​	运行此命令将在 routeguide 目录中生成以下文件：

- `route_guide.pb.go`, which contains all the protocol buffer code to populate, serialize, and retrieve request and response message types.
  `route_guide.pb.go` ，其中包含用于填充、序列化和检索请求和响应消息类型的全部协议缓冲区代码。

- ```
  route_guide_grpc.pb.go
  ```

  , which contains the following:

  
  `route_guide_grpc.pb.go` ，其中包含以下内容：

  - An interface type (or *stub*) for clients to call with the methods defined in the `RouteGuide` service.
    一个接口类型（或存根）供客户端使用在 `RouteGuide` 服务中定义的方法进行调用。
  - An interface type for servers to implement, also with the methods defined in the `RouteGuide` service.
    服务器要实现的接口类型，还包括在 `RouteGuide` 服务中定义的方法。

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Go/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  实现从服务定义中生成的服务接口：执行服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and dispatch them to the right service implementation.
  运行 gRPC 服务器以侦听来自客户端的请求并将它们分派到正确的服务实现。

You can find our example `RouteGuide` server in [server/server.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/server/server.go). Let’s take a closer look at how it works.

​	您可以在 server/server.go 中找到我们的示例 `RouteGuide` 服务器。让我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `routeGuideServer` struct type that implements the generated `RouteGuideServer` interface:

​	如您所见，我们的服务器有一个 `routeGuideServer` 结构类型，它实现了生成的 `RouteGuideServer` 接口：

```go
type routeGuideServer struct {
        ...
}
...

func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
        ...
}
...

func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
        ...
}
...

func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
        ...
}
...

func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
        ...
}
...
```

##### Simple RPC 简单 RPC

The `routeGuideServer` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	 `routeGuideServer` 实现了我们所有的服务方法。我们先来看最简单的类型 `GetFeature` ，它只是从客户端获取一个 `Point` ，并从其数据库中返回相应的特征信息，保存在 `Feature` 中。

```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
  for _, feature := range s.savedFeatures {
    if proto.Equal(feature.Location, point) {
      return feature, nil
    }
  }
  // No feature was found, return an unnamed feature
  return &pb.Feature{Location: point}, nil
}
```

The method is passed a context object for the RPC and the client’s `Point` protocol buffer request. It returns a `Feature` protocol buffer object with the response information and an `error`. In the method we populate the `Feature` with the appropriate information, and then `return` it along with a `nil` error to tell gRPC that we’ve finished dealing with the RPC and that the `Feature` can be returned to the client.

​	该方法会传递一个 RPC 的上下文对象和客户端的 `Point` 协议缓冲区请求。它返回一个包含响应信息和 `error` 的 `Feature` 协议缓冲区对象。在该方法中，我们使用适当的信息填充 `Feature` ，然后使用 `nil` 错误将其 `return` ，以告诉 gRPC 我们已完成处理 RPC，并且可以将 `Feature` 返回给客户端。

##### Server-side streaming RPC 服务器端流式 RPC

Now let’s look at one of our streaming RPCs. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	现在，我们来看一下我们的流式 RPC 之一。 `ListFeatures` 是一个服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```go
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
  for _, feature := range s.savedFeatures {
    if inRange(feature.Location, rect) {
      if err := stream.Send(feature); err != nil {
        return err
      }
    }
  }
  return nil
}
```

As you can see, instead of getting simple request and response objects in our method parameters, this time we get a request object (the `Rectangle` in which our client wants to find `Feature`s) and a special `RouteGuide_ListFeaturesServer` object to write our responses.

​	正如您所见，这次我们不是在方法参数中获取简单的请求和响应对象，而是获取一个请求对象（ `Rectangle` ，其中我们的客户端想要查找 `Feature` ）和一个特殊的 `RouteGuide_ListFeaturesServer` 对象来写入我们的响应。

In the method, we populate as many `Feature` objects as we need to return, writing them to the `RouteGuide_ListFeaturesServer` using its `Send()` method. Finally, as in our simple RPC, we return a `nil` error to tell gRPC that we’ve finished writing responses. Should any error happen in this call, we return a non-`nil` error; the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

​	在该方法中，我们填充尽可能多的 `Feature` 对象以供返回，使用其 `Send()` 方法将它们写入 `RouteGuide_ListFeaturesServer` 。最后，与在我们的简单 RPC 中一样，我们返回 `nil` 错误以告诉 gRPC 我们已完成写入响应。如果在此调用中发生任何错误，我们返回一个非 `nil` 错误；gRPC 层会将其转换为适当的 RPC 状态以在网络上发送。

##### Client-side streaming RPC 客户端流式 RPC

Now let’s look at something a little more complicated: the client-side streaming method `RecordRoute`, where we get a stream of `Point`s from the client and return a single `RouteSummary` with information about their trip. As you can see, this time the method doesn’t have a request parameter at all. Instead, it gets a `RouteGuide_RecordRouteServer` stream, which the server can use to both read *and* write messages - it can receive client messages using its `Recv()` method and return its single response using its `SendAndClose()` method.

​	现在，我们来看一些更复杂的内容：客户端流式传输方法 `RecordRoute` ，其中我们从客户端获取 `Point` 流并返回一个包含有关其行程信息的 `RouteSummary` 。如您所见，这次该方法根本没有请求参数。相反，它获取 `RouteGuide_RecordRouteServer` 流，服务器可以使用该流来读取和写入消息 - 它可以使用其 `Recv()` 方法接收客户端消息并使用其 `SendAndClose()` 方法返回其单个响应。

```go
func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
  var pointCount, featureCount, distance int32
  var lastPoint *pb.Point
  startTime := time.Now()
  for {
    point, err := stream.Recv()
    if err == io.EOF {
      endTime := time.Now()
      return stream.SendAndClose(&pb.RouteSummary{
        PointCount:   pointCount,
        FeatureCount: featureCount,
        Distance:     distance,
        ElapsedTime:  int32(endTime.Sub(startTime).Seconds()),
      })
    }
    if err != nil {
      return err
    }
    pointCount++
    for _, feature := range s.savedFeatures {
      if proto.Equal(feature.Location, point) {
        featureCount++
      }
    }
    if lastPoint != nil {
      distance += calcDistance(lastPoint, point)
    }
    lastPoint = point
  }
}
```

In the method body we use the `RouteGuide_RecordRouteServer`’s `Recv()` method to repeatedly read in our client’s requests to a request object (in this case a `Point`) until there are no more messages: the server needs to check the error returned from `Recv()` after each call. If this is `nil`, the stream is still good and it can continue reading; if it’s `io.EOF` the message stream has ended and the server can return its `RouteSummary`. If it has any other value, we return the error “as is” so that it’ll be translated to an RPC status by the gRPC layer.

​	在方法体中，我们使用 `RouteGuide_RecordRouteServer` 的 `Recv()` 方法反复读取客户端的请求到请求对象（在本例中为 `Point` ）中，直到没有更多消息：服务器需要在每次调用后检查从 `Recv()` 返回的错误。如果这是 `nil` ，则流仍然良好，并且可以继续读取；如果它是 `io.EOF` ，则消息流已结束，服务器可以返回其 `RouteSummary` 。如果它有任何其他值，我们原样返回错误，以便将其由 gRPC 层转换为 RPC 状态。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```go
func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
  for {
    in, err := stream.Recv()
    if err == io.EOF {
      return nil
    }
    if err != nil {
      return err
    }
    key := serialize(in.Location)
                ... // look for notes to be sent to client
    for _, note := range s.routeNotes[key] {
      if err := stream.Send(note); err != nil {
        return err
      }
    }
  }
}
```

This time we get a `RouteGuide_RouteChatServer` stream that, as in our client-side streaming example, can be used to read and write messages. However, this time we return values via our method’s stream while the client is still writing messages to *their* message stream.

​	这次我们得到一个 `RouteGuide_RouteChatServer` 流，与我们的客户端流式传输示例一样，可用于读写消息。但是，这次我们通过方法的流返回值，而客户端仍在向其消息流写入消息。

The syntax for reading and writing here is very similar to our client-streaming method, except the server uses the stream’s `Send()` method rather than `SendAndClose()` because it’s writing multiple responses. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	此处用于读写消息的语法与我们的客户端流式传输方法非常相似，只是服务器使用流的 `Send()` 方法而不是 `SendAndClose()` ，因为它正在写入多个响应。尽管每一方总是按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序读写——流完全独立地运行。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```go
lis, err := net.Listen("tcp", fmt.Sprintf("localhost:%d", port))
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}
var opts []grpc.ServerOption
...
grpcServer := grpc.NewServer(opts...)
pb.RegisterRouteGuideServer(grpcServer, newServer())
grpcServer.Serve(lis)
```

To build and start a server, we:

​	要构建并启动服务器，我们：

1. Specify the port we want to use to listen for client requests using:
   使用以下方式指定我们想要用来侦听客户端请求的端口：
   `lis, err := net.Listen(...)`.
2. Create an instance of the gRPC server using `grpc.NewServer(...)`.
   使用 `grpc.NewServer(...)` 创建 gRPC 服务器的实例。
3. Register our service implementation with the gRPC server.
   将我们的服务实现注册到 gRPC 服务器。
4. Call `Serve()` on the server with our port details to do a blocking wait until the process is killed or `Stop()` is called.
   在服务器上使用我们的端口详细信息调用 `Serve()` ，以执行阻塞等待，直到进程被终止或调用 `Stop()` 。

### Creating the client 创建客户端

In this section, we’ll look at creating a Go client for our `RouteGuide` service. You can see our complete example client code in [grpc-go/examples/route_guide/client/client.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/client/client.go).

​	在本节中，我们将着眼于为我们的 `RouteGuide` 服务创建一个 Go 客户端。您可以在 grpc-go/examples/route_guide/client/client.go 中看到我们完整的示例客户端代码。

#### Creating a stub 创建存根

To call service methods, we first need to create a gRPC *channel* to communicate with the server. We create this by passing the server address and port number to `grpc.Dial()` as follows:

​	要调用服务方法，我们首先需要创建一个 gRPC 通道来与服务器通信。我们通过将服务器地址和端口号传递给 `grpc.Dial()` 来创建此通道，如下所示：

```go
var opts []grpc.DialOption
...
conn, err := grpc.Dial(*serverAddr, opts...)
if err != nil {
  ...
}
defer conn.Close()
```

You can use `DialOptions` to set the auth credentials (for example, TLS, GCE credentials, or JWT credentials) in `grpc.Dial` when a service requires them. The `RouteGuide` service doesn’t require any credentials.

​	当服务需要时，您可以使用 `DialOptions` 在 `grpc.Dial` 中设置身份验证凭据（例如，TLS、GCE 凭据或 JWT 凭据）。 `RouteGuide` 服务不需要任何凭据。

Once the gRPC *channel* is setup, we need a client *stub* to perform RPCs. We get it using the `NewRouteGuideClient` method provided by the `pb` package generated from the example `.proto` file.

​	一旦 gRPC 通道设置好，我们就需要一个客户端存根来执行 RPC。我们可以使用由示例 `.proto` 文件生成的 `pb` 包提供的 `NewRouteGuideClient` 方法来获取它。

```go
client := pb.NewRouteGuideClient(conn)
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. Note that in gRPC-Go, RPCs operate in a blocking/synchronous mode, which means that the RPC call waits for the server to respond, and will either return a response or an error.

​	现在，我们来看看如何调用我们的服务方法。请注意，在 gRPC-Go 中，RPC 以阻塞/同步模式运行，这意味着 RPC 调用会等待服务器响应，并且会返回响应或错误。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.

​	调用简单 RPC `GetFeature` 几乎与调用本地方法一样简单。

```go
feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
if err != nil {
  ...
}
```

As you can see, we call the method on the stub we got earlier. In our method parameters we create and populate a request protocol buffer object (in our case `Point`). We also pass a `context.Context` object which lets us change our RPC’s behavior if necessary, such as time-out/cancel an RPC in flight. If the call doesn’t return an error, then we can read the response information from the server from the first return value.

​	如您所见，我们调用了之前获得的存根上的方法。在我们的方法参数中，我们创建并填充了一个请求协议缓冲区对象（在本例中为 `Point` ）。我们还传递了一个 `context.Context` 对象，它允许我们在必要时更改 RPC 的行为，例如超时/取消正在进行的 RPC。如果调用没有返回错误，那么我们可以从第一个返回值中读取服务器的响应信息。

```go
log.Println(feature)
```

##### Server-side streaming RPC 服务器端流式 RPC

Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s. If you’ve already read [Creating the server]({{< ref "/Languages/Go/Basicstutorial#server" >}}) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides.

​	我们在此调用服务器端流式方法 `ListFeatures` ，该方法返回一个地理 `Feature` 流。如果您已经阅读了创建服务器，其中一些内容可能看起来非常熟悉 - 流式 RPC 在双方以类似的方式实现。

```go
rect := &pb.Rectangle{ ... }  // initialize a pb.Rectangle
stream, err := client.ListFeatures(context.Background(), rect)
if err != nil {
  ...
}
for {
    feature, err := stream.Recv()
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
    }
    log.Println(feature)
}
```

As in the simple RPC, we pass the method a context and a request. However, instead of getting a response object back, we get back an instance of `RouteGuide_ListFeaturesClient`. The client can use the `RouteGuide_ListFeaturesClient` stream to read the server’s responses.

​	与简单 RPC 一样，我们将方法、上下文和请求传递给方法。但是，我们不会返回一个响应对象，而是返回一个 `RouteGuide_ListFeaturesClient` 实例。客户端可以使用 `RouteGuide_ListFeaturesClient` 流来读取服务器的响应。

We use the `RouteGuide_ListFeaturesClient`’s `Recv()` method to repeatedly read in the server’s responses to a response protocol buffer object (in this case a `Feature`) until there are no more messages: the client needs to check the error `err` returned from `Recv()` after each call. If `nil`, the stream is still good and it can continue reading; if it’s `io.EOF` then the message stream has ended; otherwise there must be an RPC error, which is passed over through `err`.

​	我们使用 `RouteGuide_ListFeaturesClient` 的 `Recv()` 方法反复读取服务器对响应协议缓冲区对象（在本例中为 `Feature` ）的响应，直到没有更多消息：客户端需要检查 `Recv()` 每次调用后返回的错误 `err` 。如果 `nil` ，则流仍然良好，并且可以继续读取；如果它是 `io.EOF` ，则消息流已结束；否则，必须存在通过 `err` 传递的 RPC 错误。

##### Client-side streaming RPC 客户端流式 RPC

The client-side streaming method `RecordRoute` is similar to the server-side method, except that we only pass the method a context and get a `RouteGuide_RecordRouteClient` stream back, which we can use to both write *and* read messages.

​	客户端流方法 `RecordRoute` 与服务器端方法类似，只是我们只将方法传递给上下文并获得 `RouteGuide_RecordRouteClient` 流，我们可以使用它来写入和读取消息。

```go
// Create a random number of random points
r := rand.New(rand.NewSource(time.Now().UnixNano()))
pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points
var points []*pb.Point
for i := 0; i < pointCount; i++ {
  points = append(points, randomPoint(r))
}
log.Printf("Traversing %d points.", len(points))
stream, err := client.RecordRoute(context.Background())
if err != nil {
  log.Fatalf("%v.RecordRoute(_) = _, %v", client, err)
}
for _, point := range points {
  if err := stream.Send(point); err != nil {
    log.Fatalf("%v.Send(%v) = %v", stream, point, err)
  }
}
reply, err := stream.CloseAndRecv()
if err != nil {
  log.Fatalf("%v.CloseAndRecv() got error %v, want %v", stream, err, nil)
}
log.Printf("Route summary: %v", reply)
```

The `RouteGuide_RecordRouteClient` has a `Send()` method that we can use to send requests to the server. Once we’ve finished writing our client’s requests to the stream using `Send()`, we need to call `CloseAndRecv()` on the stream to let gRPC know that we’ve finished writing and are expecting to receive a response. We get our RPC status from the `err` returned from `CloseAndRecv()`. If the status is `nil`, then the first return value from `CloseAndRecv()` will be a valid server response.

​	 `RouteGuide_RecordRouteClient` 具有 `Send()` 方法，我们可以使用它向服务器发送请求。一旦我们使用 `Send()` 将客户端的请求写入流中，我们就需要在流上调用 `CloseAndRecv()` 以便让 gRPC 知道我们已经完成写入并期望收到响应。我们从 `CloseAndRecv()` 返回的 `err` 中获取我们的 RPC 状态。如果状态是 `nil` ，则 `CloseAndRecv()` 的第一个返回值将是有效的服务器响应。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. As in the case of `RecordRoute`, we only pass the method a context object and get back a stream that we can use to both write and read messages. However, this time we return values via our method’s stream while the server is still writing messages to *their* message stream.

​	最后，我们来看看我们的双向流式 RPC `RouteChat()` 。与 `RecordRoute` 的情况一样，我们只将方法传递给上下文对象，并返回一个流，我们可以用它来写入和读取消息。但是，这次我们通过方法的流返回值，而服务器仍在向其消息流写入消息。

```go
stream, err := client.RouteChat(context.Background())
waitc := make(chan struct{})
go func() {
  for {
    in, err := stream.Recv()
    if err == io.EOF {
      // read done.
      close(waitc)
      return
    }
    if err != nil {
      log.Fatalf("Failed to receive a note : %v", err)
    }
    log.Printf("Got message %s at point(%d, %d)", in.Message, in.Location.Latitude, in.Location.Longitude)
  }
}()
for _, note := range notes {
  if err := stream.Send(note); err != nil {
    log.Fatalf("Failed to send a note: %v", err)
  }
}
stream.CloseSend()
<-waitc
```

The syntax for reading and writing here is very similar to our client-side streaming method, except we use the stream’s `CloseSend()` method once we’ve finished our call. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	此处用于读写操作的语法与我们的客户端流式方法非常相似，只不过在我们完成调用后，我们使用流的 `CloseSend()` 方法。尽管每一方总是按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序进行读写操作——这些流完全独立地运行。

### Try it out! 试一试！

Execute the following commands from the `examples/route_guide` directory:

​	从 `examples/route_guide` 目录执行以下命令：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ go run server/server.go
   ```

2. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ go run client/client.go
   ```

You’ll see output like this:

​	您将看到类似这样的输出：

```nocode
Getting feature for point (409146138, -746188906)
name:"Berkshire Valley Management Area Trail, Jefferson, NJ, USA" location:<latitude:409146138 longitude:-746188906 >
Getting feature for point (0, 0)
location:<>
Looking for features within lo:<latitude:400000000 longitude:-750000000 > hi:<latitude:420000000 longitude:-730000000 >
name:"Patriots Path, Mendham, NJ 07945, USA" location:<latitude:407838351 longitude:-746143763 >
...
name:"3 Hasta Way, Newton, NJ 07860, USA" location:<latitude:410248224 longitude:-747127767 >
Traversing 56 points.
Route summary: point_count:56 distance:497013163
Got message First message at point(0, 1)
Got message Second message at point(0, 2)
Got message Third message at point(0, 3)
Got message First message at point(0, 1)
Got message Fourth message at point(0, 1)
Got message Second message at point(0, 2)
Got message Fifth message at point(0, 2)
Got message Third message at point(0, 3)
Got message Sixth message at point(0, 3)
```

#### Note 注意

We’ve omitted timestamps from the client and server trace output shown in this page.
我们已从本页中显示的客户端和服务器跟踪输出中省略了时间戳。
