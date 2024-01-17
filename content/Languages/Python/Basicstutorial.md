+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/python/basics/](https://grpc.io/docs/languages/python/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Python.

​	Python 中 gRPC 的基本教程简介。



This tutorial provides a basic Python programmer’s introduction to working with gRPC.

​	本教程为 Python 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Python gRPC API to write a simple client and server for your service.
  使用 Python gRPC API 为您的服务编写一个简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). You can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and [Python generated code guide](https://protobuf.dev/reference/python/python-generated).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。您可以在 proto3 语言指南和 Python 生成的代码指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for this tutorial is in [grpc/grpc/examples/python/route_guide](https://github.com/grpc/grpc/tree/v1.60.0/examples/python/route_guide). To download the example, clone the `grpc` repository by running the following command:

​	本教程的示例代码位于 grpc/grpc/examples/python/route_guide 中。要下载示例，请通过运行以下命令克隆 `grpc` 存储库：

```sh
$ git clone -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

Then change your current directory to `examples/python/route_guide` in the repository:

​	然后将您的当前目录更改为存储库中的 `examples/python/route_guide` ：

```sh
$ cd grpc/examples/python/route_guide
```

You also should have the relevant tools installed to generate the server and client interface code - if you don’t already, follow the setup instructions in [Quick start]({{< ref "/Languages/Python/Quickstart" >}}).

​	您还应该安装相关工具来生成服务器和客户端接口代码 - 如果您还没有安装，请按照快速入门中的设置说明进行操作。

### Defining the service 定义服务

Your first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete `.proto` file in [`examples/protos/route_guide.proto`](https://github.com/grpc/grpc/blob/v1.60.0/examples/protos/route_guide.proto).

​	您的第一步（正如您从 gRPC 简介中了解到的）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 `examples/protos/route_guide.proto` 中看到完整的 `.proto` 文件。

To define a service, you specify a named `service` in your `.proto` file:

​	要定义服务，您需要在 `.proto` 文件中指定一个命名的 `service` ：

```protobuf
service RouteGuide {
   // (Method definitions not shown)
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the `RouteGuide` service:

​	然后在服务定义中定义 `rpc` 方法，指定它们的请求和响应类型。gRPC 允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

- A *simple RPC* where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.

  ​	一个简单的 RPC，其中客户端使用存根向服务器发送请求并等待响应返回，就像一个普通的函数调用一样。

  ```protobuf
  // Obtains the feature at a given position.
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A *response-streaming RPC* where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in the example, you specify a response-streaming method by placing the `stream` keyword before the *response* type.

  ​	客户端向服务器发送请求并获取一个流来读取一系列消息的响应流式 RPC。客户端从返回的流中读取，直到没有更多消息。如您在示例中所见，您可以通过在响应类型之前放置 `stream` 关键字来指定响应流式方法。

  ```protobuf
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A *request-streaming RPC* where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a request-streaming method by placing the `stream` keyword before the *request* type.

  ​	客户端编写一系列消息并使用提供的流将它们发送到服务器的请求流式 RPC。客户端完成编写消息后，它会等待服务器读取所有消息并返回其响应。通过在请求类型前放置 `stream` 关键字来指定请求流式方法。

  ```protobuf
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A *bidirectionally-streaming RPC* where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  ​	双方使用读写流发送一系列消息的双向流式 RPC。两个流独立运行，因此客户端和服务器可以按任何顺序进行读写：例如，服务器可以等到收到所有客户端消息后再编写其响应，或者可以交替读取消息然后编写消息，或其他一些读写组合。每个流中的消息顺序保持不变。通过在请求和响应前放置 `stream` 关键字来指定此类型的函数。

  ```protobuf
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
  ```

Your `.proto` file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the `Point` message type:

​	您的 `.proto` 文件还包含我们服务方法中使用的所有请求和响应类型的协议缓冲区消息类型定义 - 例如，以下是 `Point` 消息类型：

```protobuf
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

Next you need to generate the gRPC client and server interfaces from your `.proto` service definition.

​	接下来，您需要从 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。

First, install the grpcio-tools package:

​	首先，安装 grpcio-tools 包：

```sh
$ pip install grpcio-tools
```

Use the following command to generate the Python code:

​	使用以下命令生成 Python 代码：

```sh
$ python -m grpc_tools.protoc -I../../protos --python_out=. --pyi_out=. --grpc_python_out=. ../../protos/route_guide.proto
```

Note that as we’ve already provided a version of the generated code in the example directory, running this command regenerates the appropriate file rather than creates a new one. The generated code files are called `route_guide_pb2.py` and `route_guide_pb2_grpc.py` and contain:

​	请注意，由于我们已在示例目录中提供了一个已生成代码的版本，因此运行此命令会重新生成相应的文件，而不是创建一个新文件。生成的代码文件名为 `route_guide_pb2.py` 和 `route_guide_pb2_grpc.py` ，其中包含：

- classes for the messages defined in `route_guide.proto`
  在 `route_guide.proto` 中定义的消息的类

- classes for the service defined in

   

  ```
  route_guide.proto
  ```

  
  在 `route_guide.proto` 中定义的服务的类

  - `RouteGuideStub`, which can be used by clients to invoke RouteGuide RPCs
    `RouteGuideStub` ，客户端可使用它来调用 RouteGuide RPC
  - `RouteGuideServicer`, which defines the interface for implementations of the RouteGuide service
    `RouteGuideServicer` ，它定义了 RouteGuide 服务实现的接口

- a function for the service defined in

   

  ```
  route_guide.proto
  ```

  
  在 `route_guide.proto` 中定义的服务的函数

  - `add_RouteGuideServicer_to_server`, which adds a RouteGuideServicer to a `grpc.Server`
    `add_RouteGuideServicer_to_server` ，它将 RouteGuideServicer 添加到 `grpc.Server`

#### Note 注意

The `2` in pb2 indicates that the generated code is following Protocol Buffers Python API version 2. Version 1 is obsolete. It has no relation to the Protocol Buffers Language version, which is the one indicated by `syntax = "proto3"` or `syntax = "proto2"` in a `.proto` file.
pb2 中的 `2` 表示生成的代码遵循 Protocol Buffers Python API 版本 2。版本 1 已过时。它与 Protocol Buffers 语言版本无关，后者由 `.proto` 文件中的 `syntax = "proto3"` 或 `syntax = "proto2"` 指示。

#### Generating gRPC interfaces with custom package path 使用自定义包路径生成 gRPC 接口

To generate gRPC client interfaces with a custom package path, you can use the `-I` parameter along with the `grpc_tools.protoc` command. This approach allows you to specify a custom package name for the generated files.

​	要使用自定义包路径生成 gRPC 客户端接口，您可以将 `-I` 参数与 `grpc_tools.protoc` 命令结合使用。此方法允许您为生成的文件指定自定义包名称。

Here’s an example command to generate the gRPC client interfaces with a custom package path:

​	以下是一个使用自定义包路径生成 gRPC 客户端接口的示例命令：

```sh
$ python -m grpc_tools.protoc -Igrpc/example/custom/path=../../protos \
  --python_out=. --grpc_python_out=. \
  ../../protos/route_guide.proto
```

The generated files will be placed in the `./grpc/example/custom/path/` directory:

​	生成的文件将放置在 `./grpc/example/custom/path/` 目录中：

- `./grpc/example/custom/path/route_guide_pb2.py`
- `./grpc/example/custom/path/route_guide_pb2_grpc.py`

With this setup, the generated `route_guide_pb2_grpc.py` file will correctly import the protobuf definitions using the custom package structure, as shown below:

​	通过此设置，生成的 `route_guide_pb2_grpc.py` 文件将使用自定义包结构正确导入 protobuf 定义，如下所示：

```python
import grpc.example.custom.path.route_guide_pb2 as route_guide_pb2
```

By following this approach, you can ensure that the files will call each other correctly with respect to the specified package path. This method allows you to maintain a custom package structure for your gRPC client interfaces.

​	通过遵循此方法，您可以确保文件将根据指定的包路径正确地相互调用。此方法允许您为 gRPC 客户端接口维护自定义包结构。

### Creating the server 创建服务器

First let’s look at how you create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Python/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，让我们看看如何创建 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，则可以跳过本部分并直接转到创建客户端（尽管您可能仍然会觉得它很有趣！）。

Creating and running a `RouteGuide` server breaks down into two work items:

​	创建和运行 `RouteGuide` 服务器分为两项工作：

- Implementing the servicer interface generated from our service definition with functions that perform the actual “work” of the service.
  使用执行服务的实际“工作”的函数来实现从我们的服务定义生成的服务器接口。
- Running a gRPC server to listen for requests from clients and transmit responses.
  运行 gRPC 服务器以侦听来自客户端的请求并传输响应。

You can find the example `RouteGuide` server in [examples/python/route_guide/route_guide_server.py](https://github.com/grpc/grpc/blob/v1.60.0/examples/python/route_guide/route_guide_server.py).

​	您可以在 examples/python/route_guide/route_guide_server.py 中找到示例 `RouteGuide` 服务器。

#### Implementing RouteGuide 实现 RouteGuide

`route_guide_server.py` has a `RouteGuideServicer` class that subclasses the generated class `route_guide_pb2_grpc.RouteGuideServicer`:

​	 `route_guide_server.py` 有一个 `RouteGuideServicer` 类，它是生成类 `route_guide_pb2_grpc.RouteGuideServicer` 的子类：

```python
# RouteGuideServicer provides an implementation of the methods of the RouteGuide service.
class RouteGuideServicer(route_guide_pb2_grpc.RouteGuideServicer):
```

`RouteGuideServicer` implements all the `RouteGuide` service methods.

​	 `RouteGuideServicer` 实现所有 `RouteGuide` 服务方法。

##### Simple RPC 简单 RPC

Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	我们首先来看最简单的类型 `GetFeature` ，它只是从客户端获取一个 `Point` ，并从其数据库中返回相应的特征信息，形式为 `Feature` 。

```python
def GetFeature(self, request, context):
    feature = get_feature(self.db, request)
    if feature is None:
        return route_guide_pb2.Feature(name="", location=request)
    else:
        return feature
```

The method is passed a `route_guide_pb2.Point` request for the RPC, and a `grpc.ServicerContext` object that provides RPC-specific information such as timeout limits. It returns a `route_guide_pb2.Feature` response.

​	该方法传递一个 `route_guide_pb2.Point` RPC 请求，以及一个 `grpc.ServicerContext` 对象，该对象提供特定于 RPC 的信息，例如超时限制。它返回一个 `route_guide_pb2.Feature` 响应。

##### Response-streaming RPC 响应式流 RPC

Now let’s look at the next method. `ListFeatures` is a response-streaming RPC that sends multiple `Feature`s to the client.

​	现在我们来看下一个方法。 `ListFeatures` 是一个响应式流 RPC，它向客户端发送多个 `Feature` 。

```python
def ListFeatures(self, request, context):
    left = min(request.lo.longitude, request.hi.longitude)
    right = max(request.lo.longitude, request.hi.longitude)
    top = max(request.lo.latitude, request.hi.latitude)
    bottom = min(request.lo.latitude, request.hi.latitude)
    for feature in self.db:
        if (
            feature.location.longitude >= left
            and feature.location.longitude <= right
            and feature.location.latitude >= bottom
            and feature.location.latitude <= top
        ):
            yield feature
```

Here the request message is a `route_guide_pb2.Rectangle` within which the client wants to find `Feature`s. Instead of returning a single response the method yields zero or more responses.

​	此处，请求消息是一个 `route_guide_pb2.Rectangle` ，客户端希望在其中找到 `Feature` 。该方法不会返回单个响应，而是会产生零个或多个响应。

##### Request-streaming RPC 请求式流 RPC

The request-streaming method `RecordRoute` uses an [iterator](https://docs.python.org/2/library/stdtypes.html#iterator-types) of request values and returns a single response value.

​	请求式流方法 `RecordRoute` 使用请求值的迭代器并返回单个响应值。

```python
def RecordRoute(self, request_iterator, context):
    point_count = 0
    feature_count = 0
    distance = 0.0
    prev_point = None

    start_time = time.time()
    for point in request_iterator:
        point_count += 1
        if get_feature(self.db, point):
            feature_count += 1
        if prev_point:
            distance += get_distance(prev_point, point)
        prev_point = point

    elapsed_time = time.time() - start_time
    return route_guide_pb2.RouteSummary(
        point_count=point_count,
        feature_count=feature_count,
        distance=int(distance),
        elapsed_time=int(elapsed_time),
    )
```

##### Bidirectional streaming RPC 双向流式 RPC

Lastly let’s look at the bidirectionally-streaming method `RouteChat`.

​	最后我们来看一下双向流式方法 `RouteChat` 。

```python
def RouteChat(self, request_iterator, context):
    prev_notes = []
    for new_note in request_iterator:
        for prev_note in prev_notes:
            if prev_note.location == new_note.location:
                yield prev_note
        prev_notes.append(new_note)
```

This method’s semantics are a combination of those of the request-streaming method and the response-streaming method. It is passed an iterator of request values and is itself an iterator of response values.

​	此方法的语义是请求流式方法和响应流式方法语义的组合。它传递一个请求值迭代器，本身是一个响应值迭代器。

#### Starting the server 启动服务器

Once you have implemented all the `RouteGuide` methods, the next step is to start up a gRPC server so that clients can actually use your service:

​	一旦你实现了所有 `RouteGuide` 方法，下一步就是启动一个 gRPC 服务器，以便客户端实际使用你的服务：

```python
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    route_guide_pb2_grpc.add_RouteGuideServicer_to_server(RouteGuideServicer(), server)
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

The server `start()` method is non-blocking. A new thread will be instantiated to handle requests. The thread calling `server.start()` will often not have any other work to do in the meantime. In this case, you can call `server.wait_for_termination()` to cleanly block the calling thread until the server terminates.

​	服务器 `start()` 方法是非阻塞的。将实例化一个新线程来处理请求。在此期间，调用 `server.start()` 的线程通常没有其他工作要做。在这种情况下，你可以调用 `server.wait_for_termination()` 来干净地阻塞调用线程，直到服务器终止。

### Creating the client 创建客户端

You can see the complete example client code in [examples/python/route_guide/route_guide_client.py](https://github.com/grpc/grpc/blob/v1.60.0/examples/python/route_guide/route_guide_client.py).

​	你可以在 examples/python/route_guide/route_guide_client.py 中看到完整的示例客户端代码。

#### Creating a stub 创建存根

To call service methods, we first need to create a *stub*.

​	要调用服务方法，我们首先需要创建一个存根。

We instantiate the `RouteGuideStub` class of the `route_guide_pb2_grpc` module, generated from our `.proto`.

​	我们实例化 `route_guide_pb2_grpc` 模块的 `RouteGuideStub` 类，该类由我们的 `.proto` 生成。

```python
channel = grpc.insecure_channel('localhost:50051')
stub = route_guide_pb2_grpc.RouteGuideStub(channel)
```

#### Calling service methods 调用服务方法

For RPC methods that return a single response (“response-unary” methods), gRPC Python supports both synchronous (blocking) and asynchronous (non-blocking) control flow semantics. For response-streaming RPC methods, calls immediately return an iterator of response values. Calls to that iterator’s `next()` method block until the response to be yielded from the iterator becomes available.

​	对于返回单个响应的 RPC 方法（“response-unary”方法），gRPC Python 支持同步（阻塞）和异步（非阻塞）控制流语义。对于响应流式传输 RPC 方法，调用会立即返回一个响应值迭代器。对该迭代器的 `next()` 方法的调用会阻塞，直到要从迭代器中生成的响应可用。

##### Simple RPC 简单 RPC

A synchronous call to the simple RPC `GetFeature` is nearly as straightforward as calling a local method. The RPC call waits for the server to respond, and will either return a response or raise an exception:

​	对简单 RPC `GetFeature` 的同步调用几乎与调用本地方法一样简单。RPC 调用等待服务器响应，并将返回响应或引发异常：

```python
feature = stub.GetFeature(point)
```

An asynchronous call to `GetFeature` is similar, but like calling a local method asynchronously in a thread pool:

​	对 `GetFeature` 的异步调用类似，但就像在某个线程池中异步调用本地方法一样：

```python
feature_future = stub.GetFeature.future(point)
feature = feature_future.result()
```

##### Response-streaming RPC 响应流式 RPC

Calling the response-streaming `ListFeatures` is similar to working with sequence types:

​	调用响应流式 `ListFeatures` 类似于使用序列类型：

```python
for feature in stub.ListFeatures(rectangle):
```

##### Request-streaming RPC 请求流式 RPC

Calling the request-streaming `RecordRoute` is similar to passing an iterator to a local method. Like the simple RPC above that also returns a single response, it can be called synchronously or asynchronously:

​	调用请求流式 `RecordRoute` 类似于将迭代器传递给本地方法。就像上面也返回单个响应的简单 RPC 一样，它可以同步或异步调用：

```python
route_summary = stub.RecordRoute(point_iterator)
route_summary_future = stub.RecordRoute.future(point_iterator)
route_summary = route_summary_future.result()
```

##### Bidirectional streaming RPC 双向流式 RPC

Calling the bidirectionally-streaming `RouteChat` has (as is the case on the service-side) a combination of the request-streaming and response-streaming semantics:

​	调用双向流式 `RouteChat` （就像在服务端一样）具有请求流式和响应流式语义的组合：

```python
for received_route_note in stub.RouteChat(sent_route_note_iterator):
```

### Try it out! 试一试！

Run the server: 
​	运行服务器：

```sh
$ python route_guide_server.py
```

From a different terminal, run the client:

​	从另一个终端运行客户端：

```sh
$ python route_guide_client.py
```
