+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/java/basics/](https://grpc.io/docs/languages/java/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Java.

​	Java 中 gRPC 的基本教程介绍。



This tutorial provides a basic Java programmer’s introduction to working with gRPC.

​	本教程为基本 Java 程序员提供了使用 gRPC 的入门介绍。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Java gRPC API to write a simple client and server for your service.
  使用 Java gRPC API 为您的服务编写简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the [proto3](https://github.com/google/protobuf/releases) version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and [Java generated code guide](https://protobuf.dev/reference/java/java-generated).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南和 Java 生成的代码指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc-java/examples/src/main/java/io/grpc/examples/routeguide](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/routeguide). To download the example, clone the latest release in `grpc-java` repository by running the following command:

​	本教程的示例代码位于 grpc/grpc-java/examples/src/main/java/io/grpc/examples/routeguide。要下载示例，请通过运行以下命令克隆 `grpc-java` 存储库中的最新版本：

```sh
$ git clone -b v1.61.0 --depth 1 https://github.com/grpc/grpc-java
```

Then change your current directory to `grpc-java/examples`:

​	然后将当前目录更改为 `grpc-java/examples` ：

```sh
$ cd grpc-java/examples
```

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete `.proto` file in [grpc-java/examples/src/main/proto/route_guide.proto](https://github.com/grpc/grpc-java/blob/master/examples/src/main/proto/route_guide.proto).

​	我们的第一步（您将从 gRPC 简介中了解到）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 grpc-java/examples/src/main/proto/route_guide.proto 中看到完整的 `.proto` 文件。

As we’re generating Java code in this example, we’ve specified a `java_package` file option in our `.proto`:

​	由于我们在此示例中生成 Java 代码，因此我们在 `.proto` 中指定了 `java_package` 文件选项：

```proto
option java_package = "io.grpc.examples.routeguide";
```

This specifies the package we want to use for our generated Java classes. If no explicit `java_package` option is given in the `.proto` file, then by default the proto package (specified using the “package” keyword) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If we generate code in another language from this `.proto`, the `java_package` option has no effect.

​	这指定了我们想要用于生成的 Java 类的包。如果在 `.proto` 文件中没有给出明确的 `java_package` 选项，那么默认情况下将使用 proto 包（使用“package”关键字指定）。但是，proto 包通常不会生成良好的 Java 包，因为 proto 包预计不会以反向域名开头。如果我们从此 `.proto` 中生成其他语言的代码，则 `java_package` 选项无效。

To define a service, we specify a named `service` in the `.proto` file:

​	要定义服务，我们在 `.proto` 文件中指定一个命名的 `service` ：

```proto
service RouteGuide {
   ...
}
```

Then we define `rpc` methods inside our service definition, specifying their request and response types. gRPC lets you define four kinds of service methods, all of which are used in the `RouteGuide` service:

​	然后我们在服务定义中定义 `rpc` 方法，指定它们的请求和响应类型。gRPC 允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

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

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC Java plugin. You need to use the [proto3](https://github.com/google/protobuf/releases) compiler (which supports both proto2 and proto3 syntax) in order to generate gRPC services.

​	接下来，我们需要从 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。我们使用协议缓冲区编译器 `protoc` 和特殊的 gRPC Java 插件来执行此操作。您需要使用 proto3 编译器（它支持 proto2 和 proto3 语法）才能生成 gRPC 服务。

When using Gradle or Maven, the protoc build plugin can generate the necessary code as part of the build. You can refer to the [grpc-java README](https://github.com/grpc/grpc-java/blob/master/README.md) for how to generate code from your own `.proto` files.

​	在使用 Gradle 或 Maven 时，protoc 构建插件可以生成必要的代码作为构建的一部分。您可以参考 grpc-java 自述文件，了解如何从您自己的 `.proto` 文件生成代码。

The following classes are generated from our service definition:

​	以下类是从我们的服务定义中生成的：

- `Feature.java`, `Point.java`, `Rectangle.java`, and others which contain all the protocol buffer code to populate, serialize, and retrieve our request and response message types.
  `Feature.java` 、 `Point.java` 、 `Rectangle.java` 等，其中包含用于填充、序列化和检索我们的请求和响应消息类型的所有协议缓冲区代码。

- ```
  RouteGuideGrpc.java
  ```

   

  which contains (along with some other useful code):

  
  `RouteGuideGrpc.java` 其中包含（以及一些其他有用的代码）：

  - a base class for `RouteGuide` servers to implement, `RouteGuideGrpc.RouteGuideImplBase`, with all the methods defined in the `RouteGuide` service.
    一个供 `RouteGuide` 服务器实现的基础类， `RouteGuideGrpc.RouteGuideImplBase` ，其中包含 `RouteGuide` 服务中定义的所有方法。
  - *stub* classes that clients can use to talk to a `RouteGuide` server.
    客户端可用于与 `RouteGuide` 服务器通信的存根类。

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Java/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Overriding the service base class generated from our service definition: doing the actual “work” of our service.
  覆盖从我们的服务定义生成的 service 基类：执行我们服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and return the service responses.
  运行 gRPC 服务器以侦听来自客户端的请求并返回服务响应。

You can find our example `RouteGuide` server in [grpc-java/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideServer.java](https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideServer.java). Let’s take a closer look at how it works.

​	您可以在 grpc-java/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideServer.java 中找到我们的示例 `RouteGuide` 服务器。让我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `RouteGuideService` class that extends the generated `RouteGuideGrpc.RouteGuideImplBase` abstract class:

​	如您所见，我们的服务器有一个 `RouteGuideService` 类，它扩展了生成的 `RouteGuideGrpc.RouteGuideImplBase` 抽象类：

```java
private static class RouteGuideService extends RouteGuideGrpc.RouteGuideImplBase {
...
}
```

##### Simple RPC 简单 RPC

`RouteGuideService` implements all our service methods. Let’s look at the simplest method first, `GetFeature()`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	 `RouteGuideService` 实现我们所有的服务方法。让我们先看看最简单的方法 `GetFeature()` ，它只是从客户端获取一个 `Point` ，并从其数据库中的 `Feature` 中返回相应的要素信息。

```java
@Override
public void getFeature(Point request, StreamObserver<Feature> responseObserver) {
  responseObserver.onNext(checkFeature(request));
  responseObserver.onCompleted();
}

...

private Feature checkFeature(Point location) {
  for (Feature feature : features) {
    if (feature.getLocation().getLatitude() == location.getLatitude()
        && feature.getLocation().getLongitude() == location.getLongitude()) {
      return feature;
    }
  }

  // No feature was found, return an unnamed feature.
  return Feature.newBuilder().setName("").setLocation(location).build();
}
```

The `getFeature()` method takes two parameters:

​	 `getFeature()` 方法采用两个参数：

- `Point`: the request `Point` ：请求
- `StreamObserver<Feature>`: a response observer, which is a special interface for the server to call with its response.
  `StreamObserver<Feature>` ：响应观察器，这是一个特殊的接口，服务器可以使用其响应进行调用。

To return our response to the client and complete the call:

​	要向客户端返回我们的响应并完成调用：

1. We construct and populate a `Feature` response object to return to the client, as specified in our service definition. In this example, we do this in a separate private `checkFeature()` method.
   我们构造并填充一个 `Feature` 响应对象以返回给客户端，如我们的服务定义中所指定。在此示例中，我们在一个单独的私有 `checkFeature()` 方法中执行此操作。
2. We use the response observer’s `onNext()` method to return the `Feature`.
   我们使用响应观察器的 `onNext()` 方法返回 `Feature` 。
3. We use the response observer’s `onCompleted()` method to specify that we’ve finished dealing with the RPC.
   我们使用响应观察者的 `onCompleted()` 方法来指定我们已完成处理 RPC。

##### Server-side streaming RPC 服务器端流式 RPC

Next let’s look at one of our streaming RPCs. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	接下来，我们来看一下我们的一个流式 RPC。 `ListFeatures` 是一个服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```java
private final Collection<Feature> features;

...

@Override
public void listFeatures(Rectangle request, StreamObserver<Feature> responseObserver) {
  int left = min(request.getLo().getLongitude(), request.getHi().getLongitude());
  int right = max(request.getLo().getLongitude(), request.getHi().getLongitude());
  int top = max(request.getLo().getLatitude(), request.getHi().getLatitude());
  int bottom = min(request.getLo().getLatitude(), request.getHi().getLatitude());

  for (Feature feature : features) {
    if (!RouteGuideUtil.exists(feature)) {
      continue;
    }

    int lat = feature.getLocation().getLatitude();
    int lon = feature.getLocation().getLongitude();
    if (lon >= left && lon <= right && lat >= bottom && lat <= top) {
      responseObserver.onNext(feature);
    }
  }
  responseObserver.onCompleted();
}
```

Like the simple RPC, this method gets a request object (the `Rectangle` in which our client wants to find `Feature`s) and a `StreamObserver` response observer.

​	与简单的 RPC 一样，此方法获取一个请求对象（客户端希望在其中查找 `Feature` 的 `Rectangle` ）和一个 `StreamObserver` 响应观察器。

This time, we get as many `Feature` objects as we need to return to the client (in this case, we select them from the service’s feature collection based on whether they’re inside our request `Rectangle`), and write them each in turn to the response observer using its `onNext()` method. Finally, as in our simple RPC, we use the response observer’s `onCompleted()` method to tell gRPC that we’ve finished writing responses.

​	这次，我们获取了我们需要返回给客户端的 `Feature` 对象（在本例中，我们根据它们是否位于请求 `Rectangle` 中从服务的要素集合中选择它们），并使用其 `onNext()` 方法将它们逐个写入响应观察器。最后，与在简单的 RPC 中一样，我们使用响应观察器的 `onCompleted()` 方法告诉 gRPC 我们已完成写入响应。

##### Client-side streaming RPC 客户端流式 RPC

Now let’s look at something a little more complicated: the client-side streaming method `RecordRoute()`, where we get a stream of `Point`s from the client and return a single `RouteSummary` with information about their trip.

​	现在，我们来看一些更复杂的内容：客户端流式传输方法 `RecordRoute()` ，其中我们从客户端获取 `Point` 流并返回一个包含有关其行程信息的 `RouteSummary` 。

```java
@Override
public StreamObserver<Point> recordRoute(final StreamObserver<RouteSummary> responseObserver) {
  return new StreamObserver<Point>() {
    int pointCount;
    int featureCount;
    int distance;
    Point previous;
    long startTime = System.nanoTime();

    @Override
    public void onNext(Point point) {
      pointCount++;
      if (RouteGuideUtil.exists(checkFeature(point))) {
        featureCount++;
      }
      // For each point after the first, add the incremental distance from the previous point
      // to the total distance value.
      if (previous != null) {
        distance += calcDistance(previous, point);
      }
      previous = point;
    }

    @Override
    public void onError(Throwable t) {
      logger.log(Level.WARNING, "Encountered error in recordRoute", t);
    }

    @Override
    public void onCompleted() {
      long seconds = NANOSECONDS.toSeconds(System.nanoTime() - startTime);
      responseObserver.onNext(RouteSummary.newBuilder().setPointCount(pointCount)
          .setFeatureCount(featureCount).setDistance(distance)
          .setElapsedTime((int) seconds).build());
      responseObserver.onCompleted();
    }
  };
}
```

As you can see, like the previous method types our method gets a `StreamObserver` response observer parameter, but this time it returns a `StreamObserver` for the client to write its `Point`s.

​	如您所见，与之前的方法类型一样，我们的方法获取一个 `StreamObserver` 响应观察器参数，但这次它返回一个 `StreamObserver` 供客户端写入其 `Point` 。

In the method body we instantiate an anonymous `StreamObserver` to return, in which we:

​	在方法体中，我们实例化一个匿名 `StreamObserver` 以返回，其中我们：

- Override the `onNext()` method to get features and other information each time the client writes a `Point` to the message stream.
  每次客户端将 `Point` 写入消息流时，覆盖 `onNext()` 方法以获取要素和其他信息。
- Override the `onCompleted()` method (called when the *client* has finished writing messages) to populate and build our `RouteSummary`. We then call our method’s own response observer’s `onNext()` with our `RouteSummary`, and then call its `onCompleted()` method to finish the call from the server side.
  覆盖 `onCompleted()` 方法（在客户端完成写入消息时调用）以填充并构建我们的 `RouteSummary` 。然后，我们使用我们的 `RouteSummary` 调用我们方法自己的响应观察器的 `onNext()` ，然后调用它的 `onCompleted()` 方法以完成来自服务器端的调用。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```java
@Override
public StreamObserver<RouteNote> routeChat(final StreamObserver<RouteNote> responseObserver) {
  return new StreamObserver<RouteNote>() {
    @Override
    public void onNext(RouteNote note) {
      List<RouteNote> notes = getOrCreateNotes(note.getLocation());

      // Respond with all previous notes at this location.
      for (RouteNote prevNote : notes.toArray(new RouteNote[0])) {
        responseObserver.onNext(prevNote);
      }

      // Now add the new note to the list
      notes.add(note);
    }

    @Override
    public void onError(Throwable t) {
      logger.log(Level.WARNING, "Encountered error in routeChat", t);
    }

    @Override
    public void onCompleted() {
      responseObserver.onCompleted();
    }
  };
}
```

As with our client-side streaming example, we both get and return a `StreamObserver` response observer, except this time we return values via our method’s response observer while the client is still writing messages to *their* message stream. The syntax for reading and writing here is exactly the same as for our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	与我们的客户端流式传输示例一样，我们都会获取并返回一个 `StreamObserver` 响应观察器，但这次我们通过我们方法的响应观察器返回值，而客户端仍在向其消息流写入消息。此处用于读写操作的语法与我们的客户端流式传输和服务器流式传输方法完全相同。尽管每一方始终会按写入顺序获取另一方的消息，但客户端和服务器都可以按任何顺序进行读写操作——流完全独立地运行。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```java
public RouteGuideServer(int port, URL featureFile) throws IOException {
  this(ServerBuilder.forPort(port), port, RouteGuideUtil.parseFeatures(featureFile));
}

/** Create a RouteGuide server using serverBuilder as a base and features as data. */
public RouteGuideServer(ServerBuilder<?> serverBuilder, int port, Collection<Feature> features) {
  this.port = port;
  server = serverBuilder.addService(new RouteGuideService(features))
      .build();
}
...
public void start() throws IOException {
  server.start();
  logger.info("Server started, listening on " + port);
 ...
}
```

As you can see, we build and start our server using a `ServerBuilder`.

​	如您所见，我们使用 `ServerBuilder` 构建并启动服务器。

To do this, we: 
​	为此，我们：

1. Specify the address and port we want to use to listen for client requests using the builder’s `forPort()` method.
   使用构建器的 `forPort()` 方法指定我们想要用于侦听客户端请求的地址和端口。
2. Create an instance of our service implementation class `RouteGuideService` and pass it to the builder’s `addService()` method.
   创建服务实现类的实例 `RouteGuideService` ，并将其传递给构建器的 `addService()` 方法。
3. Call `build()` and `start()` on the builder to create and start an RPC server for our service.
   在构建器上调用 `build()` 和 `start()` 以创建和启动服务的 RPC 服务器。

### Creating the client 创建客户端

In this section, we’ll look at creating a client for our `RouteGuide` service. You can see our complete example client code in [grpc-java/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideClient.java](https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideClient.java).

​	在本节中，我们将着眼于为我们的 `RouteGuide` 服务创建一个客户端。您可以在 grpc-java/examples/src/main/java/io/grpc/examples/routeguide/RouteGuideClient.java 中看到我们完整的示例客户端代码。

#### Instantiating a stub 实例化存根

To call service methods, we first need to create a *stub*, or rather, two stubs:

​	要调用服务方法，我们首先需要创建一个存根，或者更确切地说，两个存根：

- a *blocking/synchronous* stub: this means that the RPC call waits for the server to respond, and will either return a response or raise an exception.
  一个阻塞/同步存根：这意味着 RPC 调用会等待服务器响应，并且会返回响应或引发异常。
- a *non-blocking/asynchronous* stub that makes non-blocking calls to the server, where the response is returned asynchronously. You can make certain types of streaming call only using the asynchronous stub.
  一个非阻塞/异步存根，它对服务器进行非阻塞调用，其中响应以异步方式返回。您只能使用异步存根进行某些类型的流式调用。

First we need to create a gRPC *channel* for our stub, specifying the server address and port we want to connect to:

​	首先，我们需要为存根创建一个 gRPC 通道，指定我们要连接到的服务器地址和端口：

```java
public RouteGuideClient(String host, int port) {
  this(ManagedChannelBuilder.forAddress(host, port).usePlaintext());
}

/** Construct client for accessing RouteGuide server using the existing channel. */
public RouteGuideClient(ManagedChannelBuilder<?> channelBuilder) {
  channel = channelBuilder.build();
  blockingStub = RouteGuideGrpc.newBlockingStub(channel);
  asyncStub = RouteGuideGrpc.newStub(channel);
}
```

We use a `ManagedChannelBuilder` to create the channel.

​	我们使用 `ManagedChannelBuilder` 创建通道。

Now we can use the channel to create our stubs using the `newStub` and `newBlockingStub` methods provided in the `RouteGuideGrpc` class we generated from our `.proto`.

​	现在，我们可以使用通道使用 `RouteGuideGrpc` 类中提供的 `newStub` 和 `newBlockingStub` 方法创建存根，该类是我们从 `.proto` 生成的。

```java
blockingStub = RouteGuideGrpc.newBlockingStub(channel);
asyncStub = RouteGuideGrpc.newStub(channel);
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods.

​	现在，我们来看看如何调用服务方法。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` on the blocking stub is as straightforward as calling a local method.

​	在阻塞存根上调用简单 RPC `GetFeature` 与调用本地方法一样简单。

```java
Point request = Point.newBuilder().setLatitude(lat).setLongitude(lon).build();
Feature feature;
try {
  feature = blockingStub.getFeature(request);
} catch (StatusRuntimeException e) {
  logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
  return;
}
```

We create and populate a request protocol buffer object (in our case `Point`), pass it to the `getFeature()` method on our blocking stub, and get back a `Feature`.

​	我们创建一个并填充请求协议缓冲区对象（在本例中为 `Point` ），将其传递给阻塞存根上的 `getFeature()` 方法，然后获取 `Feature` 。

If an error occurs, it is encoded as a `Status`, which we can obtain from the `StatusRuntimeException`.

​	如果发生错误，它将被编码为 `Status` ，我们可以从 `StatusRuntimeException` 获取该错误。

##### Server-side streaming RPC 服务器端流式 RPC

Next, let’s look at a server-side streaming call to `ListFeatures`, which returns a stream of geographical `Feature`s:

​	接下来，我们来看一下对 `ListFeatures` 的服务器端流式调用，它返回地理 `Feature` 流：

```java
Rectangle request =
    Rectangle.newBuilder()
        .setLo(Point.newBuilder().setLatitude(lowLat).setLongitude(lowLon).build())
        .setHi(Point.newBuilder().setLatitude(hiLat).setLongitude(hiLon).build()).build();
Iterator<Feature> features;
try {
  features = blockingStub.listFeatures(request);
} catch (StatusRuntimeException e) {
  logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
  return;
}
```

As you can see, it’s very similar to the simple RPC we just looked at, except instead of returning a single `Feature`, the method returns an `Iterator` that the client can use to read all the returned `Feature`s.

​	如您所见，它与我们刚刚看到的简单 RPC 非常相似，只是该方法返回 `Iterator` ，客户端可以使用它来读取所有返回的 `Feature` ，而不是返回单个 `Feature` 。

##### Client-side streaming RPC 客户端流式 RPC

Now for something a little more complicated: the client-side streaming method `RecordRoute`, where we send a stream of `Point`s to the server and get back a single `RouteSummary`. For this method we need to use the asynchronous stub. If you’ve already read [Creating the server]({{< ref "/Languages/Java/Basicstutorial#server" >}}) some of this may look very familiar - asynchronous streaming RPCs are implemented in a similar way on both sides.

​	现在来看一些更复杂的内容：客户端流式方法 `RecordRoute` ，我们向服务器发送 `Point` 流并获取单个 `RouteSummary` 。对于此方法，我们需要使用异步存根。如果您已经阅读了创建服务器，其中一些内容可能看起来非常熟悉 - 异步流式 RPC 在双方以类似的方式实现。

```java
public void recordRoute(List<Feature> features, int numPoints) throws InterruptedException {
  info("*** RecordRoute");
  final CountDownLatch finishLatch = new CountDownLatch(1);
  StreamObserver<RouteSummary> responseObserver = new StreamObserver<RouteSummary>() {
    @Override
    public void onNext(RouteSummary summary) {
      info("Finished trip with {0} points. Passed {1} features. "
          + "Travelled {2} meters. It took {3} seconds.", summary.getPointCount(),
          summary.getFeatureCount(), summary.getDistance(), summary.getElapsedTime());
    }

    @Override
    public void onError(Throwable t) {
      Status status = Status.fromThrowable(t);
      logger.log(Level.WARNING, "RecordRoute Failed: {0}", status);
      finishLatch.countDown();
    }

    @Override
    public void onCompleted() {
      info("Finished RecordRoute");
      finishLatch.countDown();
    }
  };

  StreamObserver<Point> requestObserver = asyncStub.recordRoute(responseObserver);
  try {
    // Send numPoints points randomly selected from the features list.
    Random rand = new Random();
    for (int i = 0; i < numPoints; ++i) {
      int index = rand.nextInt(features.size());
      Point point = features.get(index).getLocation();
      info("Visiting point {0}, {1}", RouteGuideUtil.getLatitude(point),
          RouteGuideUtil.getLongitude(point));
      requestObserver.onNext(point);
      // Sleep for a bit before sending the next one.
      Thread.sleep(rand.nextInt(1000) + 500);
      if (finishLatch.getCount() == 0) {
        // RPC completed or errored before we finished sending.
        // Sending further requests won't error, but they will just be thrown away.
        return;
      }
    }
  } catch (RuntimeException e) {
    // Cancel RPC
    requestObserver.onError(e);
    throw e;
  }
  // Mark the end of requests
  requestObserver.onCompleted();

  // Receiving happens asynchronously
  finishLatch.await(1, TimeUnit.MINUTES);
}
```

As you can see, to call this method we need to create a `StreamObserver`, which implements a special interface for the server to call with its `RouteSummary` response. In our `StreamObserver` we:

​	如您所见，要调用此方法，我们需要创建一个 `StreamObserver` ，它为服务器实现了一个特殊接口，以便使用其 `RouteSummary` 响应进行调用。在我们的 `StreamObserver` 中，我们：

- Override the `onNext()` method to print out the returned information when the server writes a `RouteSummary` to the message stream.
  当服务器向消息流写入 `RouteSummary` 时，覆盖 `onNext()` 方法以打印出返回的信息。
- Override the `onCompleted()` method (called when the *server* has completed the call on its side) to reduce a `CountDownLatch` that we can check to see if the server has finished writing.
  重写 `onCompleted()` 方法（在服务器在其端完成调用时调用）以减少 `CountDownLatch` ，我们可以检查它以查看服务器是否已完成写入。

We then pass the `StreamObserver` to the asynchronous stub’s `recordRoute()` method and get back our own `StreamObserver` request observer to write our `Point`s to send to the server. Once we’ve finished writing points, we use the request observer’s `onCompleted()` method to tell gRPC that we’ve finished writing on the client side. Once we’re done, we check our `CountDownLatch` to check that the server has completed on its side.

​	然后，我们将 `StreamObserver` 传递给异步存根的 `recordRoute()` 方法，并取回我们自己的 `StreamObserver` 请求观察器，以写入我们的 `Point` 并发送到服务器。一旦我们完成写入点，我们就使用请求观察器的 `onCompleted()` 方法告诉 gRPC 我们已完成在客户端上的写入。完成后，我们检查我们的 `CountDownLatch` 以检查服务器是否已在其端完成。

##### Bidirectional streaming RPC 双向流式 RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```java
public void routeChat() throws Exception {
  info("*** RoutChat");
  final CountDownLatch finishLatch = new CountDownLatch(1);
  StreamObserver<RouteNote> requestObserver =
      asyncStub.routeChat(new StreamObserver<RouteNote>() {
        @Override
        public void onNext(RouteNote note) {
          info("Got message \"{0}\" at {1}, {2}", note.getMessage(), note.getLocation()
              .getLatitude(), note.getLocation().getLongitude());
        }

        @Override
        public void onError(Throwable t) {
          Status status = Status.fromThrowable(t);
          logger.log(Level.WARNING, "RouteChat Failed: {0}", status);
          finishLatch.countDown();
        }

        @Override
        public void onCompleted() {
          info("Finished RouteChat");
          finishLatch.countDown();
        }
      });

  try {
    RouteNote[] requests =
        {newNote("First message", 0, 0), newNote("Second message", 0, 1),
            newNote("Third message", 1, 0), newNote("Fourth message", 1, 1)};

    for (RouteNote request : requests) {
      info("Sending message \"{0}\" at {1}, {2}", request.getMessage(), request.getLocation()
          .getLatitude(), request.getLocation().getLongitude());
      requestObserver.onNext(request);
    }
  } catch (RuntimeException e) {
    // Cancel RPC
    requestObserver.onError(e);
    throw e;
  }
  // Mark the end of requests
  requestObserver.onCompleted();

  // Receiving happens asynchronously
  finishLatch.await(1, TimeUnit.MINUTES);
}
```

As with our client-side streaming example, we both get and return a `StreamObserver` response observer, except this time we send values via our method’s response observer while the server is still writing messages to *their* message stream. The syntax for reading and writing here is exactly the same as for our client-streaming method. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	与我们的客户端流式传输示例一样，我们都会获取并返回一个 `StreamObserver` 响应观察器，但这次我们通过方法的响应观察器发送值，而服务器仍在向其消息流写入消息。此处用于读写操作的语法与用于客户端流式传输方法的语法完全相同。尽管每一方总是会按写入顺序获取另一方的消息，但客户端和服务器都可以按任何顺序进行读写操作——流完全独立地运行。

### Try it out! 试一试！[ ]({{< ref "/Languages/Java/Basicstutorial#try-it-out" >}})

Follow the instructions in the [example directory README](https://github.com/grpc/grpc-java/blob/master/examples/README.md) to build and run the client and server.

​	按照示例目录 README 中的说明构建并运行客户端和服务器。
