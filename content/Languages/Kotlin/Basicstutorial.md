+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/kotlin/basics/](https://grpc.io/docs/languages/kotlin/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Kotlin.

​	gRPC 在 Kotlin 中的基本教程介绍。



This tutorial provides a basic Kotlin programmer’s introduction to working with gRPC.

​	本教程为 Kotlin 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Kotlin gRPC API to write a simple client and server for your service.
  使用 Kotlin gRPC API 为您的服务编写一个简单的客户端和服务器。

You should already be familiar gRPC and protocol buffers; if not, see [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and the proto3 [Language guide](https://protobuf.dev/programming-guides/proto3).

​	您应该已经熟悉 gRPC 和协议缓冲区；如果不熟悉，请参阅 gRPC 简介和 proto3 语言指南。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Setup 设置

This tutorial has the same [prerequisites]({{< ref "/Languages/Kotlin/Quickstart#prerequisites" >}}) as the [Quick start]({{< ref "/Languages/Kotlin/Quickstart" >}}). Install the necessary SDKs and tools before proceeding.

​	本教程具有与快速入门相同的先决条件。在继续之前，请安装必要的 SDK 和工具。

### Get the example code 获取示例代码

The example code is part of the [grpc-kotlin](https://github.com/grpc/grpc-kotlin) repo.

​	示例代码是 grpc-kotlin 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-kotlin/archive/master.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone --depth 1 https://github.com/grpc/grpc-kotlin
   ```

2. Change to the examples directory:

   ​	更改到 examples 目录：

   ```sh
   $ cd grpc-kotlin/examples
   ```

### Defining the service 定义服务

Your first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/programming-guides/proto3).

​	您的第一步（您会从 gRPC 简介中了解到）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。

If you’d like to follow along by looking at the complete `.proto` file, see `routeguide/route_guide.proto` from the [protos/src/main/proto/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/protos/src/main/proto/io/grpc/examples) folder.

​	如果您想通过查看完整的 `.proto` 文件来了解，请参阅 protos/src/main/proto/io/grpc/examples 文件夹中的 `routeguide/route_guide.proto` 。

To define a service, you specify a named `service` in the `.proto` file like this:

​	要定义服务，您需要在 `.proto` 文件中指定一个命名的 `service` ，如下所示：

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

The `.proto` file also contains protocol buffer message type definitions for all the request and response types used by the service methods – for example, here’s the `Point` message type:

​	 `.proto` 文件还包含服务方法使用的所有请求和响应类型的协议缓冲区消息类型定义 - 例如，以下是 `Point` 消息类型：

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

Next, you need to generate the gRPC client and server interfaces from the `.proto` service definition. You do this using the protocol buffer compiler `protoc` with special gRPC Kotlin and Java plugins.

​	接下来，您需要从 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。您可以使用协议缓冲区编译器 `protoc` 和特殊的 gRPC Kotlin 和 Java 插件来执行此操作。

When using Gradle or Maven, the `protoc` build plugin will generate the necessary code as part of the build process. For a Gradle example, see [stub/build.gradle.kts](https://github.com/grpc/grpc-kotlin/blob/master/examples/stub/build.gradle.kts).

​	使用 Gradle 或 Maven 时， `protoc` 构建插件会将必要的代码作为构建过程的一部分生成。有关 Gradle 示例，请参阅 stub/build.gradle.kts。

If you run `./gradlew installDist` from the examples folder, the following files are generated from the service definition – you’ll find the generated files in subdirectories below `stub/build/generated/source/proto/main`:

​	如果从 examples 文件夹运行 `./gradlew installDist` ，则会从服务定义中生成以下文件 - 您会在 `stub/build/generated/source/proto/main` 下的子目录中找到生成的文件：

- `Feature.java`, `Point.java`, `Rectangle.java`, and others, which contain all the protocol buffer code to populate, serialize, and retrieve our request and response message types.

  ​	 `Feature.java` 、 `Point.java` 、 `Rectangle.java` 等，其中包含用于填充、序列化和检索我们的请求和响应消息类型的全部协议缓冲区代码。

  You’ll find these files in the `java/io/grpc/examples/routeguide` subdirectory.

  ​	您会在 `java/io/grpc/examples/routeguide` 子目录中找到这些文件。

- `RouteGuideOuterClassGrpcKt.kt`, which contains, among other things:

  ​	 `RouteGuideOuterClassGrpcKt.kt` ，其中包含以下内容：

  - `RouteGuideGrpcKt.RouteGuideCoroutineImplBase`, an abstract base class for `RouteGuide` servers to implement, with all the methods defined in the `RouteGuide` service.
    `RouteGuideGrpcKt.RouteGuideCoroutineImplBase` ， `RouteGuide` 服务器实现的抽象基类，其中包含 `RouteGuide` 服务中定义的所有方法。
  - `RouteGuideGrpcKt.RouteGuideCoroutineStub`, a class that clients use to talk to a `RouteGuide` server.
    `RouteGuideGrpcKt.RouteGuideCoroutineStub` ，一个客户端用来与 `RouteGuide` 服务器通信的类。

  You’ll find this Kotlin file under `grpckt/io/grpc/examples/routeguide`.

  ​	您可以在 `grpckt/io/grpc/examples/routeguide` 下找到此 Kotlin 文件。

### Creating the server 创建服务器

First consider how to create a `RouteGuide` server. If you’re only interested in creating gRPC clients, skip ahead to [Creating the client]({{< ref "/Languages/Kotlin/Basicstutorial#client" >}}) – though you might find this section interesting anyway!

​	首先考虑如何创建 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，请跳至创建客户端 - 尽管您可能仍然会觉得此部分很有趣！

There are two main things that you need to do when creating a `RouteGuide` server:

​	在创建 `RouteGuide` 服务器时，您需要做两件主要的事情：

- Extend the `RouteGuideCoroutineImplBase` service base class to do the actual service work.
  扩展 `RouteGuideCoroutineImplBase` 服务基类以执行实际的服务工作。
- Create and run a gRPC server to listen for requests from clients and return the service responses.
  创建并运行一个 gRPC 服务器来侦听来自客户端的请求并返回服务响应。

Open the example `RouteGuide` server code in `routeguide/RouteGuideServer.kt` under the [server/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/server/src/main/kotlin/io/grpc/examples) folder.

​	在 server/src/main/kotlin/io/grpc/examples 文件夹下打开示例 `RouteGuide` 服务器代码。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, the server has a `RouteGuideService` class that extends the generated service base class:

​	如您所见，服务器有一个 `RouteGuideService` 类，它扩展了生成的 service 基类：

```kotlin
class RouteGuideService(
  val features: Collection<Feature>,
  /* ... */
) : RouteGuideGrpcKt.RouteGuideCoroutineImplBase() {
  /* ... */
}
```

##### Simple RPC 简单 RPC

`RouteGuideService` implements all the service methods. Consider the simplest method first, `GetFeature()`, which gets a `Point` from the client and returns a `Feature` built from the corresponding feature information in the database.

​	 `RouteGuideService` 实现所有服务方法。首先考虑最简单的方法 `GetFeature()` ，它从客户端获取一个 `Point` ，并从数据库中相应的功能信息构建一个 `Feature` 并返回。

```kotlin
override suspend fun getFeature(request: Point): Feature =
    features.find { it.location == request } ?:
    // No feature was found, return an unnamed feature.
    Feature.newBuilder().apply { location = request }.build()
```

The method accepts a client’s `Point` message request as a parameter, and it returns a `Feature` message as a response. The method populates the `Feature` with the appropriate information, and then returns it to the gRPC framework, which sends it back to the client.

​	该方法接受客户端的 `Point` 消息请求作为参数，并返回一个 `Feature` 消息作为响应。该方法使用适当的信息填充 `Feature` ，然后将其返回给 gRPC 框架，后者将其发送回客户端。

##### Server-side streaming RPC 服务器端流式 RPC

Next, consider one of the streaming RPCs. `ListFeatures()` is a server-side streaming RPC, so the server gets to send back multiple `Feature` messages to the client.

​	接下来，考虑一个流式 RPC。 `ListFeatures()` 是一个服务器端流式 RPC，因此服务器可以向客户端发送多个 `Feature` 消息。

```kotlin
override fun listFeatures(request: Rectangle): Flow<Feature> =
  features.asFlow().filter { it.exists() && it.location in request }
```

The request object is a `Rectangle`. The server collects, and returns to the client, all the `Feature` objects in its collection that are inside the given `Rectangle`.

​	请求对象是一个 `Rectangle` 。服务器收集并向客户端返回其集合中位于给定 `Rectangle` 内的所有 `Feature` 对象。

##### Client-side streaming RPC 客户端流式 RPC

Now consider something a little more complicated: the client-side streaming method `RecordRoute()`, where the server gets a stream of `Point` objects from the client, and returns a single `RouteSummary` with information about their trip through the given points.

​	现在考虑一些更复杂的内容：客户端流式传输方法 `RecordRoute()` ，其中服务器从客户端获取 `Point` 对象的流，并返回一个包含有关其通过给定点的行程的信息的 `RouteSummary` 。

```kotlin
override suspend fun recordRoute(requests: Flow<Point>): RouteSummary {
  var pointCount = 0
  var featureCount = 0
  var distance = 0
  var previous: Point? = null
  val stopwatch = Stopwatch.createStarted(ticker)
  requests.collect { request ->
    pointCount++
    if (getFeature(request).exists()) {
      featureCount++
    }
    val prev = previous
    if (prev != null) {
      distance += prev distanceTo request
    }
    previous = request
  }
  return RouteSummary.newBuilder().apply {
    this.pointCount = pointCount
    this.featureCount = featureCount
    this.distance = distance
    this.elapsedTime = Durations.fromMicros(stopwatch.elapsed(TimeUnit.MICROSECONDS))
  }.build()
}
```

The request parameter is a stream of client request messages represented as a Kotlin [Flow](https://kotlinlang.org/docs/reference/coroutines/flow.html#flows). The server returns a single response just like in the simple RPC case.

​	请求参数是表示为 Kotlin Flow 的客户端请求消息流。服务器返回一个响应，就像在简单的 RPC 案例中一样。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, consider the bidirectional streaming RPC `RouteChat()`.

​	最后，考虑双向流式传输 RPC `RouteChat()` 。

```kotlin
override fun routeChat(requests: Flow<RouteNote>): Flow<RouteNote> =
  flow {
    // could use transform, but it's currently experimental
    requests.collect { note ->
      val notes: MutableList<RouteNote> = routeNotes.computeIfAbsent(note.location) {
        Collections.synchronizedList(mutableListOf<RouteNote>())
      }
      for (prevNote in notes.toTypedArray()) { // thread-safe snapshot
        emit(prevNote)
      }
      notes += note
    }
  }
```

Similar to the client-side streaming example, for this method, the server gets a stream of `RouteNote` objects as a `Flow`. However, this time the server returns `RouteNote` instances via the method’s returned stream *while* the client is still writing messages to *its* message stream.

​	与客户端流式传输示例类似，对于此方法，服务器将 `RouteNote` 对象的流作为 `Flow` 获取。但是，这次服务器通过方法返回的流返回 `RouteNote` 实例，而客户端仍在向其消息流写入消息。

#### Starting the server 启动服务器

Once all the server’s methods are implemented, you need code to create a gRPC server instance, something like this:

​	一旦实现服务器的所有方法，您需要代码来创建 gRPC 服务器实例，如下所示：

```kotlin
class RouteGuideServer(
    val port: Int,
    val features: Collection<Feature> = Database.features(),
    val server: Server =
      ServerBuilder.forPort(port)
        .addService(RouteGuideService(features)).build()
) {

  fun start() {
    server.start()
    println("Server started, listening on $port")
    /* ... */
  }
  /* ... */
}

fun main(args: Array<String>) {
  val port = 8980
  val server = RouteGuideServer(port)
  server.start()
  server.awaitTermination()
 }
```

A server instance is built and started using a `ServerBuilder` as follows:

​	服务器实例构建并使用 `ServerBuilder` 启动，如下所示：

1. Specify the port, that the server will listen for client requests on, using `forPort()`.
   使用 `forPort()` 指定服务器将侦听客户端请求的端口。
2. Create an instance of the service implementation class `RouteGuideService` and pass it to the builder’s `addService()` method.
   创建服务实现类的实例 `RouteGuideService` 并将其传递给构建器的 `addService()` 方法。
3. Call `build()` and `start()` on the builder to create and start an RPC server for the route guide service.
   在构建器上调用 `build()` 和 `start()` 以创建和启动用于路线指南服务的 RPC 服务器。
4. Call `awaitTermination()` on the server to block the main function until the application receives a signal to terminate.
   在服务器上调用 `awaitTermination()` 以阻止主函数，直到应用程序收到终止信号。

### Creating the client 创建客户端

In this section, you’ll look at a client for the `RouteGuide` service.

​	在本部分中，您将查看 `RouteGuide` 服务的客户端。

For the complete client code, open `routeguide/RouteGuideClient.kt` under the [client/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/client/src/main/kotlin/io/grpc/examples) folder.

​	对于完整的客户端代码，请在 client/src/main/kotlin/io/grpc/examples 文件夹下打开 `routeguide/RouteGuideClient.kt` 。

#### Instantiating a stub 实例化存根

To call service methods, you first need to create a gRPC *channel* using a `ManagedChannelBuilder`. You’ll use this channel to communicate with the server.

​	要调用服务方法，您首先需要使用 `ManagedChannelBuilder` 创建一个 gRPC 通道。您将使用此通道与服务器通信。

```kotlin
val channel = ManagedChannelBuilder.forAddress("localhost", 8980).usePlaintext().build()
```

Once the gRPC channel is setup, you need a client *stub* to perform RPCs. Get it by instantiating `RouteGuideCoroutineStub`, which is available from the package that was generated from the `.proto` file.

​	设置 gRPC 通道后，您需要一个客户端存根来执行 RPC。通过实例化 `RouteGuideCoroutineStub` 获取它，该存根可从由 `.proto` 文件生成的包中获得。

```kotlin
val stub = RouteGuideCoroutineStub(channel)
```

#### Calling service methods 调用服务方法

Now consider how you’ll call service methods.

​	现在考虑您将如何调用服务方法。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature()` is as straightforward as calling a local method:

​	调用简单 RPC `GetFeature()` 与调用本地方法一样简单：

```kotlin
val request = point(latitude, longitude)
val feature = stub.getFeature(request)
```

The stub method `getFeature()` executes the corresponding RPC, suspending until the RPC completes:

​	存根方法 `getFeature()` 执行相应的 RPC，挂起直到 RPC 完成：

```kotlin
suspend fun getFeature(latitude: Int, longitude: Int) {
  val request = point(latitude, longitude)
  val feature = stub.getFeature(request)
  if (feature.exists()) { /* ... */ }
}
```

##### Server-side streaming RPC 服务器端流式 RPC

Next, consider the server-side streaming `ListFeatures()` RPC, which returns a stream of geographical features:

​	接下来，考虑返回地理特征流的服务器端流式 `ListFeatures()` RPC：

```kotlin
suspend fun listFeatures(lowLat: Int, lowLon: Int, hiLat: Int, hiLon: Int) {
  val request = Rectangle.newBuilder()
    .setLo(point(lowLat, lowLon))
    .setHi(point(hiLat, hiLon))
    .build()
  var i = 1
  stub.listFeatures(request).collect { feature ->
    println("Result #${i++}: $feature")
  }
}
```

The stub `listFeatures()` method returns a stream of features in the form of a `Flow<Feature>` instance. The flow `collect()` method allows the client to processes the server-provided features as they become available.

​	存根 `listFeatures()` 方法以 `Flow<Feature>` 实例的形式返回特征流。流 `collect()` 方法允许客户端在服务器提供的特征可用时对其进行处理。

##### Client-side streaming RPC 客户端流式 RPC

The client-side streaming `RecordRoute()` RPC sends a stream of `Point` messages to the server and gets back a single `RouteSummary`.

​	客户端流式 `RecordRoute()` RPC 向服务器发送 `Point` 消息流，并获取单个 `RouteSummary` 。

```kotlin
suspend fun recordRoute(points: Flow<Point>) {
  println("*** RecordRoute")
  val summary = stub.recordRoute(points)
  println("Finished trip with ${summary.pointCount} points.")
  println("Passed ${summary.featureCount} features.")
  println("Travelled ${summary.distance} meters.")
  val duration = summary.elapsedTime.seconds
  println("It took $duration seconds.")
}
```

The method generates the route points from the points associated with a randomly selected list of features. The random selection is taken from a previously loaded feature collection:

​	该方法从与随机选择的要素列表相关联的点生成路径点。随机选择取自先前加载的要素集合：

```kotlin
fun generateRoutePoints(features: List<Feature>, numPoints: Int): Flow<Point> = flow {
  for (i in 1..numPoints) {
    val feature = features.random(random)
    println("Visiting point ${feature.location.toStr()}")
    emit(feature.location)
    delay(timeMillis = random.nextLong(500L..1500L))
  }
}
```

Note that flow points are emitted lazily, that is, only once the server requests them. Once a point has been emitted to the flow, the point generator suspends until the server requests the next point.

​	请注意，流点是延迟发出的，即仅在服务器请求它们时才发出。一旦将点发送到流，点生成器就会挂起，直到服务器请求下一个点。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, consider the bidirectional streaming RPC `RouteChat()`. As in the case of `RecordRoute()`, you pass to the stub method a stream that you use to write the request messages to; like in `ListFeatures()`, you get back a stream that you can use to read response messages from. However, this time you send values via our method’s stream while the server is also writing messages to *its* message stream.

​	最后，考虑双向流式 RPC `RouteChat()` 。与 `RecordRoute()` 的情况一样，您将流传递给存根方法，您使用该流将请求消息写入其中；与 `ListFeatures()` 中一样，您会收到一个流，您可以使用该流从其中读取响应消息。但是，这次您通过我们方法的流发送值，而服务器也会将其消息写入其消息流。

```kotlin
suspend fun routeChat() {
  val requests = generateOutgoingNotes()
  stub.routeChat(requests).collect { note ->
    println("Got message \"${note.message}\" at ${note.location.toStr()}")
  }
  println("Finished RouteChat")
}

private fun generateOutgoingNotes(): Flow<RouteNote> = flow {
  val notes = listOf(/* ... */)
  for (note in notes) {
    println("Sending message \"${note.message}\" at ${note.location.toStr()}")
    emit(note)
    delay(500)
  }
}
```

The syntax for reading and writing here is very similar to the client-side and server-side streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order —- the streams operate completely independently.

​	这里的读取和写入语法与客户端和服务器端流方法非常相似。尽管每一方总是会按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序读取和写入——流完全独立地运行。

### Try it out! 试一试！

Run the following commands from the `grpc-kotlin/examples` directory:

​	从 `grpc-kotlin/examples` 目录运行以下命令：

1. Compile the client and server

   ​	编译客户端和服务器

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./server/build/install/server/bin/route-guide-server
   Server started, listening on 8980
   ```

3. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ ./client/build/install/client/bin/route-guide-client
   ```

You’ll see client output like this:

​	您将看到类似这样的客户端输出：

```nocode
*** GetFeature: lat=409146138 lon=-746188906
Found feature called "Berkshire Valley Management Area Trail, Jefferson, NJ, USA" at 40.9146138, -74.6188906
*** GetFeature: lat=0 lon=0
Found no feature at 0.0, 0.0
*** ListFeatures: lowLat=400000000 lowLon=-750000000 hiLat=420000000 liLon=-730000000
Result #1: name: "Patriots Path, Mendham, NJ 07945, USA"
location {
  latitude: 407838351
  longitude: -746143763
}
...
Result #64: name: "3 Hasta Way, Newton, NJ 07860, USA"
location {
  latitude: 410248224
  longitude: -747127767
}

*** RecordRoute
Visiting point 40.0066188, -74.6793294
...
Visiting point 40.4318328, -74.0835638
Finished trip with 10 points.
Passed 3 features.
Travelled 93238790 meters.
It took 9 seconds.
*** RouteChat
Sending message "First message" at 0.0, 0.0
Sending message "Second message" at 0.0, 0.0
Got message "First message" at 0.0, 0.0
Sending message "Third message" at 1.0, 0.0
Sending message "Fourth message" at 1.0, 1.0
Sending message "Last message" at 0.0, 0.0
Got message "First message" at 0.0, 0.0
Got message "Second message" at 0.0, 0.0
Finished RouteChat
```
