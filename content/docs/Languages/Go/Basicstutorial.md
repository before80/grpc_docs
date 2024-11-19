+++
title = "基础教程"
date = 2024-11-19T10:19:42+08:00
weight = 10
type = "docs"
description = "一个针对 Go 中 gRPC 的基础教程。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/languages/go/basics/](https://grpc.io/docs/languages/go/basics/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Basics tutorial - 基础教程

A basic tutorial introduction to gRPC in Go.

​	一个针对 Go 中 gRPC 的基础教程。

This tutorial provides a basic Go programmer’s introduction to working with gRPC.

​	本教程为 Go 程序员提供了 gRPC 的基础入门。

By walking through this example you’ll learn how to:

​	通过学习本示例，您将了解如何：

- Define a service in a `.proto` file.
  - 在 `.proto` 文件中定义服务。

- Generate server and client code using the protocol buffer compiler.
  - 使用协议缓冲区编译器生成服务器和客户端代码。

- Use the Go gRPC API to write a simple client and server for your service.
  - 使用 Go 的 gRPC API 编写一个简单的客户端和服务器。


It assumes that you have read the [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and the [Go generated code guide](https://protobuf.dev/reference/go/go-generated).

​	本教程假设您已经阅读了 [gRPC 简介]({{< ref "/docs/WhatisgRPC/Introduction" >}})，并熟悉 [协议缓冲区](https://protobuf.dev/overview)。请注意，本教程中的示例使用的是协议缓冲区语言的 proto3 版本。您可以在 [proto3 语言指南](https://protobuf.dev/programming-guides/proto3) 和 [Go 生成代码指南](https://protobuf.dev/reference/go/go-generated) 中了解更多信息。

### Why use gRPC?

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，允许客户端获取路线上的特征信息、创建路线摘要，并与服务器和其他客户端交换路线信息（例如交通更新）。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	通过 gRPC，我们可以在 `.proto` 文件中定义服务，并生成任意支持 gRPC 的语言的客户端和服务器代码，这些代码可以在从大型数据中心内的服务器到您自己的平板电脑的各种环境中运行。gRPC 为您处理了不同语言和环境之间通信的所有复杂性。此外，您还能享受到协议缓冲区的所有优势，包括高效的序列化、简单的 IDL 和轻松的接口更新。

### Setup

You should have already installed the tools needed to generate client and server interface code – if you haven’t, see the [Prerequisites](https://grpc.io/docs/languages/go/quickstart/#prerequisites) section of [Quick start]({{< ref "/docs/Languages/Go/Quickstart" >}}) for setup instructions.

​	您应该已经安装了生成客户端和服务器接口代码所需的工具。如果尚未安装，请参阅 [快速入门]({{< ref "/docs/Languages/Go/Quickstart" >}})的[前提条件](https://grpc.io/docs/languages/go/quickstart/#prerequisites)部分了解安装说明。

### 获取示例代码 Get the example code

The example code is part of the [grpc-go](https://github.com/grpc/grpc-go) repo.

​	示例代码位于 [grpc-go](https://github.com/grpc/grpc-go) 仓库中。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-go/archive/v1.68.0.zip) and unzip it, or clone the repo: [将仓库下载为 zip 文件](https://github.com/grpc/grpc-go/archive/v1.68.0.zip) 并解压，或克隆仓库：

   ```sh
   $ git clone -b v1.68.0 --depth 1 https://github.com/grpc/grpc-go
   ```

2. Change to the example directory: 切换到示例目录：

   ```sh
   $ cd grpc-go/examples/route_guide
   ```

### 定义服务 Defining the service

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). For the complete `.proto` file, see [routeguide/route_guide.proto](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide/route_guide.proto).

​	第一步（您可能从 [gRPC 简介]({{< ref "/docs/WhatisgRPC/Introduction" >}})中了解过）是使用 [协议缓冲区](https://protobuf.dev/overview) 定义 gRPC **服务**以及方法的**请求**和**响应**类型。完整的 `.proto` 文件请参见 [routeguide/route_guide.proto](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide/route_guide.proto)。

To define a service, you specify a named `service` in your `.proto` file:

​	要定义服务，请在 `.proto` 文件中指定一个命名的 `service`：

```proto
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the `RouteGuide` service:

​	然后，在服务定义中定义 `rpc` 方法，指定其请求和响应类型。gRPC 支持定义四种服务方法，`RouteGuide` 服务中使用了全部这四种方法：

- A *simple RPC* where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.

  - **简单 RPC**：客户端通过存根向服务器发送请求并等待响应，就像普通的函数调用。


  ```proto
  // Obtains the feature at a given position.
  // 获取指定位置的特征。
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A *server-side streaming RPC* where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in our example, you specify a server-side streaming method by placing the `stream` keyword before the *response* type.

  - **服务器端流式 RPC**：客户端向服务器发送请求并接收一个流来读取一系列消息，直到没有更多消息为止。要定义服务器端流式方法，可以在*响应*类型前添加 `stream` 关键字。


  ```proto
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  // 获取指定矩形内可用的特征。结果以流的形式返回，而不是一次性返回（例如通过带有重复字段的响应消息），因为矩形可能覆盖较大的区域并包含大量特征。
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A *client-side streaming RPC* where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a client-side streaming method by placing the `stream` keyword before the *request* type.

  - **客户端流式 RPC**：客户端写入一系列消息并将其发送到服务器，使用提供的流。客户端完成写入后，等待服务器读取所有消息并返回响应。在*请求*类型前添加 `stream` 关键字即可定义客户端流式方法。


  ```proto
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  // 接收被遍历的路线上的点流，并在遍历完成时返回路线摘要。
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A *bidirectional streaming RPC* where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  - **双向流式 RPC**：客户端和服务器都通过读写流发送一系列消息。两个流独立运行，客户端和服务器可以以任意顺序读取和写入。例如，服务器可以等待接收所有客户端消息后再写入响应，或交替读取消息和写入消息，或者采用其他读写组合。在每个流中的消息顺序均得以保留。您可以在请求和响应类型前都添加 `stream` 关键字来定义此类方法。

  ```proto
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  // 接收路线被遍历时发送的路线笔记流，同时接收其他路线笔记（例如来自其他用户）。
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
  ```


Our `.proto` file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the `Point` message type:

​	`.proto` 文件还包含所有服务方法中请求和响应类型的协议缓冲区消息类型定义。例如，以下是 `Point` 消息类型：

```proto
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
// 点用 E7 表示法表示为纬度-经度对（度数乘以 10**7 并四舍五入为最接近的整数）。纬度范围为 +/- 90 度， 经度范围为 +/- 180 度（包括端点）。
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

### 生成客户端和服务器代码 Generating client and server code

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC Go plugin. This is similar to what we did in the [Quick start]({{< ref "/docs/Languages/Go/Quickstart" >}}).

​	接下来，我们需要从 `.proto` 服务定义生成 gRPC 客户端和服务器接口。为此，我们使用协议缓冲区编译器 `protoc` 及其专用的 gRPC Go 插件。这与 [快速入门]({{< ref "/docs/Languages/Go/Quickstart" >}})中的过程类似。

From the `examples/route_guide` directory, run the following command:

​	在 `examples/route_guide` 目录中，运行以下命令：

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto
```

Running this command generates the following files in the [routeguide](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide) directory:

​	运行此命令会在 [routeguide](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/routeguide) 目录中生成以下文件：

- `route_guide.pb.go`, which contains all the protocol buffer code to populate, serialize, and retrieve request and response message types.
  - `route_guide.pb.go`：包含用于填充、序列化和检索请求和响应消息类型的所有协议缓冲区代码。

- `route_guide_grpc.pb.go`, which contains the following: `route_guide_grpc.pb.go`：包含以下内容：
  - An interface type (or *stub*) for clients to call with the methods defined in the `RouteGuide` service.
    - 用于客户端调用 `RouteGuide` 服务中定义的方法的接口类型（或 *存根*）。
  - An interface type for servers to implement, also with the methods defined in the `RouteGuide` service.
    - 用于服务器实现的接口类型，同样包含 `RouteGuide` 服务中定义的方法。

### Creating the server

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client](https://grpc.io/docs/languages/go/basics/#client) (though you might find it interesting anyway!).

​	接下来，我们来看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本节，直接查看 [创建客户端](https://grpc.io/docs/languages/go/basics/#client)（尽管了解一下服务器的实现也会很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	要实现 `RouteGuide` 服务，主要需要完成以下两部分工作：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  - 实现从服务定义生成的服务接口：执行服务的实际“工作”。

- Running a gRPC server to listen for requests from clients and dispatch them to the right service implementation.
  - 运行 gRPC 服务器以侦听客户端的请求并将其分派到正确的服务实现。


You can find our example `RouteGuide` server in [server/server.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/server/server.go). Let’s take a closer look at how it works.

​	您可以在 [server/server.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/server/server.go) 中找到我们的示例 `RouteGuide` 服务器。接下来，我们将仔细了解其工作原理。

#### Implementing RouteGuide

As you can see, our server has a `routeGuideServer` struct type that implements the generated `RouteGuideServer` interface:

​	服务器有一个 `routeGuideServer` 结构类型，它实现了生成的 `RouteGuideServer` 接口：

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

##### 简单 RPC - Simple RPC

The `routeGuideServer` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	`routeGuideServer` 实现了所有服务方法。让我们先看最简单的类型 `GetFeature`，它从客户端获取一个 `Point` 并从其数据库中返回相应的特征信息作为 `Feature`。

```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
  for _, feature := range s.savedFeatures {
    if proto.Equal(feature.Location, point) {
      return feature, nil
    }
  }
  // No feature was found, return an unnamed feature
  // 未找到特征，返回一个未命名的特征
  return &pb.Feature{Location: point}, nil
}
```

The method is passed a context object for the RPC and the client’s `Point` protocol buffer request. It returns a `Feature` protocol buffer object with the response information and an `error`. In the method we populate the `Feature` with the appropriate information, and then `return` it along with a `nil` error to tell gRPC that we’ve finished dealing with the RPC and that the `Feature` can be returned to the client.

​	该方法接收一个 RPC 的上下文对象和客户端的 `Point` 协议缓冲区请求。它返回一个包含响应信息的 `Feature` 协议缓冲区对象以及一个 `error`。在方法中，我们填充 `Feature` 的相应信息，然后返回它以及一个 `nil` 错误，以告诉 gRPC 我们已完成 RPC 的处理，并且 `Feature` 可以返回给客户端。

##### 服务器端流式 RPC - Server-side streaming RPC

Now let’s look at one of our streaming RPCs. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	现在来看一个流式 RPC 示例。`ListFeatures` 是一个服务器端流式 RPC，因此我们需要将多个 `Feature` 返回给客户端。

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

​	如您所见，与简单 RPC 不同，此时我们从方法参数中获取了一个请求对象（客户端想在其中找到 `Feature` 的 `Rectangle`）和一个特殊的 `RouteGuide_ListFeaturesServer` 对象，用于写入响应。

In the method, we populate as many `Feature` objects as we need to return, writing them to the `RouteGuide_ListFeaturesServer` using its `Send()` method. Finally, as in our simple RPC, we return a `nil` error to tell gRPC that we’ve finished writing responses. Should any error happen in this call, we return a non-`nil` error; the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

​	在方法中，我们填充需要返回的多个 `Feature` 对象，并使用 `RouteGuide_ListFeaturesServer` 的 `Send()` 方法将它们写入响应流。最后，与简单 RPC 类似，我们返回一个 `nil` 错误以告诉 gRPC 响应已完成。如果在调用中发生任何错误，我们返回非 `nil` 错误，gRPC 层将其转换为适当的 RPC 状态并发送。

##### 客户端流式 RPC - Client-side streaming RPC

Now let’s look at something a little more complicated: the client-side streaming method `RecordRoute`, where we get a stream of `Point`s from the client and return a single `RouteSummary` with information about their trip. As you can see, this time the method doesn’t have a request parameter at all. Instead, it gets a `RouteGuide_RecordRouteServer` stream, which the server can use to both read *and* write messages - it can receive client messages using its `Recv()` method and return its single response using its `SendAndClose()` method.

​	现在来看更复杂一些的客户端流式方法 `RecordRoute`。在此方法中，我们从客户端获取 `Point` 的流，并返回一个包含行程信息的 `RouteSummary`。您会注意到，此方法没有请求参数，而是获取一个 `RouteGuide_RecordRouteServer` 流，服务器可以使用它同时读取和写入消息——通过其 `Recv()` 方法接收客户端消息，通过 `SendAndClose()` 方法返回单个响应。

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

​	在方法体中，我们使用 `RouteGuide_RecordRouteServer` 的 `Recv()` 方法反复读取客户端请求对象（此处为 `Point`），直到没有更多消息：服务器需要在每次调用后检查 `Recv()` 返回的错误。如果是 `nil`，表示流仍然有效，可以继续读取；如果是 `io.EOF`，表示消息流已结束，服务器可以返回其 `RouteSummary`；如果是其他值，则返回该错误，gRPC 层将其转换为适当的 RPC 状态。

##### 双向流式 RPC - Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，来看我们的双向流式 RPC `RouteChat()`。

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

​	在这里，我们获得一个 `RouteGuide_RouteChatServer` 流，与客户端流式 RPC 示例类似，可用于读取和写入消息。但这次，我们在方法流中返回值，而客户端仍在其消息流中写入消息。

The syntax for reading and writing here is very similar to our client-streaming method, except the server uses the stream’s `Send()` method rather than `SendAndClose()` because it’s writing multiple responses. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	读取和写入的语法与客户端流式方法非常相似，但服务器使用流的 `Send()` 方法而非 `SendAndClose()`，因为它要写入多个响应。尽管两端始终按写入顺序接收对方的消息，但客户端和服务器可以以任意顺序读取和写入——两个流完全独立运行。

#### Starting the server

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们需要启动 gRPC 服务器以便客户端可以使用我们的服务。以下代码片段展示了我们如何为 `RouteGuide` 服务启动服务器：

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

​	要构建并启动服务器，我们需要：

1. Specify the port we want to use to listen for client requests using: 指定用于监听客户端请求的端口，使用： `lis, err := net.Listen(...)`。
   `lis, err := net.Listen(...)`.
2. Create an instance of the gRPC server using `grpc.NewServer(...)`. 使用 `grpc.NewServer(...)` 创建 gRPC 服务器实例。
3. Register our service implementation with the gRPC server. 将服务实现注册到 gRPC 服务器。
4. Call `Serve()` on the server with our port details to do a blocking wait until the process is killed or `Stop()` is called. 使用端口详细信息调用服务器的 `Serve()` 方法，进行阻塞等待，直到进程被终止或调用 `Stop()`。

### Creating the client

In this section, we’ll look at creating a Go client for our `RouteGuide` service. You can see our complete example client code in [grpc-go/examples/route_guide/client/client.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/client/client.go).

​	接下来，我们将创建一个用于 `RouteGuide` 服务的 Go 客户端。完整的示例客户端代码可以在 [grpc-go/examples/route_guide/client/client.go](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/client/client.go) 中找到。

#### 创建存根 Creating a stub

To call service methods, we first need to create a gRPC *channel* to communicate with the server. We create this by passing the server address and port number to `grpc.NewClient()` as follows:

​	为了调用服务方法，我们首先需要创建一个 gRPC *通道* 与服务器通信。通过将服务器地址和端口号传递给 `grpc.NewClient()` 可以创建：

```go
var opts []grpc.DialOption
...
conn, err := grpc.NewClient(*serverAddr, opts...)
if err != nil {
  ...
}
defer conn.Close()
```

You can use `DialOptions` to set the auth credentials (for example, TLS, GCE credentials, or JWT credentials) in `grpc.NewClient` when a service requires them. The `RouteGuide` service doesn’t require any credentials.

​	如果服务需要身份验证，可以在 `grpc.NewClient` 中使用 `DialOptions` 设置身份验证凭据（例如 TLS、GCE 凭据或 JWT 凭据）。`RouteGuide` 服务不需要任何凭据。

Once the gRPC *channel* is setup, we need a client *stub* to perform RPCs. We get it using the `NewRouteGuideClient` method provided by the `pb` package generated from the example `.proto` file.

​	设置 gRPC *通道* 后，我们需要一个客户端 *存根* 来执行 RPC。通过示例 `.proto` 文件生成的 `pb` 包中提供的 `NewRouteGuideClient` 方法可以获取：

```go
client := pb.NewRouteGuideClient(conn)
```

#### 调用服务方法 Calling service methods

Now let’s look at how we call our service methods. Note that in gRPC-Go, RPCs operate in a blocking/synchronous mode, which means that the RPC call waits for the server to respond, and will either return a response or an error.

​	接下来，我们来看如何调用服务方法。请注意，在 gRPC-Go 中，RPC 以阻塞/同步模式运行，即 RPC 调用会等待服务器响应，返回响应或错误。

##### Simple RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.

​	调用简单 RPC `GetFeature` 几乎和调用本地方法一样直接。

```go
feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
if err != nil {
  ...
}
```

As you can see, we call the method on the stub we got earlier. In our method parameters we create and populate a request protocol buffer object (in our case `Point`). We also pass a `context.Context` object which lets us change our RPC’s behavior if necessary, such as time-out/cancel an RPC in flight. If the call doesn’t return an error, then we can read the response information from the server from the first return value.

​	如您所见，我们在之前获得的存根上调用了该方法。在方法参数中，我们创建并填充了一个请求的协议缓冲区对象（在本例中为 `Point`）。我们还传递了一个 `context.Context` 对象，以便在需要时更改 RPC 的行为，例如超时/取消正在执行的 RPC。如果调用没有返回错误，我们可以通过返回值读取来自服务器的响应信息。

```go
log.Println(feature)
```

##### Server-side streaming RPC

Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s. If you’ve already read [Creating the server](https://grpc.io/docs/languages/go/basics/#server) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides.

​	以下是调用服务器端流式方法 `ListFeatures` 的示例，该方法返回地理 `Feature` 的流。如果您已经阅读了 [创建服务器](https://grpc.io/docs/languages/go/basics/#server)，这部分内容可能会显得很熟悉——流式 RPC 在两端的实现方式非常相似。

```go
rect := &pb.Rectangle{ ... }  // initialize a pb.Rectangle 初始化一个 pb.Rectangle
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

​	与简单 RPC 类似，我们向方法传递了一个上下文和一个请求。不过，这次返回的不是一个响应对象，而是一个 `RouteGuide_ListFeaturesClient` 实例。客户端可以使用 `RouteGuide_ListFeaturesClient` 流读取服务器的响应。

We use the `RouteGuide_ListFeaturesClient`’s `Recv()` method to repeatedly read in the server’s responses to a response protocol buffer object (in this case a `Feature`) until there are no more messages: the client needs to check the error `err` returned from `Recv()` after each call. If `nil`, the stream is still good and it can continue reading; if it’s `io.EOF` then the message stream has ended; otherwise there must be an RPC error, which is passed over through `err`.

​	我们使用 `RouteGuide_ListFeaturesClient` 的 `Recv()` 方法反复读取服务器的响应到响应协议缓冲区对象（在本例中为 `Feature`），直到没有更多消息：客户端需要在每次调用后检查 `Recv()` 返回的错误 `err`。如果为 `nil`，则流仍然有效，可以继续读取；如果为 `io.EOF`，则消息流已结束；否则可能是 RPC 错误，通过 `err` 返回。

##### Client-side streaming RPC

The client-side streaming method `RecordRoute` is similar to the server-side method, except that we only pass the method a context and get a `RouteGuide_RecordRouteClient` stream back, which we can use to both write *and* read messages.

​	客户端流式方法 `RecordRoute` 与服务器端方法类似，但我们只需传递一个上下文给方法，并获取一个 `RouteGuide_RecordRouteClient` 流，该流可用于同时写入和读取消息。

```go
// Create a random number of random points
// 创建随机数量的随机点
r := rand.New(rand.NewSource(time.Now().UnixNano()))
pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points 至少遍历两个点
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

​	`RouteGuide_RecordRouteClient` 提供了一个 `Send()` 方法，可以用来向服务器发送请求。完成向流中写入请求后，我们需要调用流的 `CloseAndRecv()` 方法，让 gRPC 知道我们已经完成了写入，并期待接收一个响应。我们从 `CloseAndRecv()` 返回的 `err` 中获取 RPC 状态。如果状态为 `nil`，则 `CloseAndRecv()` 的第一个返回值将是有效的服务器响应。

##### Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. As in the case of `RecordRoute`, we only pass the method a context object and get back a stream that we can use to both write and read messages. However, this time we return values via our method’s stream while the server is still writing messages to *their* message stream.

​	最后，我们来看双向流式 RPC `RouteChat()`。与 `RecordRoute` 的情况类似，我们只需传递一个上下文对象给方法，并获取一个流，该流可用于同时写入和读取消息。不过，这次我们在方法流中返回值，而服务器仍在其消息流中写入消息。

```go
stream, err := client.RouteChat(context.Background())
waitc := make(chan struct{})
go func() {
  for {
    in, err := stream.Recv()
    if err == io.EOF {
      // read done.
      // 读取完成
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

​	这里的读取和写入语法与客户端流式方法非常相似，但我们在完成调用后使用流的 `CloseSend()` 方法。尽管双方始终按写入顺序接收对方的消息，但客户端和服务器可以以任意顺序读取和写入——流是完全独立运行的。

### Try it out!

Execute the following commands from the `examples/route_guide` directory:

​	从 `examples/route_guide` 目录执行以下命令：

1. Run the server:

   ```sh
   $ go run server/server.go
   ```

2. From another terminal, run the client: 在另一个终端启动客户端：

   ```sh
   $ go run client/client.go
   ```

You’ll see output like this:

​	您将看到类似以下的输出：

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

> Note
>
> We’ve omitted timestamps from the client and server trace output shown in this page.
>
> ​	我们已省略此页面中客户端和服务器跟踪输出的时间戳。
