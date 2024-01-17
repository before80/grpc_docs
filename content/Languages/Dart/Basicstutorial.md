+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/dart/basics/](https://grpc.io/docs/languages/dart/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Dart.

​	Dart 中 gRPC 的基本教程介绍。



This tutorial provides a basic Dart programmer’s introduction to working with gRPC.

​	本教程为 Dart 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Dart gRPC API to write a simple client and server for your service.
  使用 Dart gRPC API 为您的服务编写一个简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc-dart/example/route_guide](https://github.com/grpc/grpc-dart/tree/master/example/route_guide). To download the example, clone the `grpc-dart` repository by running the following command:

​	本教程的示例代码位于 grpc/grpc-dart/example/route_guide 中。要下载示例，请通过运行以下命令克隆 `grpc-dart` 存储库：

```sh
$ git clone --depth 1 https://github.com/grpc/grpc-dart
```

Then change your current directory to `grpc-dart/example/route_guide`:

​	然后将当前目录更改为 `grpc-dart/example/route_guide` ：

```sh
$ cd grpc-dart/example/route_guide
```

You should have already installed the tools needed to generate client and server interface code – if you haven’t, see [Quick start]({{< ref "/Languages/Dart/Quickstart" >}}) for setup instructions.

​	您应该已经安装了生成客户端和服务器接口代码所需的工具 - 如果您还没有安装，请参阅快速入门以获取设置说明。

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete `.proto` file in [`example/route_guide/protos/route_guide.proto`](https://github.com/grpc/grpc-dart/blob/master/example/route_guide/protos/route_guide.proto).

​	我们的第一步（您会从 gRPC 简介中了解到）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 `example/route_guide/protos/route_guide.proto` 中看到完整的 `.proto` 文件。

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

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special Dart plugin. This is similar to what we did in the [Quick start]({{< ref "/Languages/Dart/Quickstart" >}}).

​	接下来，我们需要从 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。我们使用协议缓冲区编译器 `protoc` 和特殊的 Dart 插件来执行此操作。这与我们在快速入门中所做的事情类似。

From the `route_guide` example directory run:

​	从 `route_guide` 示例目录运行：

```sh
protoc -I protos/ protos/route_guide.proto --dart_out=grpc:lib/src/generated
```

Running this command generates the following files in the `lib/src/generated` directory under the `route_guide` example directory:

​	运行此命令会在 `route_guide` 示例目录下的 `lib/src/generated` 目录中生成以下文件：

- `route_guide.pb.dart`
- `route_guide.pbenum.dart`
- `route_guide.pbgrpc.dart`
- `route_guide.pbjson.dart`

This contains: 
​	其中包含：

- All the protocol buffer code to populate, serialize, and retrieve our request and response message types
  填充、序列化和检索我们的请求和响应消息类型的所有协议缓冲区代码
- An interface type (or *stub*) for clients to call with the methods defined in the `RouteGuide` service.
  一个接口类型（或存根）供客户端使用在 `RouteGuide` 服务中定义的方法进行调用。
- An interface type for servers to implement, also with the methods defined in the `RouteGuide` service.
  服务器要实现的接口类型，还包括在 `RouteGuide` 服务中定义的方法。

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Dart/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  实现从服务定义中生成的服务接口：执行服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and dispatch them to the right service implementation.
  运行 gRPC 服务器以侦听来自客户端的请求并将它们分派到正确的服务实现。

You can find our example `RouteGuide` server in [grpc-dart/example/route_guide/lib/src/server.dart](https://github.com/grpc/grpc-dart/tree/master/example/route_guide/lib/src/server.dart). Let’s take a closer look at how it works.

​	您可以在 grpc-dart/example/route_guide/lib/src/server.dart 中找到我们的示例 `RouteGuide` 服务器。让我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `RouteGuideService` class that extends the generated abstract `RouteGuideServiceBase` class:

​	如您所见，我们的服务器有一个 `RouteGuideService` 类，它扩展了生成的抽象 `RouteGuideServiceBase` 类：

```dart
class RouteGuideService extends RouteGuideServiceBase {
  Future<Feature> getFeature(grpc.ServiceCall call, Point request) async {
    ...
  }

  Stream<Feature> listFeatures(
      grpc.ServiceCall call, Rectangle request) async* {
    ...
  }

  Future<RouteSummary> recordRoute(
      grpc.ServiceCall call, Stream<Point> request) async {
    ...
  }

  Stream<RouteNote> routeChat(
      grpc.ServiceCall call, Stream<RouteNote> request) async* {
    ...
  }

  ...
}
```

##### Simple RPC 简单 RPC

`RouteGuideService` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	 `RouteGuideService` 实现我们所有的服务方法。我们先来看最简单的类型 `GetFeature` ，它只是从客户端获取 `Point` ，并从其数据库中的 `Feature` 中返回相应的要素信息。

```dart
/// GetFeature handler. Returns a feature for the given location.
/// The [context] object provides access to client metadata, cancellation, etc.
@override
Future<Feature> getFeature(grpc.ServiceCall call, Point request) async {
  return featuresDb.firstWhere((f) => f.location == request,
      orElse: () => Feature()..location = request);
}
```

The method is passed a context object for the RPC and the client’s `Point` protocol buffer request. It returns a `Feature` protocol buffer object with the response information. In the method we populate the `Feature` with the appropriate information, and then `return` it to the gRPC framework, which sends it back to the client.

​	该方法传递一个用于 RPC 的上下文对象和客户端的 `Point` 协议缓冲区请求。它返回一个包含响应信息的 `Feature` 协议缓冲区对象。在此方法中，我们使用适当的信息填充 `Feature` ，然后将其 `return` 到 gRPC 框架，该框架会将其发送回客户端。

##### Server-side streaming RPC 服务器端流式 RPC

Now let’s look at one of our streaming RPCs. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	现在，我们来看一下我们的流式 RPC 之一。 `ListFeatures` 是一个服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```dart
/// ListFeatures handler. Returns a stream of features within the given
/// rectangle.
@override
Stream<Feature> listFeatures(
    grpc.ServiceCall call, Rectangle request) async* {
  final normalizedRectangle = _normalize(request);
  // For each feature, check if it is in the given bounding box
  for (var feature in featuresDb) {
    if (feature.name.isEmpty) continue;
    final location = feature.location;
    if (_contains(normalizedRectangle, location)) {
      yield feature;
    }
  }
}
```

As you can see, instead of getting and returning simple request and response objects in our method, this time we get a request object (the `Rectangle` in which our client wants to find `Feature`s) and return a `Stream` of `Feature` objects.

​	如您所见，这次我们不是在方法中获取和返回简单的请求和响应对象，而是获取一个请求对象（客户端希望在其中查找 `Feature` 的 `Rectangle` ）并返回一个 `Stream` 的 `Feature` 对象。

In the method, we populate as many `Feature` objects as we need to return, adding them to the returned stream using `yield`. The stream is automatically closed when the method returns, telling gRPC that we have finished writing responses.

​	在此方法中，我们填充需要返回的尽可能多的 `Feature` 对象，并使用 `yield` 将它们添加到返回的流中。当方法返回时，流会自动关闭，这会告诉 gRPC 我们已完成写入响应。

Should any error happen in this call, the error will be added as an exception to the stream, and the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

​	如果此调用中发生任何错误，该错误将作为异常添加到流中，gRPC 层会将其转换为适当的 RPC 状态以在网络上发送。

##### Client-side streaming RPC 客户端流式 RPC

Now let’s look at something a little more complicated: the client-side streaming method `RecordRoute`, where we get a stream of `Point`s from the client and return a single `RouteSummary` with information about their trip. As you can see, this time the request parameter is a stream, which the server can use to both read request messages from the client. The server returns its single response just like in the simple RPC case.

​	现在，我们来看一些更复杂的内容：客户端流式方法 `RecordRoute` ，其中我们从客户端获取 `Point` 流并返回一个包含其行程信息的 `RouteSummary` 。如您所见，这次请求参数是一个流，服务器可以使用它从客户端读取请求消息。服务器返回其单个响应，就像在简单 RPC 案例中一样。

```dart
/// RecordRoute handler. Gets a stream of points, and responds with statistics
/// about the "trip": number of points, number of known features visited,
/// total distance traveled, and total time spent.
@override
Future<RouteSummary> recordRoute(
    grpc.ServiceCall call, Stream<Point> request) async {
  int pointCount = 0;
  int featureCount = 0;
  double distance = 0.0;
  Point previous;
  final timer = Stopwatch();

  await for (var location in request) {
    if (!timer.isRunning) timer.start();
    pointCount++;
    final feature = featuresDb.firstWhere((f) => f.location == location,
        orElse: () => null);
    if (feature != null) {
      featureCount++;
    }
    // For each point after the first, add the incremental distance from the
    // previous point to the total distance value.
    if (previous != null) distance += _distance(previous, location);
    previous = location;
  }
  timer.stop();
  return RouteSummary()
    ..pointCount = pointCount
    ..featureCount = featureCount
    ..distance = distance.round()
    ..elapsedTime = timer.elapsed.inSeconds;
}
```

In the method body we use `await for` in the request stream to repeatedly read in our client’s requests (in this case `Point` objects) until there are no more messages. Once the request stream is done, the server can return its `RouteSummary`.

​	在方法体中，我们在请求流中使用 `await for` 重复读取客户端的请求（在本例中为 `Point` 对象），直到没有更多消息。请求流完成后，服务器可以返回其 `RouteSummary` 。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```dart
/// RouteChat handler. Receives a stream of message/location pairs, and
/// responds with a stream of all previous messages at each of those
/// locations.
@override
Stream<RouteNote> routeChat(
    grpc.ServiceCall call, Stream<RouteNote> request) async* {
  await for (var note in request) {
    final notes = routeNotes.putIfAbsent(note.location, () => <RouteNote>[]);
    for (var note in notes) yield note;
    notes.add(note);
  }
}
```

This time we get a stream of `RouteNote` that, as in our client-side streaming example, can be used to read messages. However, this time we return values via our method’s returned stream while the client is still writing messages to *their* message stream.

​	这次我们得到一个 `RouteNote` 流，与我们的客户端流式示例一样，可用于读取消息。但是，这次我们在客户端仍向其消息流写入消息时通过方法的返回流返回值。

The syntax for reading and writing here is the same as our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	这里的读写语法与我们的客户端流和服务器流方法相同。尽管每一方总是按写入顺序接收另一方的消息，但客户端和服务器都可以按任何顺序读写——流完全独立地运行。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```dart
Future<void> main(List<String> args) async {
  final server = grpc.Server.create([RouteGuideService()]);
  await server.serve(port: 8080);
  print('Server listening...');
}
```

To build and start a server, we:

​	要构建并启动服务器，我们：

1. Create an instance of the gRPC server using `grpc.Server.create()`, giving a list of service implementations.
   使用 `grpc.Server.create()` 创建 gRPC 服务器实例，提供服务实现列表。
2. Call `serve()` on the server to start listening for requests, optionally passing in the address and port to listen on. The server will continue to serve requests asynchronously until `shutdown()` is called on it.
   在服务器上调用 `serve()` 以开始侦听请求，可以选择传入要侦听的地址和端口。服务器将继续异步提供请求，直到对其调用 `shutdown()` 。

### Creating the client 创建客户端

In this section, we’ll look at creating a Dart client for our `RouteGuide` service. The complete client code is available from [grpc-dart/example/route_guide/lib/src/client.dart](https://github.com/grpc/grpc-dart/tree/master/example/route_guide/lib/src/client.dart).

​	在本节中，我们将着眼于为我们的 `RouteGuide` 服务创建一个 Dart 客户端。完整的客户端代码可从 grpc-dart/example/route_guide/lib/src/client.dart 获取。

#### Creating a stub 创建存根

To call service methods, we first need to create a gRPC *channel* to communicate with the server. We create this by passing the server address and port number to `ClientChannel()` as follows:

​	要调用服务方法，我们首先需要创建一个 gRPC 通道来与服务器通信。我们通过将服务器地址和端口号传递给 `ClientChannel()` 来创建此通道，如下所示：

```dart
final channel = ClientChannel('127.0.0.1',
    port: 8080,
    options: const ChannelOptions(
        credentials: ChannelCredentials.insecure()));
```

You can use `ChannelOptions` to set TLS options (for example, trusted certificates) for the channel, if necessary.

​	如果需要，您可以使用 `ChannelOptions` 为通道设置 TLS 选项（例如，受信任的证书）。

Once the gRPC *channel* is setup, we need a client *stub* to perform RPCs. We get it by instantiating `RouteGuideClient`, which is provided by the package generated from the example `.proto` file.

​	一旦 gRPC 通道设置好，我们就需要一个客户端存根来执行 RPC。我们可以通过实例化 `RouteGuideClient` 来获取它，该存根由从示例 `.proto` 文件生成的包提供。

```dart
stub = RouteGuideClient(channel,
    options: CallOptions(timeout: Duration(seconds: 30)));
```

You can use `CallOptions` to set auth credentials (for example, GCE credentials or JWT credentials) when a service requires them. The `RouteGuide` service doesn’t require any credentials.

​	当服务需要时，您可以使用 `CallOptions` 设置身份验证凭据（例如，GCE 凭据或 JWT 凭据）。 `RouteGuide` 服务不需要任何凭据。

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. Note that in gRPC-Dart, RPCs are always asynchronous, which means that the RPC returns a `Future` or `Stream` that must be listened to, to get the response from the server or an error.

​	现在，我们来看看如何调用我们的服务方法。请注意，在 gRPC-Dart 中，RPC 始终是异步的，这意味着 RPC 会返回一个 `Future` 或 `Stream` ，必须对其进行侦听，才能从服务器或错误中获取响应。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.

​	调用简单 RPC `GetFeature` 几乎与调用本地方法一样简单。

```dart
final point = Point()
  ..latitude = 409146138
  ..longitude = -746188906;
final feature = await stub.getFeature(point));
```

As you can see, we call the method on the stub we got earlier. In our method parameters we pass a request protocol buffer object (in our case `Point`). We can also pass an optional `CallOptions` object which lets us change our RPC’s behavior if necessary, such as time-out. If the call doesn’t return an error, the returned `Future` completes with the response information from the server. If there is an error, the `Future` will complete with the error.

​	如您所见，我们在之前获得的存根上调用该方法。在我们的方法参数中，我们传递一个请求协议缓冲区对象（在本例中为 `Point` ）。我们还可以传递一个可选的 `CallOptions` 对象，该对象允许我们在必要时更改 RPC 的行为，例如超时。如果调用没有返回错误，则返回的 `Future` 将使用来自服务器的响应信息完成。如果出现错误，则 `Future` 将使用该错误完成。

##### Server-side streaming RPC 服务器端流式 RPC

Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s. If you’ve already read [Creating the server]({{< ref "/Languages/Dart/Basicstutorial#server" >}}) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides.

​	我们在此调用服务器端流式方法 `ListFeatures` ，该方法返回一个地理 `Feature` 流。如果您已经阅读了创建服务器，其中一些内容可能看起来非常熟悉 - 流式 RPC 在双方以类似的方式实现。

```dart
final rect = Rectangle()...; // initialize a Rectangle

try {
  await for (var feature in stub.listFeatures(rect)) {
    print(feature);
  }
catch (e) {
  print('ERROR: $e');
}
```

As in the simple RPC, we pass the method a request. However, instead of getting a `Future` back, we get a `Stream`. The client can use the stream to read the server’s responses.

​	与简单的 RPC 一样，我们将方法传递给请求。但是，我们不会收到 `Future` ，而是收到 `Stream` 。客户端可以使用流来读取服务器的响应。

We use `await for` on the returned stream to repeatedly read in the server’s responses to a response protocol buffer object (in this case a `Feature`) until there are no more messages.

​	我们对返回的流使用 `await for` ，以便反复读取服务器对响应协议缓冲区对象（在本例中为 `Feature` ）的响应，直到没有更多消息为止。

##### Client-side streaming RPC 客户端流式 RPC

The client-side streaming method `RecordRoute` is similar to the server-side method, except that we pass the method a `Stream` and get a `Future` back.

​	客户端流式方法 `RecordRoute` 与服务器端方法类似，不同之处在于我们将方法传递给 `Stream` 并收到 `Future` 。

```dart
final random = Random();

// Generate a number of random points
Stream<Point> generateRoute(int count) async* {
  for (int i = 0; i < count; i++) {
    final point = featuresDb[random.nextInt(featuresDb.length)].location;
    yield point;
  }
}

final pointCount = random.nextInt(100) + 2; // Traverse at least two points

final summary = await stub.recordRoute(generateRoute(pointCount));
print('Route summary: $summary');
```

Since the `generateRoute()` method is `async*`, the points will be generated when gRPC listens to the request stream and sends the point messages to the server. Once the stream is done (when `generateRoute()` returns), gRPC knows that we’ve finished writing and are expecting to receive a response. The returned `Future` will either complete with the `RouteSummary` message received from the server, or an error.

​	由于 `generateRoute()` 方法是 `async*` ，因此当 gRPC 侦听请求流并将点消息发送到服务器时，将生成点。一旦流完成（当 `generateRoute()` 返回时），gRPC 便知道我们已完成写入并期望收到响应。返回的 `Future` 将使用从服务器收到的 `RouteSummary` 消息或错误完成。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. As in the case of `RecordRoute`, we pass the method a stream where we will write the request messages, and like in `ListFeatures`, we get back a stream that we can use to read the response messages. However, this time we will send values via our method’s stream while the server is also writing messages to *their* message stream.

​	最后，我们来看一下我们的双向流式 RPC `RouteChat()` 。与 `RecordRoute` 的情况一样，我们将方法传递给一个流，我们将在其中写入请求消息，就像在 `ListFeatures` 中一样，我们返回一个流，我们可以用它来读取响应消息。但是，这次我们将通过我们方法的流发送值，而服务器也会将消息写入它们的消息流。

```dart
Stream<RouteNote> outgoingNotes = ...;

final responses = stub.routeChat(outgoingNotes);
await for (var note in responses) {
  print('Got message ${note.message} at ${note.location.latitude}, ${note
      .location.longitude}');
}
```

The syntax for reading and writing here is very similar to our client-side and server-side streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	这里的读写语法与我们的客户端和服务器端流式方法非常相似。尽管每一方总是按写入顺序获取另一方的消息，但客户端和服务器都可以按任何顺序读取和写入——流完全独立地运行。

### Try it out! 试一试！

Work from the example directory:

​	从示例目录中工作：

```sh
$ cd example/route_guide
```

Get packages: 
​	获取软件包：

```sh
$ dart pub get
```

Run the server: 
​	运行服务器：

```sh
$ dart bin/server.dart
```

From a different terminal, run the client:

​	从另一个终端运行客户端：

```sh
$ dart bin/client.dart
```

### Reporting issues 报告问题

If you find a problem with Dart gRPC, please [file an issue](https://github.com/grpc/grpc-dart/issues/new) in our issue tracker.

​	如果您发现 Dart gRPC 有问题，请在我们的问题跟踪器中提交问题。
